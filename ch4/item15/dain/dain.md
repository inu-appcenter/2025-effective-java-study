<aside>

클래스와 멤버의 접근 권한을 최소화하라

</aside>

정보 은닉을 강조하는 장이다. 잘 설계된 컴포넌트는 내부 구현을 완벽히 숨겨 구현과 API를 깔끔히 분리한다. → 클래스와 멤버의 접근성을 가능한 한 좁혀라.

```java
// 잘못된 예시 - 내부 구현이 노출됨
public class BadStack {
    public Object[] elements;  // 내부 배열이 public
    public int size;           // 크기도 public
    
    public void push(Object e) {
        // 클라이언트가 직접 elements[size++] = e 할 수 있음
        ensureCapacity();
        elements[size++] = e;
    }
    
    private void ensureCapacity() { /* ... */ }
}

// 올바른 예시 - 내부 구현 숨김
public class GoodStack {
    private Object[] elements;    // private으로 숨김
    private int size = 0;         // private으로 숨김
    private static final int DEFAULT_CAPACITY = 16;
    
    public GoodStack() {
        elements = new Object[DEFAULT_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0) throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 메모리 누수 방지
        return result;
    }
    
    public int size() { return size; }
    
    public boolean isEmpty() { return size == 0; }
    
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
    
    public static void main(String[] args) {
        GoodStack stack = new GoodStack();
        stack.push("첫 번째");
        stack.push("두 번째");
        
        System.out.println("크기: " + stack.size());           // 2
        System.out.println("팝: " + stack.pop());              // 두 번째
        System.out.println("비어있나: " + stack.isEmpty());     // false
        
        // stack.elements[0] = "해킹";  // 컴파일 에러, 접근 불가
        // stack.size = 100;            // 컴파일 에러, 접근 불가
    }
}
```

### public 상수 활용법

```java
public class Constants {
    // 올바른 예시 - 불변 객체 참조
    public static final String COMPANY_NAME = "TechCorp";
    public static final int MAX_USERS = 1000;
    public static final BigDecimal INTEREST_RATE = new BigDecimal("0.05");
    
    // 잘못된 예시 - 가변 배열 노출
    public static final String[] BAD_VALUES = {"A", "B", "C"}; // 위험!
    
    // 올바른 해결책 1 - 불변 리스트
    private static final String[] PRIVATE_VALUES = {"A", "B", "C"};
    public static final List<String> VALUES = 
        Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
    
    // 올바른 해결책 2 - 방어적 복사
    public static String[] getValues() {
        return PRIVATE_VALUES.clone();
    }
    
    public static void main(String[] args) {
        // BAD_VALUES의 문제점
        System.out.println("변경 전: " + Arrays.toString(BAD_VALUES));
        BAD_VALUES[0] = "HACKED"; // 외부에서 변경 가능!
        System.out.println("변경 후: " + Arrays.toString(BAD_VALUES));
        
        // VALUES는 안전함
        System.out.println("불변 리스트: " + VALUES);
        // VALUES.set(0, "HACK"); // UnsupportedOperationException 발생
        
        // 방어적 복사는 매번 새 배열 반환
        String[] copy1 = getValues();
        String[] copy2 = getValues();
        copy1[0] = "수정됨";
        System.out.println("원본: " + Arrays.toString(PRIVATE_VALUES)); // [A, B, C]
        System.out.println("복사본: " + Arrays.toString(copy1));         // [수정됨, B, C]
    }
}
```

### 계층 구조에서의 접근 제어

```java
// 기본 클래스
public class Vehicle {
    private String engine;           // 하위 클래스에서도 접근 불가
    protected String brand;          // 하위 클래스에서 접근 가능
    String model;                   // 같은 패키지에서 접근 가능
    public int year;                // 어디서든 접근 가능
    
    protected void startEngine() {
        System.out.println(brand + " 엔진 시동");
    }
    
    // private 메서드는 하위 클래스에서 재정의 불가
    private void checkEngine() {
        System.out.println("엔진 점검");
    }
}

// 하위 클래스
class Car extends Vehicle {
    public Car(String brand, String model, int year) {
        // this.engine = "V6";     // 컴파일 에러, private 접근 불가
        this.brand = brand;        // OK - protected
        this.model = model;        // OK - package-private (같은 패키지)
        this.year = year;          // OK - public
    }
    
    @Override
    protected void startEngine() { // 접근 수준을 더 좁게 할 수 없음
        super.startEngine();
        System.out.println("자동차 시동 완료");
    }
    
    // 더 넓은 접근 수준으로는 재정의 가능
    public void startEngine2() { // protected -> public 가능
        System.out.println("공개 시동 메서드");
    }
    
    public static void main(String[] args) {
        Car car = new Car("현대", "소나타", 2024);
        car.startEngine();
        
        System.out.println("브랜드: " + car.brand); // protected 접근
        System.out.println("모델: " + car.model);   // package-private 접근
        System.out.println("연도: " + car.year);    // public 접근
    }
}
```

### 테스트에서는…

```java
// 프로덕션 코드
public class Calculator {
    private double result;
    
    public double add(double a, double b) {
        return result = a + b;
    }
    
    public double subtract(double a, double b) {
        return result = a - b;
    }
    
    // 테스트를 위해 private을 package-private으로 완화 (허용됨)
    double getInternalResult() { // 원래는 private이었지만 테스트용으로 완화
        return result;
    }
    
    // 하지만 public까지는 안 됨
    // public double getInternalResult() { // 과도한 노출
}

// 테스트 코드 (같은 패키지)
class CalculatorTest {
    public static void main(String[] args) {
        Calculator calc = new Calculator();
        
        double sum = calc.add(10, 5);
        assert sum == 15;
        
        // 테스트를 위해 내부 상태 확인 가능
        assert calc.getInternalResult() == 15; // package-private 접근
        
        System.out.println("모든 테스트 통과!");
    }
}
```

### 정리

- **최소 권한 원칙**: 꼭 필요한 것만 공개
- **public 클래스**: 상수(public static final) 외에는 public 필드 금지
- **가변 객체**: 절대 public으로 노출하지 말 것
- **배열 필드**: public으로 노출하면 보안 허점, 불변 리스트나 방어적 복사 활용
- **테스트**: package-private까지는 허용, public은 금지