<aside>

공유 중인 가변 데이터는 동기화해 사용하라

</aside>

### 동기화의 두 가지 기능

많은 개발자들이 동기화를 ‘한 번에 한 스레드만 실행되도록 보장하는 것’으로만 생각한다. 하지만 동기화에는 중요한 기능이 하나 더 있다. 바로 ‘스레드 간 통신’이다. 

1. **배타적 실행**
    
    한 스레드가 객체를 수정하는 동안 다른 스레드가 일관되지 않은 상태를 보지 못하도록 막는다. (lock을 건다)
    
2. **스레드 간 통신**
    
    한 스레드가 만든 변화를 다른 스레드가 볼 수 있도록 보장한다.
    

⇒ 동기화는 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론이고, 같은 락의 보호하에 수행된 모든 수정의 최종 결과를 보게 해주기도 하는 것.

```java
// 동기화 없이는 이런 일이 발생할 수 있다
public class DataRaceExample {
    private static boolean ready = false;
    private static int number = 0;
    
    public static void main(String[] args) {
        // Reader 스레드
        new Thread(() -> {
            while (!ready) {
                Thread.yield();
            }
            System.out.println(number);  // 0이 출력될 수도 있다!
        }).start();
        
        // Writer 스레드
        number = 42;
        ready = true;
    }
}
```

위 코드는 42를 출력할 것 같지만, 동기화가 없으면 0을 출력하거나 영원히 종료되지 않을 수 있다. Writer 스레드가 값을 변경했지만, Reader 스레드가 그 변경을 보지 못하는 것이다... 이는 자바의 메모리 모델 때문이라고 한다. (스레드는 성능을 위해 변수를 CPU 캐시에 저장해두는데, 동기화가 없으면 한 스레드가 메인 메모리에 쓴 값이 다른 스레드의 캐시에 반영되지 않을 수 있다. )

### 1. 가변 데이터 공유 시 읽기/쓰기 모두 동기화

자바 언어 명세상 long과 double 외의 변수를 읽고 쓰는 동작은 **원자적**이다. 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 뜻이다.

**🤔💭 그럼 원자적인 경우엔 동기화 쓰지 않는 게 성능상 이득이겠네?** 

😣✋ 아니다… 자바 언어 명세는 스레드가 필드를 읽을 때 ‘수정이 완전히 반영된' 값을 얻는다고 보장하지만, ‘한 스레드가 저장한 값이 **다른 스레드에게 보이는가**'는 보장하지 않는다! 

```java
public class StopThread {
    private static boolean stopRequested;
    
    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
            }
        });
        
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

위는 스레드를 멈추는 코드이다. 1초 후에 끝날 것으로 예상되지만, 끝나지 않고 계속 실행된다. 이것도 동기화 때문이다.

```java
// 원래 코드
while (!stopRequested) {
    i++;
}

// JVM이 최적화한 코드
if (!stopRequested) {
    while (true) {  // 어차피 안 바뀌는 것 같으니 한 번만 체크
        i++;
    }
}
```

동기화가 빠진 상황에서, JVM은은 위와 같은 최적화를 수행할 수 있다. 이건 끌어올리기(hoisting) 최적화라고 한댜. `stopRequested`가 바뀌지 않는 것 같으니 반복문 밖으로 검사를 끌어올려 버린다... 

이런 문제를 해결하려면, **읽기와 쓰기 둘 다를 동기화해야 한다.**  쓰기만 동기화하면 읽는 쪽이 변경을 못 볼 수 있고, 읽기만 동기화하면 쓴 값이 다른 스레드에 전달되지 않을 수 있으니 하나만 해서는 소용이 없다. 

```java
public class StopThread {
    private static boolean stopRequested;
    
    // 쓰기 메서드
    private static synchronized void requestStop() {
        stopRequested = true;
    }
    
    // 읽기 메서드
    private static synchronized boolean stopRequested() {
        return stopRequested;
    }
    
    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested()) {  // 동기화된 메서드로 읽기
                i++;
            }
        });
        
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        requestStop();  // 동기화된 메서드로 쓰기
    }
}
```

이 코드에서 synchronized는 **스레드 간 통신 목적으로만** 쓰였다. `stopRequested` 메서드들은 단순해서 동시에 실행되어도 문제없지만, 변경사항을 다른 스레드에게 알리기 위해 동기화가 필요한 것이다.

### 2. 스레드 간 통신만 필요하다면 volatile 사용

배타적 실행은 필요 없고 통신만 필요한 경우, 더 간단한 방법이 있다.

```java
public class StopThread {
    private static volatile boolean stopRequested;
    
    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
            }
        });
        
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

