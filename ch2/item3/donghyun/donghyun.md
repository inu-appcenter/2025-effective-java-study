# 아이템 3

## private 생성자나 열거 타입으로 싱글턴임을
보증하라

### 싱글턴(Singleton)이란?

싱글턴은 **인스턴스를 오직 하나만 생성할 수 있는 클래스**

- 무상태 함수형 객체
- 설계상 유일해야 하는 시스템 컴포넌트 (데이터베이스 연결 풀, 로그 관리자 등)

### 싱글턴 구현의 3가지 방법

**1. public static final 필드 방식**

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
}
```

**장점:**

- API에서 싱글턴임이 명확하게 드러남
- 간결함

**단점:**

- 리플렉션 공격에 취약???? (AccessibleObject.setAccessible로 private 생성자 호출 가능)

**2. 정적 팩터리 메서드 방식**

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    
    public static Elvis getInstance() {
        return INSTANCE;
    }
    
    public void leaveTheBuilding() { ... }
}
```

**장점:**

- API 변경 없이 싱글턴을 포기할 수 있음 (나중에 스레드별로 다른 인스턴스 반환 가능)
- 제네릭 싱글턴 팩터리로 만들 수 있음
- 메서드 참조를 Supplier로 사용 가능

**3. 열거 타입 방식**

```java
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() { ... }
}
```

**장점:**

- 가장 간결함
- 추가 노력 없이 직렬화 가능
- 리플렉션 공격 완벽 차단
- 복잡한 직렬화 상황에서도 안전

**단점:**

- Enum 외의 클래스를 상속해야 하는 경우 사용 불가

### 직렬화 문제

첫 번째, 두 번째 방식을 사용할 때 직렬화하려면:

1. Serializable 구현
2. 모든 인스턴스 필드를 transient 로 선언
3. readResolve 메서드 제공

```java
private Object readResolve() {
    *// 진짜 Elvis를 반환하고, 가짜 Elvis는 GC에 맡김*
    return INSTANCE;
}
```

**해결 이유**

직렬화, 역직렬화 하고 싶으면 클래스에 Serializable을 구현해야 하는데

```java
직렬화

// 바이트로 변환해서 저장
oos.writeObject(elvis1);

역직렬화
// 바이트를 다시 객체로 복원, 이 때 새로운 Elvis가 만들어짐
Elvis elvis2 = (Elvis) ois.readObject();

따라서 elvis1 != elvis2

하지만 readResolve를 구현하면 
 private Object readResolve() {
    return INSTANCE;  // 가짜가 아니라 진짜를 반환함
}

readResolve는 자동으로 역직렬화할 때 JVM이 실행해줌
```