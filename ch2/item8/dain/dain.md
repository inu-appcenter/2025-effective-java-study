# Item8

<aside>

finalizer와 cleaner 사용을 피하라

</aside>

자바는 객체를 자동으로 정리하는 finalizer와 cleaner를 제공한다. 그런데 예측할 수도 없고, 느린데다가, 위험하기까지 하다고…

따라서 아래와 같은 해결책을 알아둬야 한다.

AutoCloseable을 사용하여 해결

```java
@Override
    public void close() throws IOException {
        if (!closed) {
            try {
                studentFile.close();
            } finally {
                reportFile.close();
            }
            closed = true;
        }
    }
```

```java
public void processStudentData() {
    try (StudentFileProcessor processor = new StudentFileProcessor("students.txt", "report.txt")) {
        processor.processStudents();
        // 자동으로 close() 호출됨
    } catch (IOException e) {
        System.err.println("파일 처리 중 오류: " + e.getMessage());
    }
}
```

cleaner는 안전망 역할로만 사용해야 함 → 클라이언트가 close()를 깜빡했을 때의 마지막 보험인 것.

```java
public class PizzaOven implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    
    // static 중첩 클래스 (바깥 클래스 참조 방지)
    private static class OvenState implements Runnable {
        int temperature;
        
        OvenState(int temperature) {
            this.temperature = temperature;
        }
        
        @Override
        public void run() {
            System.out.println("안전망 작동: 오븐 온도를 " + temperature + "도에서 0도로 냉각");
            temperature = 0;
        }
    }
    
    private final OvenState state;
    private final Cleaner.Cleanable cleanable;
    
    public PizzaOven(int temperature) {
        state = new OvenState(temperature);
        cleanable = cleaner.register(this, state);  // 안전망 등록
    }
    
    public void bakePizza() {
        System.out.println("피자를 " + state.temperature + "도에서 굽는 중...");
    }
    
    @Override
    public void close() {
        cleanable.clean();  // 수동으로 정리
    }
}
```