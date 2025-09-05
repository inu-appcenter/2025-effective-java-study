# Item01

> ***생성자 대신 정적 팩터리 메서드를 고려하라***

<br>

## 정적 팩터리 메서드란

 **정적 팩터리 메서드**(static factory method)란 클래스 내부에 **static**으로 정의된 메소드를 지칭하는 용어로, A.create()처럼 클래스 외부에서 사용할 수 있는 특징을 갖고 있다. 

<br>

## 왜 정적 팩터리 메서드를 권장할까

 본 아이템에서는 생성자를 직접 사용해서 객체를 생성하는 것을 지양하고, 정적 팩터리 메서드를 통한 객체 생성을 권장하고 있다. 권장하는 이유를 5가지 장점을 통해 설명하고 있는데 코드와 함께 확인해 보았다.

<br>

### 1. 이름을 가질 수 있다.

 일단 팩터리 ‘메서드’이므로 클래스 내에서 고유한 이름을 가질 수 있다. 이름은 어떤 행위를 하는지 기술하고 있으며 이를 통해 코드의 가독성과 유지보수 용이성에 기여할 수 있다.

 <br>

**생성자를 직접 사용하는 경우**

```java
public class User {
		// 필드
    
    // public 생성자
    public User(String name, String email, boolean isActive) {
        this.name = name;
        this.email = email;
        this.isActive = isActive;
    }
}
```

```java
// 비지니스 로직에서 사용 시
User activeUser = new User("홍길동", "hong@email.com", true);
User inactiveUser = new User("김철수", "kim@email.com", false);
```

<br>

**팩토리 메서드를 사용하는 경우**

```java
public class User {
		// 필드
		
		// private 생성자
    private User(String name, String email, boolean isActive) {
        this.name = name;
        this.email = email;
        this.isActive = isActive;
    }

    // 비활성 사용자 생성
    public static User createInactiveUser(String name, String email) {
        return new User(name, email, UserType.NORMAL, false);
    }

    // 활성 사용자 생성
    public static User createActiveUser(String name, String email) {
        return new User(name, email, UserType.NORMAL, true);
    }
}
```

```java
// 비지니스 로직에서 사용 시
User activeUser = User.createActiveUser("홍길동", "hong@email.com");
User inactiveUser = User.createInactiveUser("김철수", "kim@email.com");
```

<br>

### 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

 객체를 생성할 때, 생성자를 통해 생성하게 되면 생성할 때마다 새로운 객체가 생성된다. 물론 기능 요구사항에 따라 다르지만, 재할당이 꼭 필요하지 않은 경우도 존재한다.

<br>

 아래 Boolean 클래스를 보면 Boolean 클래스 타입으로 TRUE, FALSE 필드가 각각 생성되어 있고 생성자를 통해 true, false를 각각 할당하고 있다.

<br>

 이 Boolean 클래스는 첫 참조가 일어나는 시점에 초기화되어 힙 메모리에 할당된다. 그리고 뒤이어 정적 팩토리 메서드로 선언된 valueOf가 호출되면, 매개변수의 값을 검사하고 **첫 참조 때 초기화된 TRUE나 FALSE를 반환**하게 된다.

<br>

```java
public final class Boolean {

    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);
    
    private final boolean value;
    
    public static Boolean valueOf(boolean b) {
        return b ? TRUE : FALSE;
    }
}
```

<br>

 만약 **valueOf** 메소드가 정적 메소드가 아니라면 Boolean을 반환하기 위해 호출될 때마다 **새로운 객체를 생성해 값을 반환**해야 했을 것이다. 참조가 빈번하지 않고 필드값이 정적이라면, 정적 팩토리 메소드를 활용하는 방식도 컴퓨팅 리소스 관점에서 유리한 전략이 될 수 있을 것 같다.

<br>

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

 이 능력은 다형성을 조금 더 유연하게 활용할 수 있는 이점을 제공한다. 아이탬의 예시로, Collections 클래스는 내부적으로 **45개의 정적 클래스**와 이를 반환하는 **정적 팩토리 메서드**로 구성되어 있다.

```java
public class Collections {

    public static <T> List<T> unmodifiableList(List<? extends T> list) {
        return new UnmodifiableList<>(list);
    }
    
    static class UnmodifiableList<E> implements List<E> {
    }
}
```

<br>

**List** 인터페이스를 구현하는 **UnmodifiableList** 객체를 생성할 때, 이를 반환하는 정적 팩토리 메서드 **Collections.unmodifiableList()**를 통해 생성할 수 있다. 

<br>

개발자들은 **UnmodifiableList** 클래스가 어떻게 구현되어 있는지 몰라도 List 인터페이스 대로 동작할 것 임을 알고 있기에 사용하는데 문제가 없을 것이다. 

<br>

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

정적 팩토리 메소드를 통해 매개변수 입력에 따라 어떤 타입의 객체를 반환할지 결정할 수 있다. 아이탬에서 언급한 **EnumSet** 클래스를 한번 살펴보자. 

