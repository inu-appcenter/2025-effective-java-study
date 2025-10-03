# Item 45

추가 일시: 2025년 9월 20일 오후 7:39
강의: Effective JAVA

## 🍀 Item 45: 스트림은 주의해서 사용하라

---

Stream은 다량의 데이터를 처리하기 위한 API이다.

Stream은 기본적인 타입으로 int, long, double 세 가지를 지원한다.

### ✅ Stream의 pipeline

---

source steam → 중간 연산 → 종단 연산

각 중간 연산은 스트림을 어떠한 방식으로 변환한다.

```java
// 사람 이름 리스트
["김철수", "이영희", "박민수", "최지은"]
    ↓
.filter(name -> name.startsWith("김"))  // 중간: 김씨만
    ↓
["김철수"]
    ↓
.map(name -> name + "님")  // 중간: 님 붙이기
    ↓
["김철수님"]
    ↓
.forEach(System.out::println)  // 종단: 출력
    ↓
결과: "김철수님" 출력되고 끝!
```

스트림 파이프라인은 Lazy Evaluation 된다.

쉽게 말해 종단 연산이 없다면 스트림 파이프라인은 아무 일도 않는다.

EX) 어떠한 결과를 반환할지 정해주지 않는다면(리스트를 출력해!) 중간 연산에 어떤 코드를 적어놓던, 아무일도 하지 않는다.

**스트림 API는 메서드 연쇄를 지원하는 플루언트 API다.**

<aside>
💡

**플루언트 API란?**

파이프라인을 구성하는 모든 호출을 연결하여 하나의 표현식으로 완성 할 수 있다.

builder와 같이  . 으로 연결하여 표현식 하나로 만든다는 말이다.

</aside>

스트림 파이프 라인은 순차적으로 수행되지만, 병렬로 처리할 수도 있다.

parallel 메서드만 호출해주면 된다 → 다만, 효과를 볼 수 있는 상황이 많지는 않다.

개인적인 생각으로 AI와 협업하는 프로젝트라면 유용하게 쓸 수 있을거라는 생각이 든다.

```java
public static void main(String[] args) {
        //테스트 1: 5,000개
        List<Integer> small = IntStream.rangeClosed(1, 5_000).boxed().toList();
        test("5천개", small, false);

        //테스트 2: 100,000개
        List<Integer> large = IntStream.rangeClosed(1, 100_000).boxed().toList();
        test("10만개", large, true);
    }

    static void test(String name, List<Integer> data, boolean heavy) {

        //순차 처리
        long start = System.nanoTime();
        long sum1 = data.stream()
                .filter(n -> n % 2 == 0)
                .map(n -> heavy ? heavyWork(n) : n * 2)
                .reduce(0, Integer::sum);
        long seq = System.nanoTime() - start;

        //병렬 처리
        start = System.nanoTime();
        long sum2 = data.stream().parallel()
                .filter(n -> n % 2 == 0)
                .map(n -> heavy ? heavyWork(n) : n * 2)
                .reduce(0, Integer::sum);
        long par = System.nanoTime() - start;

        //결과 출력
        double seqMs = seq / 1_000_000.0;
        double parMs = par / 1_000_000.0;

        System.out.printf("sequence: %.2fms | parallel: %.2fms\n", seqMs, parMs);
    }

    //무거운 작업
    static int heavyWork(int n) {
        double result = n;
        for (int i = 0; i < 100; i++) {
            result = Math.sqrt(result + i) * Math.sin(result);//제곱근과 sin계산
        }
        return (int) result;
    }
    

결과
sequence: 4.64ms | parallel: 4.44ms
sequence: 92.57ms | parallel: 17.51ms
```

parallel()은 여러개의 스레드가 동시에 처리 할 수 있도록 한다.

### ✅ 스트림 과용의 문제점

---

코드 45-2

