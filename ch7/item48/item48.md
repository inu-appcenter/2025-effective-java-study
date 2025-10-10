<aside>
💡

스트림 병렬화는 주의해서 적용하라

</aside>

자바 8에서부터, 스트림에 `parallel()` 메서드만 추가하면 병렬 처리가 가능해졌다. 병렬 처리란 하나의 작업을 여러 개의 작은 작업으로 나눠서 동시에 처리하는 것이고, `parallel()` 메서드는 순차 스트림을 병렬 스트림으로 변환해준다. 여러 스레드가 작업을 나눠서 동시에 처리해준다는 말인데… 생각대로 동작하지 않을 때가 있다.

### 🤯병렬화 실패

```java
// 순차 스트림 - 12.5초 만에 완료
public static void main(String[] args) {
    primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
        .filter(mersenne -> mersenne.isProbablePrime(50))
        .limit(20)
        .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
    return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

스트림을 사용해 처음 20개의 메르센 소수를 생성하는 위 프로그램은 12.5초만에 완료된다. 더 빠르게 출력하고 싶으니 parallel()을 추가하면 어떨까? 

```java
// 병렬 스트림 - 응답 불가 상태
primes().parallel()  // 이게 문제 
    .map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
```

병렬화로 속도를 향상시킬 거라 기대했으나, 이 프로그램은 **아무것도 출력하지 못한다.** 게다가 CPU를 90% 잡아먹은 채로 멈춰 있다고 한다.

원인을 한줄요약하자면, 놀랍게도 **‘어떻게 병렬화할지 못 찾아서’**라고 한다…

병렬화가 실패하게 만드는 원인에는 두 가지가 있는데, 위 코드는 두 원인을 모두 포함하고 있다. 

1. Stream.iterate는 순차적 의존성이 있어 병렬화가 불가능하기 때문에
    
    ```java
    // Stream.iterate의 문제점
    Stream.iterate(TWO, BigInteger::nextProbablePrime)
    // 이건 이렇게 동작함:
    // 2 → nextProbablePrime(2) → 3
    // 3 → nextProbablePrime(3) → 5  
    // 5 → nextProbablePrime(5) → 7
    // 각 단계가 이전 단계에 의존! 병렬로 나눌 수가 없음
    ```
    
2. limit 연산은 순서 의존적이라 병렬화 효율이 떨어지기 때문에
    
    ```java
    // limit(20)의 의미: "처음 20개"
    // 병렬 환경에서의 문제
    
    Thread 1이 찾은 것: [3번째, 7번째, 11번째, 15번째 소수]
    Thread 2가 찾은 것: [1번째, 5번째, 9번째, 13번째 소수]
    Thread 3이 찾은 것: [2번째, 6번째, 10번째, 14번째 소수]
    Thread 4가 찾은 것: [4번째, 8번째, 12번째, 16번째 소수]
    
    // 이제 "처음 20개"를 어떻게 정할까?
    // 순서를 맞춰서 재정렬해야 함 → 병렬화 의미 없음
    ```
    

게다가 메르센 소수는 뒤로 갈수록 계산 시간이 기하급수적으로 증가한다. 20번째 메르센 소수를 찾는 시간은 앞의 19개를 모두 찾는 시간과 맞먹는다. 병렬화 알고리즘은 이런 불균등한 작업 분배를 전혀 예측하지 못한다.

### **병렬화가 효과적인(적합한) 자료구조**

데이터를 원하는 크기로, 정확하고, 손쉽게 나눌 수 있어야 한다. 다수의 스레드에 분배하기에 적합한 아래 예시들이 병렬화가 효과적인 자료구조에 해당한다. 

```java
// ArrayList - 인덱스로 정확한 분할 가능
List<Integer> numbers = new ArrayList<>(1_000_000);
IntStream.range(0, 1_000_000).forEach(numbers::add);

long sum = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();  // 효과적인 병렬화

// 배열 - 가장 좋은 참조 지역성
int[] array = new int[1_000_000];
long arraySum = Arrays.stream(array)
    .parallel()
    .sum();  // 매우 효과적

// HashMap - 버킷 단위로 분할 가능
Map<String, Integer> map = new HashMap<>();
long mapSum = map.values().parallelStream()
    .mapToInt(Integer::intValue)
    .sum();  // 효과적

// IntStream.range - 균등 분할 가능
long rangeSum = IntStream.rangeClosed(1, 1_000_000)
    .parallel()
    .sum();  // 매우 효과적
