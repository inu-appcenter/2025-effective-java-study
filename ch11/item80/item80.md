# Item 80

> ***스레드보다는 실행자, 태스크, 스트림을 애용하라***

<br>

## Concurrent

 **concurrent** 패키지가 등장하기 전에는 백그라운드 스레드를 직접 관리해야 했다. 뿐만 아니라 **안전실패**, **응답불가** 상태로 빠질 수 있는 여지들을 개발자들이 직접 찾아 이를 방지하는 코드를 작성했어야 했다.

<br>

 **concurrent** 패키지가 등장하면서 스레드를 유연하게 관리할 수 있고 작업에 활용할 수 있게 되었다. 이 패키지에서는 실행자 프레임워크라고 하는 인터페이스 기반의 태스크 실행 기능을 담고 있다.

<br>

```java
ExecutorService exec = Executors.newSingleThreadExecutor();
```

<br>

이 한줄로 작업 큐를 쉽게 생성할 수 있게 되었다. 만약, 둘 이상의 스레드로 작업을 처리하게 하고 싶다면, 아래처럼 스레드 풀을 갖는 실행자를 할당해주면 된다.

<br>

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
```

<br>

실행부인 **execute** 메서드는 **Runnable** 타입을 파라메터로 받는다. **Runnable**은 **표준 함수형 인터페이스**이기 때문에 직접 구현하여 객체를 넘겨줘도 되고, 람다식을 넘겨줘도 된다. 

<br>

**Runnable 인터페이스를 구현하는 객체를 넘겨주는 방식**

```java
public class Test {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        executor.execute(new SayHello());

    }

    public static class SayHello implements Runnable {
        @Override
        public void run() {
            System.out.println("Hello");
        }
    }
}
```

<br>

**람다식을 넘겨주는 방식**

```java
for(int i = 0; i < maxThread; i++){
    final long studentId = i + 1L;
    executorService.execute(() -> {
        try{
            registrationService.registerCourseWithPessimisticReadLock(studentId, courseId);
        } catch (CannotAcquireLockException e) {
            deadLockCount.getAndIncrement();
            System.out.println("Deadlock occurred for student: " + studentId);
        }
        finally {
            countDownLatch.countDown();
        }
    });
}
```

<br>

## 실행자 서비스

실행자 서비스는 이 외에도 다양한 기능을 제공한다. 

<br>

### 특정 태스크가 완료될 때까지 대기

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

// 나중에 할당될 값을 Future<V>로 받고
Future<String> future = executor.submit(() -> {
    Thread.sleep(2000);
    return "작업 완료";
});

// **get()** -> Future<V> 객체에서 V를 가져오는 메서드
String result = future.get();

executor.shutdown();
```

<br>

### 태스크 모음 중 하나 혹은 모든 태스크가 완료될 때까지 대기

```java
ExecutorService executor = Executors.newFixedThreadPool(3);

List<Callable<String>> tasks = Arrays.asList(
    () -> {
        Thread.sleep(3000);
        return "서버1 응답";
    },
    () -> {
        Thread.sleep(1000);
        return "서버2 응답";
    },
    () -> {
        Thread.sleep(2000);
        return "서버3 응답";
    }
);

// 1. 가장 먼저 처리된 태스크만 사용하는 경우 -> **invokeAny()**
String fastest = executor.invokeAny(tasks);

// 2. 모든 태스크가 완료될 때까지 대기하는 경우 -> **invokeAll()**
List<Future<Integer>> futures = executor.invokeAll(tasks);
```

<br>

### 실행자 서비스가 종료될 때까지 대기

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

// **submit()** -> 작업큐에 작업을 넣는 메서드
for (int i = 0; i < 5; i++) {
    final int taskId = i;
    executor.submit(() -> {
        Thread.sleep(1000);
        System.out.println("작업 " + taskId + " 완료");
    });
}

