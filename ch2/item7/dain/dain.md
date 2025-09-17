# Item7

<aside>

다 쓴 객체 참조를 해제하라

</aside>

자바는 가비지 컬렉터가 있어서 메모리를 자동으로 관리해주지만, 완벽하지 않다. 우리가 실수로 객체 참조를 계속 들고 있으면, 가비지 컬렉터가 그 객체를 회수하지 못해 **메모리 누수가 발생**한다.

```java
public class StudentStack {
    private Student[] students;
    private int size = 0;
    
    public Student pop() {
        if (size == 0) throw new RuntimeException("스택이 비어있음");
        return students[--size];  // 배열에 참조가 남아있음!!
    }
```

위 코드를 보면… pop()으로 학생을 꺼낸 후에도 배열에 참조가 남아 있다. 이런 상황에서 가비지 컬렉터는 ‘아직 참조되고 있구나’하고 메모리를 회수하지 않는다.

```java
public class StudentStack {
    private Student[] students;
    private int size = 0;
    
    public Student pop() {
        if (size == 0) throw new RuntimeException("스택이 비어있음");
        Student result = students[--size];
        students[size] = null;  // 다 쓴 참조를 null로 처리
        return result;
    }
```

다 쓴 참조를 **null처리**했다면, 객체 **참조를 해제**한 것이다.

그렇다고 해서 모든 변수를 일일이 null 처리할 필요는 없다. 자기 메모리를 직접 관리하는(내부에 배열이나 컬렉션으로 저장소를 만들어 관리하는) 클래스에서만 null 처리를 하자.

→ 이 경우 어떤 영역이 사용 중이고 어떤 영역이 사용되지 않는지를 클래스만이 알고 있다.

```java
public class PizzaOrder {
    private Pizza[] orders = new Pizza[100];  // 피자 주문 저장소
    private int activeCount = 0; 
    
    public void cancelOrder() {
        if (activeCount > 0) {
            Pizza canceledPizza = orders[--activeCount];
            orders[activeCount] = null;  // 취소된 주문 참조 해제
            System.out.println(canceledPizza.getName() + " 주문 취소");
        }
    }
}
```

메모리 누수의 주범으로 3가지 언급된다.

1. 자기 메모리를 직접 관리하는 클래스
    - 스택, 큐, 리스트 등의 자료구조
2. 캐시(Cache)

    ```java
    public class BookCache {
        private Map<String, Book> cache = new HashMap<>();
        
        public Book getBook(String isbn) {
            Book book = cache.get(isbn);
            if (book == null) {
                book = loadBookFromDatabase(isbn);
                cache.put(isbn, book);  // 캐시에서 제거하지 않으면 계속 쌓임
            }
            return book;
        }
    }
    ```

    ```java
    // 해결책 1 : WeakHashMap 사용
    public class BookCache {
        // WeakHashMap-> 키가 다른 곳에서 참조되지 않으면 자동 제거
        private Map<String, Book> cache = new WeakHashMap<>();
        
        public Book getBook(String isbn) {
            // (...)
            return cache.computeIfAbsent(isbn, this::loadBookFromDatabase);
        }
    }
    ```

    ```java
    //해결책 2: 주기적 정리
    public class BookCache {
        private Map<String, Book> cache = new LinkedHashMap<String, Book>() {
            @Override
            protected boolean removeEldestEntry(Map.Entry<String, Book> eldest) {
                return size() > 100;  // 100개 넘으면 가장 오래된 것 제거
            }
        };
    }
    ```

3. 리스너와 콜백

    ```java
    public class StudentNotifier {
        private List<StudentListener> listeners = new ArrayList<>();
        
        public void addListener(StudentListener listener) {
            listeners.add(listener);
        }
        
        // 리스너를 제거하는 메서드가 없으면 계속 쌓임
        public void removeListener(StudentListener listener) {
            listeners.remove(listener);  // 이런 메서드가 필요함
        }
    }
    ```

    ```java
    //해결책 : 약한 참조 사용
    public class StudentNotifier {
        private Map<StudentListener, Object> listeners = new WeakHashMap<>();
        
        public void addListener(StudentListener listener) {
            listeners.put(listener, new Object());  // 더미 객체
        }
        
        // 리스너가 다른 곳에서 참조되지 않으면 자동으로 제거됨
    }
    ```