```

array 예시에 ‘참조 지역성이 가장 좋다’고 적어두었는데, **참조 지역성(locality of reference)** 역시 위 자료구조들의 공통점이다. 참조 지역성이란, 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻으로, 메모리에 연속으로 저장된 데이터는 캐시 히트율이 높아 병렬 처리가 빠르다. (캐시 히트율: 캐시 메모리에서 요청한 데이터를 얼마나 성공적으로 찾았는지 나타내는 척도)

데이터가 메모리에 연속으로 저장되어 있는 경우, CPU 캐시는 한번에 여러 데이터를 가져올 수 있을 것이다. 반대로 데이터가 여기저기 흩어져 있으면? 스레드는 데이터가 주 메모리에서 캐시로 올라오길 기다리며 시간을 낭비하게 된다. 

```java
// 기본 타입 배열 - 참조 지역성 최고
int[] primitiveArray = new int[10_000_000];
// [1][2][3][4][5]... 데이터가 메모리에 쫙 붙어있음

// 참조 타입 배열 - 참조만 붙어있음 
Integer[] boxedArray = new Integer[10_000_000];
// [ref1][ref2][ref3]... 참조만 붙어있고
//   ↓      ↓      ↓
// [obj1] [obj2] [obj3]  실제 객체는 힙 여기저기 흩어짐
```

위 예시처럼, 기본 타입 배열은 참조가 아닌 데이터 자체가 연속해서 저장되어 있으니 참조 지역성이 가장 뛰어나다. 반면 박싱된 배열은 참조를 타고 가야 실제 값을 얻을 수 있다. 

병렬화의 효과는 종단 연산이 무엇인지에 따라서도 효과가 천차만별이다.  종단 연산이 순차적이거나 병합 비용이 크면 병렬화를 해봤자 소용이 없다. 

```java
stream.filter(...)  // 중간 연산 (계속 스트림 반환)
      .map(...)     // 중간 연산
      .sorted()     // 중간 연산
      .forEach(...) // 종단 연산 (스트림 종료, 결과 생성)
      
// 종단 연산의 종류들:
.forEach()      // 각 원소에 작업 수행
.collect()      // 리스트, 맵 등으로 수집
.reduce()       // 하나의 값으로 축소
.count()        // 개수 세기
.min(), .max()  // 최소/최대값
.anyMatch()     // 조건 만족하는 게 하나라도 있나?
```

**종단 연산 중에서는 축소(reduction)**가 병렬화에 가장 적합하다고 한다. 축소 연산은 모든 원소를 하나로 합치는 작업이다. 

```java
// reduce - 부분 결과를 합치기 쉬움
int sum = IntStream.range(0, 1_000_000)
    .parallel()
    .reduce(0, (a, b) -> a + b);
// 각 스레드가 부분 합을 구하고, 나중에 합치기만 하면 됨

// 내장 축소 메서드들 - 병렬화에 최적화되어 있음
OptionalInt min = stream.parallel().min();
OptionalInt max = stream.parallel().max();
long count = stream.parallel().count();
int sum = stream.parallel().sum();
```

**조건에 맞으면 바로 반환되는 메서드**들도 병렬화 효과가 좋다.

```java
// 하나라도 찾으면 바로 종료 - 효율적
boolean hasNegative = IntStream.range(-1000, 1000)
    .parallel()
    .anyMatch(n -> n < 0);  // -1000 찾자마자 끝

boolean allPositive = stream.parallel().allMatch(n -> n > 0);
boolean noneNegative = stream.parallel().noneMatch(n -> n < 0);
```

반면에 collect는 가변 축소를 수행한다. 이 경우 각 스레드의 결과를 합치는 과정이 복잡하고 비싸다.

```java
// collect - 부분 컬렉션들을 합치는 비용이 큼
List<String> result = IntStream.range(0, 1_000_000)
    .parallel()
    .mapToObj(String::valueOf)
    .collect(Collectors.toList());
