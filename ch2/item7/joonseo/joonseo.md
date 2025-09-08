# Item07

> ***다 쓴 객체 참조를 해제하라***

<br>

## 익숙함에 속지말자

 **C**와 **C++** 같은 언어에서는 할당된 메모리에 대해 이를 해제하는 것은 개발자의 몫이었다. **Java**는 **가비지 컬렉터**를 갖추고 있기 때문에 그 몫에서 자유로워질 수 있었다.

 <br>

 하지만, 완전히 자유로워진 것은 아니다. 가비지 컬렉터는 **다 쓴 객체**를 알아서 회수해가는데, **다 쓴 객체**는 무엇인지에 대한 초점을 맞춰본다.

<br>

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
        if (size == 0)  
            throw new EmptyStackException();
        return elements[--size];
    }
    
    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)  // = 를 == 로 수정
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

<br>

 위 코드는 아이탬에서 제공한 스택을 구현한 코드이다. 스택은 똑똑하게 새로운 원소를 넣기 전 배열의 자유 공간을 체크한 뒤, 부족하다면 그 크기를 2배 확장한다. 

 <br>

 2배, 4배, 8배 확장하고 원소를 채우며 공간을 활용한 후, pop 메소드가 연쇄적으로 호출되어 확장된 공간이 더이상 쓰이지 않는다면, 그 공간은 어떻게 될까?

 <br>

 **가비지 컬렉터**는 해당 공간을 더이상 사용하지 않는 공간이라 인지하지 못한다. 그 이유는 **활성화되어 있는 스택이 여전히 그 참조를 가지고 있기 때문**이다. 

 <br>

따라서, 일반적으로 자기 메모리를 직접 관리하는 클래스라면 사용하지 않는 자원에 대한 해제를 잘 관리해주어야 한다. 위 예시에서 적용할 수 있는 몇가지 방법은 아래와 같다.

<br>

- **pop** 메소드가 호출되면 그 공간을 **null**로 처리한다.
- 연결 리스트로 스택을 구현하여, 연결이 끊어졌을 때 해당 참조를 **null**로 처리한다.

<br>

## 항상 경각심을 가져야 하는가

 방금 전, 자기 메모리를 직접 관리하는 클래스라면, 사용하지 않는 자원은 해제에 대한 처리를 해줘야한다고 했지만, **객체 참조에 대한 null 처리는 대단히 예외적인 경우**여야 한다. 

 <br>

아이탬에서 권장하는 방식은 참조 변수의 **스코프**를 활용하는 방법이다.

<br>

**스코프를 적절히 활용하지 못한 경우**

```java
public class BadExample {
    private List<String> cache = new ArrayList<>(); // 클래스 레벨
    
    public void process() {
        // 임시 데이터를 클래스 변수에 저장
        for (int i = 0; i < 1000; i++) {
            cache.add("data" + i);
        }
        
        // 처리 완료 후에도 cache가 계속 메모리에 남아있음
        doSomething();
    }
}
```

<br>

**스코프를 적절히 활용한 경우**

```java
public class GoodExample {
    public void process() {
        List<String> cache = new ArrayList<>(); // 지역 변수
        
        for (int i = 0; i < 1000; i++) {
            cache.add("data" + i);
        }
        
        // 메서드 종료시 cache 자동 해제
        doSomething();
    }
}
```

<br>

## 대단히 예외적인 경우

### 캐시

일반적인 **HashMap**을 캐시로 사용한다면, key에 대한 참조를 해제해도 여전히 캐시에 남아있게 된다. 이런 경우에는 **WeakHashMap**을 사용해보자. **key**에 대한 참조가 해제되면 캐시 앤트리에서도 자동으로 제거해주는 기능을 제공한다.

<br>

```java
// HashMap
Map<Key, Value> cache = new HashMap<>();
Key key = new Key("data");
cache.put(key, new Value());
key = null; // key의 참조가 해제되도 캐시 엔트리에 남아 있음

// WeakHashMap
Map<Key, Value> cache = new WeakHashMap<>();
Key key = new Key("data");
cache.put(key, new Value());
key = null; // key의 참조가 해제되면 캐시 엔트리도 자동 제거
```
<br>

### 리스너/콜백

개발자가 리스너와 콜백을 등록만 하고 명확하게 해제하지 않는다면 계속 쌓여만 갈 것이다. 등록 시, **약한 참조**로 등록하게 되면 가비지 컬렉터가 GC되는 순간 해제하기 때문에 자원 누수를 줄일 수 있다.

<br>

```java
public class EventManager {
    private List<EventListener> listeners = new ArrayList<>();
    
    public void addListener(EventListener listener) {
        listeners.add(listener);
    }
}

public class EventManager {
    private Map<EventListener, Boolean> listeners = new WeakHashMap<>();
    
    public void addListener(EventListener listener) {
        listeners.put(listener, Boolean.TRUE);
    }
}
```
