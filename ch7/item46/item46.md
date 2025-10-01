# 아이템 46

스트림의 변환 단계는 이전 단계의 결과를 받아 처리하는 순수함수여야함

순수함수란

- 오직 입력만이 결과에 영향을 주는 함수
- 즉 다른 상태를 참조 또는 변경하지 않는다.

그러기 위해선 스트림에선 부작용 없는 함수를 사용해야함

여기서 부작용이 없다의 의미는(책에는 안 나와있고 나의 생각) 다른 상태(객체)를 참조하지 않고, 간결해야함

```java
// 조건
List<Student> students = Arrays.asList(
        new Student("김철수", 85, "수학"),
        new Student("이영희", 92, "수학"),
        new Student("박민수", 78, "영어"),
        new Student("최지원", 95, "영어"),
        new Student("김철수", 88, "영어"),
        new Student("이영희", 90, "물리")
);

// 1. 부작용 있는 함수
Map<String, Double> subjectAverage = new HashMap<>();
Map<String, Integer> subjectCount = new HashMap<>();

students.stream().forEach(student -> {
		// students만 아니라 외부상태인 subjectAverage, subjectCount도 변경하고 있음
		// 스트림 코드를 가장한 반복적 코드
    subjectAverage.merge(student.getSubject(), (double) student.getScore(), Double::sum);
    subjectCount.merge(student.getSubject(), 1, Integer::sum);
});

// 후처리까지 해야하는 문제점
subjectAverage.replaceAll((subject, sum) -> sum/subjectCount.get(subject));

System.out.print(subjectAverage);

// 2. 부작용 없는 함수
Map<String, Double> result = students.stream()
        .collect(groupingBy(
                Student::getSubject,
                averagingInt(Student::getScore)
        ));
System.out.println(result);
```

### forEach

스트림 계산 결과를 보고할 때만 사용

계산하는 데는 쓰지 말자.

가장 ‘덜’ 스트림답다.

### Collector

java.util.stream.Collectors : 39개 메서드

축소(reduction) 전략을 캡슐화한 블랙박스 객체라고 생각하기 바란다

축소는 스트림의 원소들을 객체 하나에 취합한다는 뜻

### 기본 수집기들

**세 가지 기본 수집기:**

- `toList()`: 리스트로 수집
- `toSet()`: 집합으로 수집
- `toCollection(collectionFactory)`: 지정한 컬렉션 타입으로 수집

### 세팅코드

```java
List<Order> orders = Arrays.asList(
        new Order("C001", "노트북", 1, 1200000, LocalDate.of(2024, 1, 15)),
        new Order("C002", "마우스", 2, 25000, LocalDate.of(2024, 1, 16)),
        new Order("C001", "키보드", 1, 80000, LocalDate.of(2024, 1, 17)),
        new Order("C003", "노트북", 1, 1300000, LocalDate.of(2024, 1, 18)),
        new Order("C002", "모니터", 1, 300000, LocalDate.of(2024, 1, 19))
);
```

### groupingBy

```java
// 매핑
Map<String, Long> counting = orders.stream()
        .collect(groupingBy(Order::getProductName, counting())); // 빈도수 매핑

// 정렬
List<String> sorted = counting.keySet().stream()
        .sorted(comparing(counting::get).reversed()) // 빈도수 내림차순
        .limit(3)
        .toList();

// comparing 원리
Comparator<String> comparator = comparing(counting::get);
==
Comparator<String> comparator = (product1, product2) -> {
    Long count1 = counting.get(product1);
    Long count2 = counting.get(product2);
    return count1.compareTo(count2);  // 오름차순 비교
};
```

### toMap

```java
// 중복 방지 toMap
Map<String, Double> map = orders.stream()
        .collect(Collectors.toMap(
                Order::getProductName,
                Order::getPrice,
                (existing, replacement) -> existing // 중복이 발생하면 mergeFunction.apply(existing, replacement) 이게 발생하는데 이것을 람다식으로 바꾸면 됨
        ));

// 병합 함수가 있는 toMap
Map<String, Order> customerMaxOrder = orders.stream()
      .collect(toMap(
              Order::getCustomerId, // key
              order -> order, // value
              BinaryOperator.maxBy(
                      comparing(Order::getPrice)
      )));
      
// maxBy 동작
public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
    Objects.requireNonNull(comparator);
    return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
}

// 마지막 값으로 덮기
TreeMap<String, Double> treeMap = orders.stream()
        .collect(toMap(
                Order::getProductName,
                Order::getPrice,
                (oldVal, newVal) -> newVal, // 키가 중복되면 값을 덮음
                TreeMap::new // 맵 자식 중에 어떤 맵으로 할지 정함, TreeMap은 키로 정렬됨
        ));
```