// Thread1: [0, 1, 2...]
// Thread2: [250000, 250001...]
// Thread3: [500000, 500001...]
// Thread4: [750000, 750001...]
// 이 리스트들을 하나로 합치는 과정이 비싸다
```

### 안전 실패

잘못된 병렬화는 성능 악화뿐만 아니라 **결과 자체에도 악영향**을 줄 수 있다. 이걸 안전 실패(safety failure)라고 부른다.

안전 실패를 방지하기 위해서, 병렬 스트림에서 사용하는 함수들은 아래 규칙을 만족해야 한다.

1. **결합 법칙 만족**
    
    ```java
    // Good - 덧셈은 순서 상관없음
    // (1 + 2) + 3 = 1 + (2 + 3) = 6
    int sum = stream.parallel()
        .reduce(0, (a, b) -> a + b);
    
    // Bad - 뺄셈은 순서에 따라 결과가 달라짐
    // (1 - 2) - 3 = -4
    // 1 - (2 - 3) = 2
    int wrong = stream.parallel()
        .reduce(0, (a, b) -> a - b);  // 병렬 실행 시 결과 예측 불가
    ```
    
2. **파이프라인 실행 중 데이터 소스를 건드리지 않기**
    
    ```java
    List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
    numbers.parallelStream()
        .map(n -> {
            if (n == 3) numbers.add(99);  // 절대 금지!
            return n * 2;
        })
        .forEach(System.out::println);
    // ConcurrentModificationException 또는 데이터 누락
    ```
    
3. **상태를 가지지 않기**
    
    ```java
    // Bad - 상태를 공유하는 위험한 코드
    class Counter {
        int count = 0;
    }
    Counter counter = new Counter();
    
    List<Integer> filtered = IntStream.range(0, 1000)
        .parallel()
        .filter(n -> {
            counter.count++;  // 여러 스레드가 동시에 접근
            return n % 2 == 0;
        })
        .boxed()
        .collect(Collectors.toList());
    
    System.out.println(counter.count);  // 1000이 아닐 수 있음
    ```
    

위 규칙을 지키지 못하더라도, 파이프라인을 순차적으로 수행하기만 한다면 결과에는 영향이 없다. 문제는 병렬로 수행할 때이다… 

앞에서 병렬화했던 메르센 소수 프로그램의 경우, **완료되더라도 출력된 소수의 순서가 예상과 다를 수 있다(올바르지 않을 수 있다).** forEachOrdered를 쓰면 되긴 하는데, 이러면 굳이 병렬화를 할 이유가 없는 것 같다.

```java
// forEach - 순서 개판
IntStream.range(0, 20)
    .parallel()
    .forEach(System.out::print);
// 출력: 561011121314171819234789...  

// forEachOrdered - 순서는 지키지만 느림
IntStream.range(0, 20)
    .parallel()
    .forEachOrdered(System.out::print);
// 출력: 01234567891011...19
// 순서 보장하느라 병렬화 이점이 사라짐
```

### 병렬화는 언제 하나

병렬화로 효과를 보기 위해서는 고려할 것이 많다… 빠르게 판단하는 방법은 없을까.

**원소 수 × 원소당 연산 수 ≥ 100,000**

라고 한다… 🤮🤮

음… 쓸 일이 있을까 싶지만, 조건만 잘 갖추면 parallel 하나로 프로세서 코어 수에 비례할 정도의 성능 향상을 맛볼 수 있다고 한다. 머신러닝이나 데이터 처리 등의 특정 분야에서 자주 고려하는 듯하다. 아래는 병렬화가 실제로 효과를 보이는 예제들이다.

```java
// 순차 버전
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}

// 병렬 버전 - parallel() 하나 추가
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
        .parallel()  // 이 한 줄의 마법
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}
//순차 버전은 31초, 병렬 버전은 9.2초로 3.37배 빨라짐
```

근데 또 n이 크다면 이 방식으로 하지 말란다… ☠️☠️☠️

```java
// n이 작을 때 (n = 1,000,000): 괜찮음
LongStream.rangeClosed(2, n)
    .parallel()
    .mapToObj(BigInteger::valueOf)
    .filter(i -> i.isProbablePrime(50))
    .count();

// n이 클 때 (n = 100,000,000): 문제!
// 1. 메모리: 1억 개 BigInteger 객체 생성 → 메모리 부족
// 2. 비효율: 대부분 숫자는 소수가 아님
//    예: 2~100 중 소수는 25개뿐 (75개는 쓸데없이 검사)
// 3. 소수 판별 시간: 큰 수일수록 오래 걸림

// 더 나은 방법: 소수의 성질을 이용한 알고리즘
// 예: 에라토스테네스의 체 (Sieve of Eratosthenes)
```

**올바른 수행과 성능 개선에 확신이 있는 경우에만! 병렬화를 시도하자.** 

**반드시 성능 테스트를 거쳐 사용 가치를 확인하도록 하자.**