// **awaitTermination()** -> 지정된 시간 내에 모든 작업이 완료되면 **true**, 아니면 **false**
if (executor.awaitTermination(10, TimeUnit.SECONDS)) {
    System.out.println("모든 작업 정상 완료");
} else {
    System.out.println("타임아웃 - 강제 종료 시도");
    executor.shutdownNow();
}
```

<br>

### 완료된 태스크의 결과를 차례로 받기

```java
ExecutorService executor = Executors.newFixedThreadPool(3);
ExecutorCompletionService<String> completionService = 
    new ExecutorCompletionService<>(executor);

completionService.submit(() -> {
    Thread.sleep(3000);
    return "작업1 완료";
});
completionService.submit(() -> {
    Thread.sleep(1000);
    return "작업2 완료";
});
completionService.submit(() -> {
    Thread.sleep(2000);
    return "작업3 완료";
});

// **take()** -> 완료된 작업을 바로 가져옴(시간이 오래 걸리는 작업에 영향받지 않음)
for (int i = 0; i < 3; i++) {
    Future<String> future = completionService.take();
    String result = future.get();
    System.out.println(result);
}

executor.shutdown();
```

<br>

### 태스크를 특정 시간, 주기별로 실행

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

// **scheduleAtFixedRate()** -> 고정주기 실행(첫번째 파라메터는 초기지연, 두번째 파라메터는 지연 주기)
ScheduledFuture<?> future = scheduler.scheduleAtFixedRate(() -> {
    System.out.println("실행: " + LocalTime.now());
}, 1, 2, TimeUnit.SECONDS);
```

<br>

## ThreadPoolExecutor

상황에 맞춰 스레드 풀을 커스텀하고 싶다면, **ThreadPoolExecutor**를 직접 사용하면 된다. 이 클래스는 스레드 풀에 대한 다양한 옵션을 제공한다.

```java
public ThreadPoolExecutor(
    int corePoolSize,           // 1. 기본 스레드 수
    int maximumPoolSize,        // 2. 최대 스레드 수
    long keepAliveTime,         // 3. 유휴 스레드 생존 시간
    TimeUnit unit,              // 4. 시간 단위
    BlockingQueue<Runnable> workQueue,  // 5. 작업 큐
    ThreadFactory threadFactory,        // 6. 스레드 생성 팩토리
    RejectedExecutionHandler handler    // 7. 거부 정책
)
```

<br>

<aside>

**📌 CachedThreadPool은 주의하여 사용하자!**

---

**CachedThreadPool**은 태스크를 큐에 쌓지 않고 즉시 처리할 스레드에 위임하게 되는데, 가용 스레드가 없다면 스레드를 하나 생성하여 위임한다. 

만약, 작업의 크기가 무겁고 스레드 풀 사이즈가 작다면 스레드가 지속해서 생성될 것이고 최악의 경우 시스템이 죽어버릴 수도 있다.

따라서, 무거운 작업을 처리하는 경우에는 **고정 스레드풀**을 사용하는 것이 낫다.

</aside>

<br>
<br>

<aside>

**📌 Runnable과 Callable**

---

작업 단위를 나타내는 핵심 추상 개념이 태스크인데, **Runnable**과 **Callable**로 구현할 수 있다.

**Callable**은 **Runnable**과는 다르게 반환값을 가질 수 있으며 예외도 던질 수 있다. 

</aside>

<br>
<br>

<aside>

**📌 포크-조인**

---

**Java 7**에서는 실행자 프레임워크는 **포크-조인 태스크**를 지원하도록 확장되었다. 이 태스크는 **포크-조인 풀**이라는 특별한 실행자 서비스가 실행해주며, 작업을 먼저 마친 스레드가 남은 태스크를 가져와 대신 처리할 수도 있다. 

**포크-조인 태스크**를 직접 작성하는 것은 어렵지만, **포크-조인 풀**을 활용해 개발된 **병렬 스트림**을 이용하면 더 적은 노력으로 그 이점을 누릴 수 있다. 

</aside>
