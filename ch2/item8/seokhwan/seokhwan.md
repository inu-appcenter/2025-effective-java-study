# Item 8

추가 일시: 2025년 9월 5일 오후 5:27
강의: Effective JAVA

## 🍀 Item 8: finalizer와 cleaner 사용을 피하라

---

### Java의 소멸자

---

Java는 두 가지 객체 소멸자를 제공한다.

finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.

cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

**finalizer와 cleaner는 즉시 수행된다는 보장이 없다.**

즉시 수행되지 않고 언제 실행될 지 모른다는 것, 이는 가비지 컬렉터의 알고리즘에 따라 실행 시점이 달라진다는 것이다.

→ 클래스에 finalizer를 달아두면 해당 인스턴스의 자원 회수가 제멋대로 지연될 수 있다. finalizer 대기열에서 회수되기만 기다리다 OutOfMemoryError를 발생시키고 애플리케이션이 죽을 수 있다.

수많은 문제점들…

> 상태를 영구적으로 수정하는 작업에서 사용하지 마라.
성능 문제도 동반한다.
finalizer를 사용한 클래스는 finalizer 공격에 노출되어 보안에 심각한 문제가 생길 수 있다.
객체 생성을 막기 위해 예외를 던져도 finalizer가 있다면 문제가 생길 수 있다.
> 

→ 그렇다면 왜 만들어둔거지?

자원 소유자가 close 메서드를 호출하지 않는 것에 대한 안전망이다.

즉, 자원회수를 안 하는 것보다는 늦게라도 하는것이 낫다.

두 번째, 네이티브 피어와 연결된 객체에서의 활용

<aside>
💡

네이티브 피어

일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체
→ Java 객체와 연결된 네이티브 코드(C,C++)로 구현된 객체

</aside>

```java
//실제 메모리는 네이티브 힙 영역에 할당
//Java 객체는 100 bytes 할당
ByteBuffer buffer = ByteBuffer.allocateDirect(100_000_000); //네이티브 메모리 100MB
```

가비지 컬렉터는 Java 객체의 크기만 고려하기 때문에 네이티브의 메모리를 고려하지 않는다.

때문에 실제로는 많은 메모리가 낭비되는 상황에 가비지 컬렉터가 Heap 메모리의 정리가 급하지 않다고 판단 할 수 있다.

이를 finalizer와 cleaner를 통해 안전망 역할을 하는 것이다.

### 그럼 대안으로 무엇이 있나요?

---

AutoCloseable을 구현하고, 클라이언트에서 인스턴스를 사용후 close 메서드를 호출하면 된다.

```java
class ManagedResource implements AutoCloseable {
    @Override
    public void close() {
	    if (!closed) {
		    closed = true; //상태 플래그 설정
        // 실제 정리 로직
        resource = null; //참조 제거
	    }
    }
}
```

다음은 수동으로 close를 호출한다.

더 좋은 방식은 try-with-resource 블록으로 감싸는 것이다.

```java
try (ManagedResource resource = new ManagedResource(1024)) {
    resource.doWork();
} // 자동으로 close() 호출됨
```

이는 예외가 발생하거나, early return이 발생하는 상황에도 close()를 호출한다.