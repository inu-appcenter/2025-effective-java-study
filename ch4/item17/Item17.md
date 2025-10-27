# 아이템 17

# 변경 가능성을 최소화하라

## 불변 클래스

인스턴스 생성 후 내부 값을 수정할 수 없는 클래스

String, Integer 등등

### 불변 클래스 규칙

1. setter x
2. 클래스 확장 불가(final 클래스)
3. 모든 필드 final
4. 모든 필드 private
5. 내부 가변 필드 접근 차단(방어적 복사)

### 불변 객체 장점

1. 단순함 : 생성 시점 상태를 유지
2. 스레드 안전 : 동기화 없이 여러 스레드에서 안전하게 사용 가능
3. 자유로운 공유 : 따로 방어적 복사 불필요
    
    ```java
    ImmutableUser user1 = new ImmutableUser("김철수", 25);
    ImmutableUser user2 = user1;
    ```
    
4. 실패 안전성 : 예외 발생 시에도 객체 유지
5. Map 키, Set 원소로 사용 : 값이 변하지 않으므로 적적

### 불변 객체 단점

값이 다르면 계속 새 객체를 만들어야 함

해결책 

1. 다단계 연산을 기본으로 제공 (BigInteger)
2. 가변 동반 클래스 제공(StringBuilder → String)

## 가변 클래스 예시

```java
public class MutablePoint {
    public double x;
    public double y;
    
    public MutablePoint(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public void move(double dx, double dy) {
        this.x += dx;
        this.y += dy;
    }
    
    public void setX(double x) {
        this.x = x;
    }
}

-----------

MutablePoint point = new MutablePoint(10, 20);
System.out.println(point.x);  // 10

// 다른 곳에서 실수로 변경함
point.setX(50);
point.move(5, 5);

System.out.println(point.x); // 55
```

```java
public class MutableUser {
    private String name;
    private int age;
    private List<String> hobbies;
    
    public MutableUser(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        this.hobbies = hobbies;
    }
    
    public void setName(String name) { this.name = name; }
    public void setAge(int age) { this.age = age; }
    public List<String> getHobbies() { return hobbies; }
}

-----------

List<String> myHobbies = new ArrayList<>();
myHobbies.add("독서");
myHobbies.add("운동");

// 외부 변경이 객체 내부에도 영향
MutableUser user = new MutableUser("김철수", 25, myHobbies);
myHobbies.add("게임"); // myHobbies, user 모두 바뀜
user.getHobbies().clear();

// 가변 클래스에서 멀티스레드일 때 동시성 문제 발생 예시
Thread thread1 = new Thread(() -> user.setAge(26));
Thread thread2 = new Thread(() -> user.setAge(27)); // 멀티스레드일 때 동시에 접근 시 어떤 값이 될지 예측 불가능함

thread1.start();
thread2.start();

sleep(3000);

System.out.println(user.getAge()); // 26 or 27
```

## 불변 클래스 예시

```java
public final class ImmutablePoint {
    private final double x;
    private final double y;
    
    public ImmutablePoint(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
    
    // 기존 객체는 변경 없이 값만 읽어서 새로운 객체 생성
    public ImmutablePoint move(double dx, double dy) {
        return new ImmutablePoint(x + dx, y + dy);
    }
}

-----------

ImmutablePoint point1 = new ImmutablePoint(10, 20);
ImmutablePoint point2 = point1.move(5, 5);

System.out.println(point1.getX());  // 10
System.out.println(point2.getX());  // 15
```

```java
public final class ImmutableUser {
    private final String name;
    private final int age;
    private final List<String> hobbies;
    
    public ImmutableUser(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        this.hobbies = new ArrayList<>(hobbies); // 방어적 복사해서 리턴
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    // 복사해서 리턴
    public List<String> getHobbies() { 
        retun new ArrayList<>(hobbies); 
    }
    
    // 변경사항을 반영해서 새로운 객체 리턴
    public ImmutableUser withAge(int newAge) {
        return new ImmutableUser(this.name, newAge, this.hobbies);
    }
    
    // 변경사항을 반영해서 새로운 객체 리턴
    public ImmutableUser addHobby(String hobby) {
        List<String> newHobbies = new ArrayList<>(this.hobbies);
        newHobbies.add(hobby);
        return new ImmutableUser(this.name, this.age, newHobbies);
    }
}

-----------

List<String> hobbies = new ArrayList<>();
hobbies.add("독서");
hobbies.add("운동");

ImmutableUser user1 = new ImmutableUser("김철수", 25, hobbies);

hobbies.add("게임"); // user1에는 영향 x
System.out.println(user1.getHobbies().size());  // 2

// user1의 hobbies를 정리해도 위의 hobbies에는 영향 없음
user1.getHobbies().clear();
System.out.println(user1.getHobbies().size());  // 2

// 변경 시 새 객체 생성
ImmutableUser user2 = user1.withAge(26);
System.out.println(user1.getAge());  // 25
System.out.println(user2.getAge());  // 26
```

## 함수형 프로그래밍 스타일

### 가변객체

```java
public class MutableCalculator {
    private double result;
    
    public MutableCalculator(double initial) {
        this.result = initial;
    }
    
    public void add(double value) {
        result += value
    }
    
    public void multiply(double value) {
        result *= value;
    }
    
    public double getResult() {
        return result;
    }
}

----------------

MutableCalculator calc = new MutableCalculator(10);
calc.add(5);
calc.multiply(2);
System.out.println(calc.getResult());  // 30
```

### 불변객체

```java
public final class ImmutableCalculator {
    private final double value;
    
    public ImmutableCalculator(double value) {
        this.value = value;
    }
    
    public ImmutableCalculator add(double amount) {
        return new ImmutableCalculator(value + amount);
    }
    
    public ImmutableCalculator multiply(double factor) {
        return new ImmutableCalculator(value * factor);
    }
    
    public double getValue() {
        return value;
    }
}

-------------

// 메서드 체이닝
double result = new ImmutableCalculator(10)
    .add(5)
    .multiply(2)
    .getValue(); // 30

// 연산 중간단계의 step도 따로 저장
ImmutableCalculator step1 = new ImmutableCalculator(10);
ImmutableCalculator step2 = step1.add(5);
ImmutableCalculator step3 = step2.multiply(2);

System.out.println(step1.getValue());  // 10
System.out.println(step2.getValue());  // 15
System.out.println(step3.getValue());  // 30
```

불변 객체는 각 연산이 새 객체를 반환하므로, 중간 결과를 안전하게 보관하고 재사용

## String vs StringBuilder

```java
// 일반 String
String result = "";
for (int i = 0; i < 10000; i++) {
    result += i;  // 계속 새 String 객체 생성
}

// StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);  // 추가하기
}
String result = sb.toString();  // 마지막에 String으로 변환
```

## 핵심 정리

1. 기본적으로 불변
2. setter x
3. 컬렉션은 방어적 복사
4. 성능을 고려하면 StringBuilder와 같은 가변 동반 클래스 사용
5. JPA에서 엔티티는 가변으로 해야하고, dto는 불변으로 해야함