BinaryOperator :  같은 타입의 두 개 인수를 받아서 같은 타입의 결과를 반환하는 함수형 인터페이스

### toCollection

```java
LinkedList<String> linkedList = orders.stream()
        .map(Order::getProductName)
        .collect(toCollection(LinkedList::new));
```

리스트나 집합 대신 컬렉션을 값으로 갖는 맵을 생성한다. 

원하는 컬렉션 타입을 선택할 수 있다는 유연성

### 점층적 인수 목록 패턴

점층적 인수 목록 패턴(telescoping argument list pattern)에 어긋난다.

```java
// 기존 코드
Map<String, Long> freq = words
    .collect(groupingBy(String::toLowerCase, counting()));
    

// 실패
Map<String, Long> freq = words
    .collect(groupingBy(String::toLowerCase, counting(), TreeMap::new));
    -> TreeMap을 사운데로 해야 함, 아니면 컴파일 에러
    
// 성공
Map<String, Long> freq = words
    .collect(groupingBy(String::toLowerCase, TreeMap::new, counting()));
 -> 이렇게 가운데에 추가하면 점층적 인수 목록 패턴에 어긋남
```

### counting

counting 메서드가 반환하는 수집기는 다운스트림 수집기 전용이다

collect(counting()) 형태로사용할 일은 전혀 없다

```java
// 이것보다
long count1 = words.stream()
    .collect(counting());
    
// 이게 더 나음
long count2 = words.stream()
    .count();

// counting()을 사용하는 이유는 다운스트림으로 이용하기 위해서
Map<String, Long> counting = orders.stream()
        .collect(groupingBy(Order::getProductName, counting())); // 빈도수 정렬
```

### summing

더하기

```java
// 고객별로 총 주문 금액 계산
Map<String, Double> summing = orders.stream()
        .collect(groupingBy(
                Order::getCustomerId,
                summingDouble(order -> order.getPrice() * order.getQuantity())
        ));
```

### summarizing

통계 내기

```java
Map<String, DoubleSummaryStatistics> summarizing = orders.stream()
        .collect(groupingBy(
                Order::getCustomerId,
                summarizingDouble(order -> order.getPrice() * order.getQuantity())
        ));
summarizing.forEach((customer, stats) -> {
    System.out.printf("%s: 주문수=%d, 총액=%.0f, 평균=%.0f, 최소=%.0f, 최대=%.0f%n",
            customer, stats.getCount(), stats.getSum(),
            stats.getAverage(), stats.getMin(), stats.getMax());
});
```

### joining

문자열 합치기

- `joining()`: 단순 연결
- `joining(",")`: 구분문자로 연결
- `joining(",", "[", "]")`: 접두/접미문자까지 포함

```java
String joining = orders.stream()
        .map(order -> String.join(",",
                order.getCustomerId(),
                order.getProductName(),
                String.valueOf(order.getQuantity()),
                String.valueOf(order.getPrice())
        ))
        .collect(joining("\n"));
System.out.println("joining : " + joining);

C001,노트북,1,1200000.0
C002,마우스,2,25000.0
C001,키보드,1,80000.0
C003,노트북,1,1300000.0
C002,모니터,1,300000.0
```

### partitioning

Boolean 키의 맵 반환

해당 조건에 만족하는 리스트, bool값을 map으로

```java
Map<Boolean, List<Order>> partitioningBy = orders.stream()
        .collect(partitioningBy(order -> order.getPrice() >= 50000));
partitioningBy.get(true).forEach(order -> System.out.println(order.getCustomerId()));
partitioningBy.get(false).forEach(order -> System.out.println(order.getCustomerId()));

```

## 핵심 원칙

스트림을 올바르게 사용하려면:

1. **모든 함수 객체가 부작용이 없어야 함**
2. **forEach는 계산이 아닌 보고용으로만 사용**
3. **수집기를 적극 활용**
4. **가장 중요한 수집기들**: `toList`, `toSet`, `toMap`, `groupingBy`, `joining`