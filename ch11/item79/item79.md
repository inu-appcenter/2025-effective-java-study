# 아이템 79

## 아이템 79: 과도한 동기화는 피하라

동시성을 다룰 때 충분하지 못한 동기화는 데이터 손상이나 안전 실패를 초래하지만 **과도한 동기화** 역시 또 다른 문제를 일으킴

### 문제점

- 성능 저하 :  불필요한 락 경쟁으로 병렬 실행 불가
- 교착상태 : 서로 락을 기다리며 영원히 대기
- 응답 불가 : 스레드가 락을 얻지 못해 멈춤
- 예측 불가능한 동작 : 재진입 중 불변식 깨짐, 데이터 일관성 붕괴

### 해결법

동기화 영역 안에서 제어를 클라이언트에 넘기지 x

동기화 블록(synchronized 메서드) 내부에서는 외부에서 정의한 메서드나 함수 객체를 호출 x

**외계인 메서드 :** 외부에서 들어온 함수나 재정의 가능한 메서드

- 재정의 가능한 메서드
- 클라이언트가 전달한 함수형 객체
- 내부 동작을 알 수 없는 외부 코드

외계인 메서드를 동기화 영역 안에서 호출하면 교착상태 ,ConcurrentModificationException,불변식 파괴 발생 가능

## 문제 상황 예시

### ConcurrentModificationException 발생

```java
public class ObservableSet<E> extends ForwardingSet<E> {
    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element); // 외계인 메서드를 동기화 메서드 내에서 호출하려함
        }
    }
}

```

```java
set.addObserver(new SetObserver<Integer>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23)
            s.removeObserver(this); // 다른 스레드가 락을 얻은 상태여도 다른 스레드가 해당 메서드를 실행하려함
    }
});
```

```java
public class Main {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        // 옵저버 등록
        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println("Added: " + e);
                if (e == 23) {
                    // 리스트 순회 중에 removeObserver() 호출
                    // -> ConcurrentModificationException 발생
                    s.removeObserver(this);
                }
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```

**결과:**

`ConcurrentModificationException` 발생

**교착상태 발생 원인:**

notifyElementAdded을 호출하면서 반복문을 돌릴 때 removeObserver로 리스트를 수정함

내부 count가 바뀌면서 for문의 내부에서 hasNext를 호출할 때 예외 발생

### 교착상태 예시

```java
set.addObserver(new SetObserver<Integer>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23) {
            ExecutorService exec = Executors.newSingleThreadExecutor();
            try {
                exec.submit(() -> s.removeObserver(this)).get();
            } catch (ExecutionException | InterruptedException ex) {
                throw new AssertionError(ex);
            } finally {
                exec.shutdown();
            }
        }
    }
});

```

A 스레드 : removeObserver에서 락을 획득함

get은 락을 획득할려고 대기하는 메서드

→ 락이 있음에도 락을 대기하기 때문에 계속 락을 기다린

다른 스레드는 A 스레드가 락을 획득하고 반환을 안하므로 예외 발생

**해결방법**

방법 1: 외계인 메서드 호출을 동기화 블록 밖으로 이동

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot;
    synchronized (observers) {
        snapshot = new ArrayList<>(observers);  // 복사
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```

락을 오래 잡지 않음

방법2 : CopyOnWriteArrayList 사용

```java
public class ObservableSet<E> extends ForwardingSet<E> {
    private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        observers.add(observer);
    }

    public boolean removeObserver(SetObserver<E> observer) {
        return observers.remove(obsrver); 
            }

    private void notifyElementAdded(E element) {
        for (SetObserver<E> observer : observers)
            observer.added(this, element);
    }
}

```

**성능 및 동시성**

- 락 경쟁으로 인한 대기 시간
    - 여러 스레드가 **동일한 락을 동시에 요청**하면, 하나만 락을 얻고 나머지는 대기 상태에 들어감
    - **스레드가 락을 기다리느라 작업이 지연**
- 병렬성 손실 (멀티코어 병렬 실행 불가)
    - 과도한 동기화는 여러 스레드가 **서로 다른 CPU 코어에서 병렬로 실행되는 것을 막음**
- 메모리 일관성 지연 (모든 코어가 메모리 상태를 동기화해야 함)
    - CPU 캐시 간 데이터 동기화로 인해 추가적인 성능 오버헤드 발생
    - 

**가변 클래스 설계 시 동기화 전략**

외부에서 동기화(`ArrayList`, `HashMap`)

클래스 자체는 동기화하지 않고, 사용자가 필요할 때 외부에서 락을 걸도록 함

내부에서 동기화(`ConcurrentHashMap`, `BlockingQueue`)

클래스가 자체적으로 스레드 안전성을 보장

**잘못된 내부 동기화의 해결**

StringBuffer : 대부분 단일 스레드 환경에서 사용되지만 내부 동기화 수행

→StringBuilder

Random : 불필요한 동기화로 경쟁 발생

→ThreadLocalRandom

**정적 필드 동기화 주의**

정적 필드는 여러 스레드가 공유하므로, private여도 사실상 전역 변수처럼 동작

```java
private static int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;  // 동기화 보장 안함
}

private static int nextSerialNumber = 0;

public static synchronized int generateSerialNumber() {
    return nextSerialNumber++;
}

```

**해결**

```java
private static final AtomicInteger nextSerialNumber = new AtomicInteger(0);

public static int generateSerialNumber() {
    return nextSerialNumber.getAndIncrement();
}
```

- 락이 없음
- 내부적으로 CAS 연산을 사용하여 원자적 증가를 보장
- synchronized 보다 훨씬 빠르고, 멀티코어 환경에서 병렬성 유지