```java
words.collect(
    groupingBy(word -> word.chars().sorted()
        .collect(StringBuilder::new,
            (sb, c) -> sb.append((char) c),
            StringBuilder::append).toString()))
    .values().stream()
    .filter(group -> group.size() >= minGroupSize)
    .map(group -> group.size() + " " + group)
    .forEach(System.out::println);
```

코드 45-3

```java
words.collect(groupingBy(word -> alphabetize(word)))
    .values().stream()
    .filter(group -> group.size() >= minGroupSize)
    .forEach(g -> System.out.println(g.size() + " " + g));

//복잡한 로직은 별도 메서드로 분리
private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
}
```

stream의 적절한 사용은 간단명료한 코드를 작성할 수 있도록 돕지만, 지나친 사용은 오히려 이해하기 어려운 코드를 야기할 수 있다.

### ✅ 스트림으로 할 수 없는 것

---

- 지역변수 수정

```java
int count = 0;
list.stream().forEach(x -> count++);
```

다음과 같이 지역변수를 람다식에 사용하면 컴파일 에러가 발생하게 된다.

- return, break, continue 사용

```java
List<Integer> list = Arrays.asList(5, 15, 3, 20, 8);

list.stream().forEach(x -> {
    if (x > 10) return;
});
```

다음과 같은 15가 입력됨과 동시에 반복이 멈추기를 바라는 실수를 할 수 있다.

하디만 실제 코드는 15와 20을 건너뛸 뿐 stream 반복을 종료하지 않는다.

조건에 해당하는 forEach 내부만 건너뛸 뿐이다. → 마치 continue처럼 동작한다.

- 예외처리

전혀 불가능한 것은 아니지만 for문에 비해 복잡한 처리가 필요하다.

try-catch로 감싸는 방법이나, 래퍼 메서드를 작성하는 방법이 존재는 하지만 그럴바에는 for문을 쓰는게 나아보인다.

**중간값 손실 문제**

스트림 파이프라인은 한 값을 다른 값으로 매핑하면 기존의 값을 잃어버린다.

때문에 여러 단계를 거친 데이터의 각 단계 값에 동시 접근이 어렵다.

책에서는 원래 값을 계산하기 위해 역계산을 사용하면 된다고 제시하지만, 결국 이는 중간값에 접근하는 것이 아니다.

최종 결과로부터 원래 값을 역산 하는 것 뿐이다.

### ✅ ****다음과 같은 상황에 stream을 사용하자

---

1. 원소들의 시퀀스를 일관되게 변환한다.
2. 원소들의 시퀀스를 필터링한다.
3. 원소들의 시퀀스를 하나의 연산을 사용해 결합한다(더하기, 연결하기, 최솟값 구하기 등).
4. 원소들의 시퀀스를 컬렉션에 모은다(아마도 공통된 속성을 기준으로 묶어가며).
5. 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

**flatMap이란?**

일단 책에서 설명하는 데카르트 곱을 짧게 짚고 넘어가보자

일반적으로 트럼프카드는 4가지의(스페이드, 다이아, 하트, 클로버) 무늬와 13가지의 숫자의 조합으로 이루어져있다.

이 때 모든 조합을 만들면 ♠️1, ♠️2, ♠️3 … ❤️1, ❤️2, ❤️3 … 이런식으로 나오는데 이를 데카르트 곱이라고 한다.

일반 map을 사용하면 [[♠️1, ♠️2, ♠️3 …], [❤️1, ❤️2, ❤️3 …], […], […]] 다음과 같은 결과가 반환된다.

이때 flatMap을 사용하면 [♠️1, ♠️2, ♠️3 … ❤️1, ❤️2, ❤️3 …] 다음과 같은 하나의 평평한 스트림을 반환 할 수 있다.

### ✅ 결론

---

스트림을 사용해야하는 경우도 있고, 반복문으로 처리해야하는 경우도 있다.

어느쪽이 확실하게 좋은지 판단하기 어렵다면, 둘 다 작성 후 더 나은 쪽을 작성해라