`volatile` 키워드는 다음을 보장한다.

- 항상 메인 메모리에서 최신 값을 읽는다 (캐시 무시)
- JVM이 최적화를 하지 않는다 (끌어올리기 방지)
- 다른 스레드에게 즉시 보인다 (메모리 가시성 보장)

물론 여기에도 주의점이 있다.

```java
// 일련번호 생성
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

코드는 매번 고유한 번호를 반환할 것 같지만, 중복 발생의 위험이 있다. 

**volatile은 복합 연산에 사용할 수 없다는 한계가 있다.**

++ 연산자는 하나의 동작처럼 보이지만, 실제로는 세 단계를 거친다.

```java
1. nextSerialNumber 값을 읽는다
2. 1을 더한다
3. 결과를 nextSerialNumber에 쓴다

// Thread 1: 값 0을 읽음
// Thread 2: 값 0을 읽음  ← 여기서 같은 값 읽음!
// Thread 1: 1로 증가시켜 저장
// Thread 2: 1로 증가시켜 저장  ← 중복 발생!
```

두 스레드가 동시에 읽기 단계를 수행하면 같은 값을 읽어서 중복된 번호를 생성하게 된다. 이런 오류를 **안전 실패**라고 한다. (Item 48에서도 잘못된 병렬화를 이야기하면서 언급함) 

### 3. 복합 연산에는 Atomic 클래스

**🤔💭** 아니 그러면… 락 없이도 스레드가 안전하면서 원자성을 보장하고 성능도 synchronized 좋을 수는 없나? 

😃👍 `java.util.concurrent.atomic` 패키지의 **AtomicLong**을 사용하자 (Item 59)

```java
import java.util.concurrent.atomic.AtomicLong;

private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

Atomic 클래스는 CAS(Compare-And-Swap) 연산을 사용한다. CAS는 현재 값이 예상한 값이면 새 값으로 바꾸고, 아니면 실패하는 CPU 명령어다. 실패하면 재시도하는 방식으로 락 없이도 동시성을 제어한다.

```java
// Atomic 클래스들
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();  // 증가 후 값 반환 (원자적)
counter.getAndIncrement();  // 값 반환 후 증가 (원자적)
counter.compareAndSet(0, 1);  // 0이면 1로 변경 (CAS)
counter.addAndGet(5);  // 5 더하고 결과 반환

AtomicBoolean flag = new AtomicBoolean(false);
flag.compareAndSet(false, true);  // false면 true로 변경

AtomicReference<User> userRef = new AtomicReference<>();
userRef.set(new User("Alice"));
User old = userRef.getAndSet(new User("Bob"));  // 교체하고 이전 값 반환
```

### 4. 근본적인 해결책은 가변 데이터를 공유하지 않는 것…

공유한다면 불변 데이터만, 가변 데이터는 단일 스레드에서만 사용는 것이 가장 좋은 해결책이다.

```java
// 불변 객체는 동기화 필요 없음
public final class Point {
    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    // 상태를 변경하는 대신 새 객체를 반환
    public Point move(int dx, int dy) {
        return new Point(x + dx, y + dy);
    }
}

```

```java
//  ThreadLocal로 각자 쓰기(공유x -> 동기화 필요x)
private static final ThreadLocal<SimpleDateFormat> formatter = 
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

public static String formatDate(Date date) {
    return formatter.get().format(date);  // 각 스레드가 독립된 인스턴스 사용
}
```

```java
// 한 스레드에서 완전히 초기화 후 공유
public class Cache {
    private static volatile Map<String, Integer> data;
    
    public static void initialize() {
        // 1. 새 맵 만들고 완전히 초기화 (단일 스레드에서만)
        Map<String, Integer> newData = new HashMap<>();
        newData.put("key1", 1);
        newData.put("key2", 2);
        
        // 2. 완성된 맵을 한 번에 발행
        data = newData;
        
        // 3. 이후 data를 다시 수정하지 않으면
        //    다른 스레드들은 동기화 없이 읽을 수 있음
    }
    
    public static Integer get(String key) {
        return data != null ? data.get(key) : null;
    }
}
```

### 정리

1. 여러 스레드가 가변 데이터를 공유하면 읽기/쓰기 모두 동기화해야 한다.
2. 배타적 실행 없이 스레드 간 통신만 필요하다면 volatile를 쓸 수 있다.
3. 복합 연산에는 Atomic 클래스를 사용하자.
4. 가장 좋은 방법은 가변 데이터를 공유하지 않는 것이다.
