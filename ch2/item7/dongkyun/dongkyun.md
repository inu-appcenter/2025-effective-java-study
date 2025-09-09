# item7

## 🗑️ 다 쓴 객체 참조를 해제하라

---

```java
public class Stack {
		private Object[] elements;
		private int size = 0;
		private static final int DEFAULT_INITIAL_CAPACITY = 16;
		
		public Stack() {
				elements = new Object[DEFAULT_INITIAL_CAPACITY];
		}
		public void push(Object e) {
				ensureCapacity();
				elements[size++] = e;
		}
		public Object pop() {
				if (size = 0)
						throw new EmptyStackException();
				return elements[—size];
		}
		/* 
		* 원소를 위한 공간을 적어도 하나 이상 확보한다.
		* 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
		*/
		private void ensureCapacity() {
				if (elements.length = size)
						elements = Arrays.copyOf(elements, 2 * size + 1);
		}
}
```

- 이 코드에서는 스택이 커졌다가 줄어들었을 때, 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않습니다. (더이상 프로그램에서 그 객체들을 사용하지 않아도)
    - 스택들이 다 쓴 참조를 여전히 가지고 있기 때문
- 가비지 컬렉션 언어에서는 (의도치 않게 객체를 살려두는) 메모리 누수를 찾기가 아주 까다롭습니다.
    - 객체 참조 하나를 살려두면 그 객체가 참조하는 모든 객체를 회수하지 못합니다.
- 해법은 간단합니다.
    - 해당 참조를 다 썼을 때, null 처리, 즉 참조 해제하면 됩니다.

```java
// 제대로 구현한 pop 메서드
public Object pop() {
		if (size = 0)
				throw new EmptyStackException();
		Object result = elements[——size];
		elements [size] = null; // 다 쓴 참조 해제
		return result;
}
```

- 추가적인 이점도 있습니다.
    - null 처리한 참조를 실수로 접근하면 NullPointerException을 던지며 종료됩니다.

      (오류 발견)


- 그러나 모든 객체를 쓰자마자 null 처리하는 것은 바람직하지 않습니다.
    - 객체 참조를 null 처리하는 것은 예외적인 경우여야 합니다.
- 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것입니다.

- 그렇다면 null 처리는 언제 하나요?
    - **자기 메모리를 직접 관리하는 클래스**일 경우 주의할 필요가 있습니다.
    - Stack은 자기 메모리를 직접 관리하므로 가비지 컬렉터가 메모리 누수 사실을 알 길이 없습니다.
        - 프로그래머는 비활성 영역이 되는 순간 null 처리할 필요가 있습니다.

    - **캐시** 역시 메모리 누수의 주범입니다.
        - 캐시에 넣은 객체를 다 쓰고 잊는 경우가 이에 해당합니다.
        - 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아있는 캐시가 필요할 경우
          → WeakHashMap을 이용해 캐시를 만드는걸 추천합니다.
            - 다 쓴 엔트리는 즉시 자동으로 제거됩니다.
            - 단, 이러한 상황에서만 유용합니다.
        - 캐시를 만들 때, 캐시 엔트리의 유효 기간을 정확히 정의하기 어렵기 때문에,
          시간이 갈 수록 엔트리의 가치를 떨어뜨리는 방식을 흔히 사용합니다.
            - 이따금, 쓰지 않는 엔트리를 청소해야합니다.

            ```java
            class CacheExample {
                private Map<String, Object> cache = new WeakHashMap<>();
            
                public void put(String key, Object value) {
                    cache.put(key, value);
                }
            
                public Object get(String key) {
                    return cache.get(key);
                }
            }
            
            key = null; // 이제 WeakHashMap에서 자동 제거 가능
            System.gc(); // 가비지 컬렉션 발생 시 엔트리 제거
            ```

    - 마지막 주범은 **리스너** 혹은 **콜백**입니다.
        - 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않으면, 계속 쌓입니다.
            - 이때, 콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해갑니다.
            - ex) WeakHashMap에 키로 저장하는 방법이 있습니다.

            ```java
            class EventSource {
            		// WeakReference<EventListener>를 쓰면
            		// → 약한 참조
                private List<WeakReference<EventListener>> listeners = new ArrayList<>();
            
                public void register(EventListener listener) {
                    listeners.add(new WeakReference<>(listener));
                }
            }
            
            listener = null; // 더 이상 참조하지 않음
            System.gc(); // 가비지 컬렉션 후 약한 참조 수거
            ```