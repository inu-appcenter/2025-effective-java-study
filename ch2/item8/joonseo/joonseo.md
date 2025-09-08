# Item08

> ***finalizer와 cleaner 사용을 피하라***

<br>

## finalizer & cleaner

**Java**에서는 **finalizer**와 **cleaner**라는 소멸자를 제공한다. 

<br>

**finalizer**는 예측할 수 없고 상황에 따라 위험할 수 있다. 몇 가지 사용 방식이 있지만 기본적으로 쓰지 않는게 원칙이다. 결국 **Java** **9**에서는 **finalizer**를 **deprecated**로 지정하였다. 

<br>

그 대안으로 소개되는 **cleaner**는 **finalizer**보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

<br>

## 소멸자

 **소멸자**란 객체에 할당된 메모리가 해제되기 직전에 호출되는 메서드로, 객체가 생명주기 동안 얻은 리소스를 해제하는 역할을 한다. 그중 **finalizer**는 가비지 컬렉터가 객체에 더이상 접근할 수 없다고 판단할 때 실행된다. 

 <br>

 중요하고 필수적으로 보이지만, 특정 시점이나 실행 자체가 보장되지 않으며, 특정 시점에 실행되는 작업은 finalizer와 cleaner로 는 절대 할 수 없다. 그리고 이 실행 시점에 대한 책임은 가비지 컬렉터 구현에 달려있음

 <br>

소멸자는 중요하고 필수적이다. 하지만 **finalizer**와 **cleaner**는 많은 단점을 갖고 있다.



- 성능이 나쁘다.(= 속도가 느리다.)
- 특정 시점에 실행됨이 보장되지 않으며 실행 조차도 그렇다.
- **finalizer** 동작 중에 발생한 예외는 무시되고, 처리할 작업이 남아 있어도 그 순간 종료된다.
- **finalizer**는 **finalizer** 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
    
    → 생성자나 직렬화 과정에서 예외가 발생하면 하위 클래스 마음대로 **finalizer**를 수행할 수 있다.
    
    → 이걸 막기 위해서 **final**이 아닌 클래스에 대해 아무 일도 하지 않는 **finalizer**을 **final**로 선언 해야 한다.

<br>

 따라서, 상태를 영구적으로 수정하는 작업이나 특정 시점을 보장해야 하는 작업에서는 절대 **finalizer**나 **cleaner**에 의존하면 안된다.

 <br>

## 그럼 언제 쓰나요

 이렇게 단점이 많은 finalizer와 cleaner는 언제 어디에 쓰일까?

 <br>

### 안전망

 혹시라도 자원의 소유자가 **close** 메서드를 호출하지 않을 때를 대비해 사용한다. 아예 자원이 회수되지 않는 것보다 늦게라도 회수되는게 좋으니.(~~좋은게 맞는지 옳은 설계인지는 잘 모르겠다.~~)

 <br>

대표적으로 **FileInputStream**, **FileOutputStream**, **ThreadPoolExecutor**가 이 **finalizer**를 제공한다.

<br>

### 네이티브 피어

 네이티브 피어란, **일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체**를 말한다. 코드로 쉽게 이해해보자. 

 <br>

**네이티브 메서드**

```java
public class Example {
    // 네이티브 메서드 선언
    public native void nativeMethod();
    
    static {
        System.loadLibrary("example"); // C/C++ 라이브러리 로드
    }
}
```

<br>

**자바 피어와 네이티브 피어**

```java
// 자바 피어
public class GraphicsRenderer {
    private long nativeHandle; // 네이티브 객체를 가리키는 포인터
    
    public GraphicsRenderer() {
        nativeHandle = createNativeRenderer(); // 네이티브 피어 생성
    }
    
    public void render() {
        nativeRender(nativeHandle); // 네이티브 피어에게 작업 위임
    }
    
    // 네이티브 메서드들
    private native long createNativeRenderer();
    private native void nativeRender(long handle);
    private native void destroyNativeRenderer(long handle);
}
```

<br>

네이티브 피어는 자바 피어가 인터페이스만 제공하고 실제 작업은 다른 외부 라이브러리가 처리하는 방식이다. 하지만, 자바 객체가 사라져도 외부 라이브러리 관련 인스턴스는 메모리에 남아있다. 

<br>

 가비지 컬렉터가 이것들을 처리하지 못하기 때문에 **finalizer**나 **cleaner**를 통해 직접 처리할 수 있다. 단, 성능 저하를 감당할 수 있고 네이티브 피어가 중요한 자원을 갖고 있지 않을 때만 유효하다.

<br>

### cleaner를 통해

**cleaner**를 사용하는 경우 보통 **AutoCloseable** 인터페이스를 구현한다. 바로 아이탬의 코드 예시로 살펴보자

<br>

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    
    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    private static class State implements Runnable {
        int numJunkPiles; // 방(Room) 안의 쓰레기 수
        
        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }
        
        // close 메서드나 cleaner가 호출한다.
        @Override 
        public void run() {
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }
    
    // 방의 상태. cleanable과 공유한다.
    private final State state;
    
    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;
    
    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }
    
    @Override 
    public void close() {
        cleanable.clean();
    }
}
```

 **State** 클래스는 정리해야 할 대상을 필드로 갖고 있으며, 이 대상은 오버라이딩하고 있는 **run** 메소드에 의해 정리된다. 

 <br>

 **run** 메소드가 호출되는 경우는 다음과 같다

- 개발자가 의도적으로 close 메소드를 호출하는 경우
- 가비지 컬렉터가 Room을 회수할 때까지 close가 호출되지 않는 경우

<br>

<aside>

❗️위 예시에서 **State** 인스턴스는 절대로 **Room**의 인스턴스를 참조하면 안된다.

---

외부 클래스의 인스턴스를 참조하면 **순환참조**가 생겨 가비지 컬렉터가 **Room** 인스턴스를 회수할 수 없다.
→ 따라서 이를 막기 위해 **State** 클래스를 **static**으로 선언

</aside>

<br>

만약 아래처럼 **Room** 객체를 **try-with-resources** 블록으로 감쌌다면 자동 청소는 전혀 필요하지 않다. 

```java
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```