```java
public abstract class EnumSet<E extends Enum<E>> {
    
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }
}
```

<br>

**noneOf** 메소드에서는 매개변수의 크기에 따라 **RegularEnumSet**을 반환할 수도, **JumboEnumSet**을 반환할 수도, 예외를 반환할 수도 있다. 

<br>

추후에 더 큰 입력에 대해 더 효율적인 클래스가 개발되면 개발자의 코드 수정 없이 이 효율성을 제공할 수 있다. 

<br>

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환될 객체의 클래스가 존재하지 않아도 된다.

 즉, 컴파일 시점에는 구체적인 구현체가 존재하지 않아도 되며, 런타임에 동적으로 로드할 수 있다는 것을 의미한다. 이러한 특징은 서비스 제공자 프레임워크를 구성하는 근간이 된다. 

 <br>

서비스 제공자 프레임워크는 아래 4가지 키워드를 정의한다.

- **서비스 인터페이스**
    
    → 구현체의 동작을 정의한 인터페이스
    
- **서비스 접근 API**
    
    → 적절한 구현체를 가져와 반환하는 API
    
- **제공자 등록 API**
    
    → 구현체를 등록하는 API
    

- **서비스 제공자 인터페이스**
    
    → 서비스 인터페이스의 인스턴스를 생성하는 팩토리 객체를 설명

<br>
    
개념 자체가 추상적이라 아이탬에서 예시로 든 **JDBC**를 통해 이해해보자.

<br>

**서비스 인터페이스**

```java
public interface Connection {
    Statement createStatement();
    void close();
}
```

<br>

**서비스 접근 API**

```java
public class DriverManager {
    public static Connection getConnection(String url) {
        for (Driver driver : registeredDrivers) {
            if (driver.acceptsURL(url)) {
                return driver.connect(url, new Properties());
            }
        }
        throw new SQLException("No suitable driver found");
    }
}
```

<br>

**제공자 등록 API**

```java
public class DriverManager {
    public static void registerDriver(Driver driver) {
	    // Driver를 등록
    }
}
```

<br>

**서비스 제공자 인터페이스**

```java
public interface Driver {
    Connection connect(String url, Properties info);
    boolean acceptsURL(String url);
}
```

 먼저, **제공자 등록 API**를 통해 데이터베이스 유형별 드라이버(**서비스 제공자 인터페이스**)가 등록된다. 이후, **서비스 접근 API**를 통해 데이터베이스 url이 주어졌을 때 이를 처리할 수 있는 드라이버를 가져온다. 그리고, 그 드라이버가 구현한 **connect** 메소드를 통해 실질적인 **Connection**을 가져오게 된다. 

<br>

 만약, 일반 생성자나 일반 인스턴스 메서드를 사용했다면 실제 객체를 생성해야하며, 클라이언트는 구체적 구현체를 알고 있어야 하는 단점이 있다.

<br>

## 정적 팩터리 메서드의 단점은

### 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

 상속하기 위해서는 부모 클래스의 public/protected 생성자가 필요한데, **private**으로 생성자 접근을 막으면 상속을 할 수 없게 된다.

<br>

 예시로 **Collections** 클래스도 생성자를 private로 제한하고 있어 상속이 불가능하다. 

<br>

하지만, 이 단점은 상속보다 컴포지션을 사용하도록 유도하며 불변 객체로 만들기 위해서는 이 제약을 지켜야한다는 점에서 장점으로도 볼 수 있다.

<br>

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

정적 팩토리 메서드의 장점이었던 이름을 통해 메서드의 동작을 기술하는 것의 단점이라고도 볼 수 있다. 개발자마다 사용하는 이름이 달라 docs에서 찾기 힘들다. 

<br>

대신 개발자들이 널리 사용하는 네이밍 컨벤션이 존재한다.

- **from**
    
    → 매개변수 하나를 받아 해당 타입의 인스턴스를 반환하는 메소드
    
    ```java
    User user = User.from("Han Joon Seo");
    ```

<br>
    

- **of**
    
    → 여러 매개변수를 받아 적절한 타입의 인스턴스를 반환하는 메소드
    
    ```java
    Teacher teacher = Job.of("PE", "29");
    ```

<br>
    

- **valueOf**
    
    → **from**과 **of**의 더 자세한 버전
    
    ```java
    BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
    ```

<br>
    

- **instance/getInstance**
    
    → 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는 메소드
    
    ```java
    StackWalker Luke = StackWalker.getlnstance(options);
    ```

<br>
    

- **create/newInstance**
    
    → **instance**/**getInstance**와 동일하지만, 매번 새로운 인스턴스 반환을 보장
    
    ```java
    Object newArray = Array.newInstance(classObject, arrayLen);
    ```

<br>

- **getType/newType**
    
    → **getInstance/newInstance**와 같지만 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의할 때 사용.

<br>

- **type**
    
    → **getType**과 **newType**의 간결한 버전
