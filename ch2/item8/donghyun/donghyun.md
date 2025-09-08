# 아이템 8

## finalizer와 cleaner 사용을 피하라

- **finalizer**: 자바 9에서 deprecated된 구식 소멸자
- **cleaner**: 자바 9에서 finalizer 대안으로 도입된 새로운 소멸자

### **finalizer의 문제점**

언제 실행될 지 모름

```java
private void run() {
	A a = new A();
}

---

@Override
protected void finalize() throws Throwable {
    log.info("GC");
}
```

run 메서드가 종료되면 GC가 일어나 finalize가 실행될 것 같지만 언제 실행될지 모른다.

그 이유는 GC를 수행하는 스레드가 따로 있는데 이 스레드의 우선순위가 다른 스레드보다 낮기 때문이다.

**성능문제**

일반 객체보다 50배 느림

**보안문제**

B가 A를 상속하고 있을 때 finalize로 부모 클래스의 멤버들을 탈취할 수 있다.

### **해결방법**

```java
SampleResource implements AutoCloseable {

	@Override
	protected void close() throws Throwable {
	    log.info("GC");
	}
}
```

**try-catch-finally**

```java
try {
	// 메인 코드
	sampleResource = new SampleResource();
	sampleResource.hello();
} finally {
	sampleResource.close(); // 이 때 SampleResource의 GC가 발생
}
```

**try-with-resources(위랑 같음)**

```java
try(SampleResource sampleResource = new SampleResouce()) {
	sampleResource.hello(); 
} // 실행 후 close 실행
```

**cleaner**

```java
package hello.proxy.effective;

import java.lang.ref.Cleaner;

public class SampleResource implements AutoCloseable{

    private boolean closed;
    private static Cleaner CLEANER = Cleaner.create();
    private final Cleaner.Cleanable cleanable;
    private final ResourceCleaner reousrceClaner;

    public SampleResource(){
        this.reousrceClaner = new  ResourceCleaner();
        this.cleanable = CLEANER.register(this, reousrceClaner);
    }

    private static class ResourceCleaner implements Runnable {
        @Override
        public void run() {
            System.out.println("정리 작업");
        }
    }

    @Override
    public void close() throws Exception {
        if(closed){
            throw new IllegalStateException();
        }

        closed = true;
        cleanable.clean();
    }

    public void hello() {
        if (closed) {
            throw new IllegalStateException("리소스가 이미 닫혔습니다!");
        }
        System.out.println("hello() 메서드 실행됨");
    }
}

```

동작순서

try-with-resource에서 실행이 끝나면 SampleResource 의 리소스가 생성되고

자동으로 close를 실행 → clean() 호출