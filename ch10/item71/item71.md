# 아이템 71

### **검사 예외 (Checked Exception)**

```java

// throws 선언 하면
public static int divide(int a, int b) throws ArithmeticException {
    return a / b;
}

// 메서드 사용할려면 try-catch로 잡아야함
public static void main(String[] args) {
    try {
        divide(10, 0);
    } catch (Exception e) {
        System.out.println(e); // java.lang.ArithmeticException: / by zero
    }
}
```

검사예외 : 예외가 발생했을 때 try-catch로 잡아야함

**특징**

1. 사용자에게 부담
    - catch 블록 처리
    - 바깥으로 던지기해야함 (전파)
2. 스트림에서 사용 불가
3. 하나만 예외 검사할 때 과함
    - try 블록 추가해야하는 번거로움
    - 스트림 사용 불가
    

try 블록 추가해야하는 번거로움 예시

```java
public static void process1() {
    try {
        divide(10, 0);
    } catch (ArithmeticException e) {}

}

public static void process2() {
    try {
        divide(10, 0);
    } catch (ArithmeticException e) {}
}
```

**스트림에서는 사용하기 힘든 예시**

```java
public static void main(String[] args) {
    List<String> userIds = Arrays.asList("1", "2", "3");

    List<User> users = userIds.stream()
            .map(id -> findById(id)) // java: unreported exception java.lang.Exception; must be caught or declared to be thrown
													           // 예외를 던지기 때문에 사용 불가
            .toList();
}

public static User findById(String id) throws Exception {
    // DB 조회 로직
}
```

→ 억지로 스트림 사용

```java
public static void main(String[] args) {
    List<String> userIds = Arrays.asList("user1", "user2", "user3");

    List<User> users = userIds.stream()
            .map(id -> {
                try {
                    return findById(id);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            })
            .toList();
}
```

억지로 할려면 하는데 가독성이 너무 떨어짐

### 비검사 예외

```java
    public static void main(String[] args) {
        divide(10, 0);
    }

    public static int divide(int a, int b) {
        return a / b;
    }
    
    Exception in thread "main" java.lang.ArithmeticException: / by zero
	at item71.item71_1.divide(item71_1.java:10)
	at item71.item71_1.main(item71_1.java:6)
```

비검사 예외 : 예외를 그냥 던짐(예외 처리를 선택적)

**특징**

1. 예외 처리 선택적
2. 코드 짧음
3. 런타임에 예외 발생한다는 치명적 문제(난 이것 때문에 비검사 예외 안할듯)

**검사예외를 사용해야하는 경우**

1. API를 제대로 사용해도 예외가  발생할 수 있을 때
2. try-catch로 잡아서 어떤 조치(로그, 프린트 등)을 해야할 때

## 해결방법

### 1. optional 사용

**before: 검사 예외 사용**

```java
public static void main(String[] args) {
    try {
        User user = findById(1L);
        System.out.println(user.getName());
    } catch (Exception e) {
        System.out.println("사용자를 찾을 수 없습니다.");
    }
}

public User findById(Long id) throws Exception {
    User user = userRepository.findById(id);
    if (user == null) {
        throw new Exception("User not found: " + id);
    }
    return user;
}
```

**after: Optional 사용**

```java
public static void main(String[] args) {
    Optional<User> user = findById(1L);
    user.ifPresentOrElse(
            u -> System.out.println(u.getName()),
            () -> System.out.println("사용자를 찾을 수 없습니다.")
    );
}

public Optional<User> findById(Long id) throws Exception {
    User user = userRepository.findById(id);
    return Optional.ofNullable(user);
}

```

장점 : 간결함

단점 : 예외 발생이유 제공 x(try-catch처럼 print(e)를 할 수 없기 때문)

### 2. 메서드 분리

**before: 검사 예외 방식**

```java
// 인출 코드
public void processPayment(int account, int amount) throws InsufficientBalanceException {
}

try {
    processPayment(100000, 1000);
} catch (Exception e) {  // 예외 처리 필수, 안하면 컴파일 에러
    System.out.println("잔액 부족");
}
```

**after: 비검사 예외 방식(메서드 분리)**

```java
// 인출 코드
public static boolean canProcessPayment(int account, int amount) {
    return account > amount;
}
public static void processPayment(int account, int amount) {
    if (canProcessPayment(account, amount)) {
        throw new IllegalStateException("잔액 부족"); // throws 선언 안해도됨
    }
}

processPayment(1, 1); // 컴파일 오류 안남

비검사로 하면 try-catch가 강제되지 않음

```

장점(선택 사항이 많음)

```java
// 1. if로 상태 먼저 확인하고 비즈니스 로직 실행
if (canProcessPayment(account, amount)) {
    processPayment(account, amount);
    System.out.println("결제 성공");
} else {
    System.out.println("잔액 부족");
}

// 2. 예외가 안 날 거라고 확신하고 그냥 비즈니스 로직 실행
processPayment(account, amount);

// 3. 선택적으로 예외 처리
try {
    processPayment(account, amount);
    System.out.println("결제 성공");
} catch (IllegalStateException e) {
    System.out.println("잔액 부족: " + e.getMessage());
}
```

**메서드 분리에서 주의사항**

1. 멀티스레드 환경
    
    ```java
    if (canProcessPayment(account, amount)) { 
    // 이 때 동시성 오류 발생 가능성 있음
    // A 스레드는 if문을 통과하고 비즈니스로직을 실행하기 전에 B 스레드가 동시성을 
    // 무시하고 비즈니스 로직을 실행할 수 있음
        processPayment(account, amount);  
    }
    ```
    
    → 해결책 : 동시성 해결(synchronized, CAS 등등)
    

1. 중복 작업으로 인한 성능 문제
    
    ```java
    public boolean canProcessPayment(Account account, Money amount) {
        return expensiveValidation(account, amount); 
    }
    
    public void processPayment(Account account, Money amount) {
        if (!canProcessPayment(account, amount)) {  // 중복 검증
            throw new IllegalStateException("잔액 부족");
        }
        // 결제 처리
    }
    ```
    
    → 해결책 : 이거는 그냥 코드를 잘 짜면 되는거라서 중복을 없애면 됨
    

## 결론

기본은 비검사 예외

해당 값이 있을수도 있고, 없을 수도 있으면 Optional

실패 이유가 중요하면 검사예외