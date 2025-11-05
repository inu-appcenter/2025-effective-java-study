# 아이템 55

## 옵셔널 반환은 신중히 하라

## 1. isEmpty() vs == null

- String에 사용할 경우
    
    isEmpty()는 String s = “” 일 때만 true
    
    null은 String s = null일 때만 true
    
- Collection에 사용할 경우
    
    isEmpty()는 안에 빈 배열이면 true
    
    null은 항상 false
    

## **2. 자바 8 이전: 값을 반환할 수 없을 때의 문제**

### 방법 1: 예외 던지기

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("빈 컬렉션");
    
    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    }
    return result;
}

try {
    String max = max(emptyList);
} catch (IllegalArgumentException e) {
    // 예외 처리 필요
}
```

**문제점:**

- 예외는 진짜 예외적인 상황에만 사용해야 함
- 스택 추적 캡처로 인한 성능 비용
- 제어 흐름용으로 사용하면 안 됨

---

### 방법 2: null 반환

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        return null;  // null 반환
    
    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0)
            result = e;
    }
    return result;
}

// 사용
String max = max(words);
if (max != null) {  // null 체크
    System.out.println(max.toUpperCase());
}

// null 체크를 안한다면
String max = max(words);
System.out.println(max.toUpperCase());  // NullPointerException
```

**문제점:**

- null 체크를 잊기 쉬움
- null 체크를 빼먹으면 NullPointerException
- 실제 원인과 상관없는 곳에서 예외 발생

## 3. 자바 8 이후: Optional 도입

### Optional 의미

1. 값을 담고 있음 (비지 않았다)
2. 아무것도 담지 않음 (비었다)

```java
Optional<String> optional1 = Optional.of("hello");     // 값 있음
Optional<String> optional2 = Optional.empty();         // 비어있음
optional2.get();
// Exception in thread "main" java.util.NoSuchElementException: No value present

Optional<String> optional3 = Optional.ofNullable(null); // null 허용
optional3.get();
// Exception in thread "main" java.util.NoSuchElementException: No value present
```

```java
public static <E extends Comparable<E>> 
        Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();  // 빈 Optional
    
    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    }
    return Optional.of(result);  // 값이 든 Optional
}

//스트림 버전 
public static <E extends Comparable<E>> 
        Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

---

## 4. Optional 값 꺼내기

### 방법 1: orElse()

값이 있든 없든 실행

```java
String lastWord = max(words).orElse("단어 없음");

public class UserService {
    public String getUserName(Long userId) {
        return findUser(userId) // optional 반환
            .map(User::getName)
            .orElse("Unknown"); // 값이 없으면
    }
   
}
```

### 방법 2: orElseThrow() - 예외 던지기

값이 없으면 nu기

```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

### **방법 3: get() - 값이 있다고 확신할 때**

```java
// 주의: 값이 없으면 NoSuchElementException
Optional<String> empty = Optional.empty();
String value = empty.get();  // NoSuchElementException!
```

### 방법 4: orElseGet()

값이 없을 때만 실행

```java
public String getConfig(String key) {
    return configCache.get(key)
        .orElseGet(() -> {
            // 값이 없을 때만 실행됨
            System.out.println("DB에서 설정 로드 중...");
            return loadConfigFromDatabase(key);
        });
}

```

### **방법5: ifPresent() - 값이 있을 때만 실행**

```java
public void sendEmail(User user) {
    user.getEmail()
        .ifPresent(email -> emailService.send(email, "Welcome!"));
}
```

## 5. Optional의 고급 메서드

### filter() - 조건 필터링

```java
public Optional<User> getAdultUser(Long userId) {
    return findUser(userId) // Optional 반환
        .filter(user -> user.getAge() >= 18); // Optional이 값을 가지고 있으면 내부 값 자동 반환
}
```

### map() - 값 변환

```java
public Optional<String> getUserEmail(Long userId) {
    return findUser(userId)
        .map(User::getEmail);
}
```

### flatMap() - 중첩 Optional 펼치기

```java
public class User {
    private Optional<Address> address;
    
    public Optional<Address> getAddress() {
        return address;
    }
}

public class Address {
    private String city;
    
    public String getCity() {
        return city;
    }
}

// 중첩 Optional
Optional<User> user = findUser(1L);
Optional<Optional<Address>> nested = user.map(User::getAddress);

// 중첩 Optional 풀기
Optional<Address> address = user.flatMap(User::getAddress);

public Optional<String> getUserCity(Long userId) {
    return findUser(userId)
        .flatMap(User::getAddress)
        .map(Address::getCity);
}
```

## **6. Optional 사용 시 주의사항**

컨테이너 타입을 Optional로 감싸지 말 것(리스트 … )

 Map의 값으로 Optional 사용하지 말 것

컬렉션의 키, 값, 원소로 사용하지 말 것

반환값이 Optional 일 때 return null을 하면 안됨

### 성능 고려사항

Optional은 객체 생성 비용이 있음

성능이 중요한 경우 null 반환 고려

### **기본 타입 전용 Optional**

```java
// 안좋은 예시
Optional<Integer> optInt = Optional.of(42);  // int -> Integer -> Optional
Optional<Long> optLong = Optional.of(100L);
Optional<Double> optDouble = Optional.of(3.14);

// 좋은 예시 (전용 Optional 사용)
OptionalInt optInt = OptionalInt.of(42); // int -> OptionalInt
OptionalLong optLong = OptionalLong.of(100L);
OptionalDouble optDouble = OptionalDouble.of(3.14);

// 사용법
OptionalInt max = IntStream.of(1, 2, 3, 4, 5).max();
int value = max.orElse(0);

// Boolean, Byte, Character, Short, Float은 전용 Optional 없음
// 생성하는데 3배 이상 빠름
```

## 7. 핵심 정리

### Optional을 반환해야 할 때

결과가 없을 수 있음

클라이언트가 이 상황을 특별히 처리해야 함

성능이 크게 중요하지 않음

### Optional을 사용하지 말아야 할 곳

컨테이너 타입 (List, Set, Map 등)

컬렉션의 키, 값, 원소

배열의 원소

필드 (대부분의 경우)

메서드 매개변수

> "Optional은 검사 예외와 취지가 비슷하다"
> 
> 
> "박싱된 기본 타입의 Optional 대신 OptionalInt, OptionalLong, OptionalDouble을 사용하라"
>