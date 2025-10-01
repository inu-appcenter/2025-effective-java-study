# item47

## 📂 반환 타입으로는 스트림보다 컬렉션이 낫다

---

자바에서 **여러 원소를 반환하는 메서드**는 흔히 List, Set, Collection, Iterable, 배열 같은 타입을 썼다.

👉 그런데, 자바 8에서 **Stream**이 도입되면서 문제가 복잡해졌다.

> *스트림 (Stream)*
>

스트림은 `반복(iteration)`을 지원하지 않는다. (for-each 불가)
따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다.

‼️ But, 사실 Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함할 뿐만 아니라, Iterable 인터페이스가 정의한 방식대로 동작한다.

🤔 그럼 왜 for-each로 스트림을 반복할 수 없을까?
→ 그 이유는 Stream이 Iterable을 확장하지 않기 때문이다.
→ 그럼 이를 해결할 방법은 없는가?

- Stream의 iterator 메서드에 메서드 참조를 건네기

    ```java
    for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
    		...
    }
    
    // 자바 타입 추론의 한계로 컴파일되지 않는다.
    // 이는 다음과 같은 컴파일 오류가 발생한다.
    
    Test.java:6: error: method reference not expected here
    for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
    
    // 이를 해결하려면 메서드 참조를 매개변수화된 Iterable로 적절히 형변환해줘야 한다.
    for (ProcessHandle ph : (Iterable<ProcessHandle>)
    												ProcessHandle.allProcesses()::iterator){
    		...
    }
    ```

    - 방법은 가능하다.
      But, 너무 난잡하고, 직관성이 떨어진다.
      🤧 책에서는 이를 ‘끔찍한 우회 방법’이라고 부른다.

  → 어댑터 메서드를 사용하면 상황은 조금 나아진다.

    ```java
    // Stream<E>를 Iterable<E>로 중개해주는 어댑터
    public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    		return stream::iterator;
    }
    
    // 이제 어떤 스트림도 for-each 문으로 처리할 수 있다.
    
    for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
    		...
    }
    ```


ex) 아나그램 프로그램에서 사전을 읽을 때,

스트림 버전 → Files.lines 메서드 이용
반복 버전 → 스캐너 이용

✅ Files.lines 쪽이 예외 처리 측면에서 더 우수하므로, 반복 버전에도 Files.lines 채택
→ 이때, for-each로 반복하길 원하면 위에 내용이 적용된다.

반대로, Iterable → Stream으로 중개하는 어댑터 또한 구현이 가능하다.

```java
// Iterable<E>를 Stream<E>로 중개해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
		return StreamSupport,stream(iterable.spliterator(), false);
}
```

⚠️ 우리가 메서드의 명확한 의도를 안다면 Stream, Iterable 어느 쪽을 반환하든 상관이 없지만,
→ 공개 API를 작성할 때는 둘 모두를 고려해야 한다.

> *Collection*
>

Collection은 Iterable의 하위 타입이고 stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다

```java
// Collection 파일에 들어가보면
public interface Collection<E> extends Iterable<E> {

		...

		// Stream
		default Stream<E> stream() {
		        return StreamSupport.stream(spliterator(), false);
		}
}
```

→ 원소 시퀀스를 반환하는 공개 API의 반환 타입에는
**Collection**이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.

책에서 “단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다.”
라며 멱집합을 예시로 든다.

```java
// {a, b, c}의 멱집합
{}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c}
```

→ ❌ 이를 몽땅 ArrayList로 넣으면 메모리 낭비

✅ AbstractList를 이용해 전용 컬렉션 구현 가능
→ 원소가 3개 {a, b, c} 있다고 할 때,
→ 부분집합을 나타내려면 → 각 원소를 포함할지(1) 안 할지(0)만 정하면 된다.
( index = 2^n - 1 )

| **인덱스(이진수)** | **비트 표현** | **부분집합** |
| --- | --- | --- |
| 000 (0) | 없음 | {} |
| 001 (1) | c만 포함 | {c} |
| 010 (2) | b만 포함 | {b} |
| … | … | … |

AbstractCollection을 상속하면 **반드시 구현해야 하는 건 딱 2개이다.**

1. iterator() (→ Iterable에서 이미 요구됨)
2. size()
3. contains(Object o)

```java
return new AbstractList<Set<E»() {
		@Override public int size() {
				return 1 << src.size();
		}
				
		@Override public boolean contains(Object o) {
				return o instanceof Set && src.containsAll((Set)o);
		}
		
		@Override public Set<E> get(int index) {
				Set<E> result = new HashSeto();
				for (int i = 0; index != 0; i++, index >>= 1)
						if ((index & 1) = 1)
								result.add(src.get(i));
				return result;
		}
}；
```

👉 이 세 개만 구현하면, 나머지 isEmpty, toArray, addAll 같은 메서드는
AbstractCollection이 기본 구현을 제공해 준다.

⚠️ 이때, **어떤 경우에는 contains나 size를 효율적으로 구현할 수 없다.**
바로, 아래와 같다.
→ ex)
1. 결과 집합이 **동적으로 생성**되는 경우 (ex. 무한 스트림, 게으른 계산)
2. 반복 시작하기 전까지 **원소 개수를 알 수 없는 경우**

👍🏻 이럴 때는, 차라리 `Stream`이나 `Iterable`을 반환하는 편이 낫다.

> 입력 리스트의 모든 연속 부분리스트(sublist)를 반환하는 메서드를 구현하고 싶을 경우
>
1. **Collection 반환
   → 모든 부분리스트를 만들어서 표준 컬렉션(ArrayList 등)에 담아 반환
   → ⚠️ 메모리 이슈**

2. **Stream 반환**
    - ***Prefix-Suffix 아이디어***
        - [a, b, c]에 대해서
            - prefix: [a], [a, b], [a, b, c]
            - suffix: [a, b, c], [b, c], [c]
        - prefix와 suffix 조합으로 모든 부분리스트를 얻는다.

        ```java
        // prefixes와 suffixes를 스트림으로 만들고, flatMap으로 합친다.
        public static <E> Stream<List<E>> of(List<E> list) {
            return Stream.concat(Stream.of(Collections.emptyList()),
                prefixes(list).flatMap(SubLists::suffixes));
        }
        
        private static <E> Stream<List<E>> prefixes(List<E> list) {
            return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
        }
        
        private static <E> Stream<List<E>> suffixes(List<E> list) {
            return IntStream.range(0, list.size())
                .mapToObj(start -> list.subList(start, list.size()));
        }
        ```

        ```java
        // 반복문 버전
        for (int start = 0; start < src.size(); start++)
        		for (int end = start + 1; end <= src.sizeO; end++)
        				System•out.println(src.subList(start, end));
        
        // 스트림 버전
        // 입력 리스트의 모든 부분리스트를 스트림으로 반환한다.
        public static <E> Stream<List<E>> of(List<E> list) {
            return IntStream.range(0, list.size())
                .mapToObj(start ->
                    IntStream.rangeClosed(start + 1, list.size())
                        .mapToObj(end -> list.subList(start, end)))
                .flatMap(x -> x);
        }
        ```


*📂 정리*

스트림 구현은 “코드가 간결하다”는 장점
하지만 반복문보다 읽기 어렵고, 성능도 약간 떨어진다.

책에서 말하길,

🐢 스트림 어댑터 방식은 직접 반복문보다 **2.3배 느림**
💨 직접 전용 Collection 구현은 스트림보다 **1.4배 빠름**, 대신 코드가 훨씬 지저분

만약 나중에 Stream 인터페이스가 Iterable을 지원하게 된다면, 안심하고 Stream을 쓰라고 한다.