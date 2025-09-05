# Item02

> ***생성자에 매개변수가 많다면 빌더를 고려하라***

## 빌더 패턴이 필요한 이유

 선택적 매개변수가 많을 때, 단순 생성자나 정적 팩토리 메소드로는 적절히 대응하기가 어렵다. 왜냐하면, 어떤 매개변수가 입력으로 들어올지 모르기 때문에, 경우의 수 별로 생성자나 정적 팩토리 메서드를 만들어야 하기 때문이다. 

<br>

 개발자들은 이러한 문제에 대해 **점층적 생성자 패턴** 또는 **자바 빈스 패턴**을 주로 사용했다.

<br>

**점층적 생성자 패턴**

```java
public class User {
    private final String name;           // 필수
    private final String email;          // 필수
    private final int age;               // 선택적
    private final String phoneNumber;    // 선택적
    private final String address;        // 선택적

    public User(String name, String email) {
        this(name, email, 0);
    }

    public User(String name, String email, int age) {
        this(name, email, age, null);
    }

    public User(String name, String email, int age, String phoneNumber) {
        this(name, email, age, phoneNumber, null);
    }

    public User(String name, String email, int age, String phoneNumber, String address) {
        this.name = name;
        this.email = email;
        this.age = age;
        this.phoneNumber = phoneNumber;
        this.address = address;
    }
}
```

<br>

 **점층적 생성자 패턴**은 매개변수를 1개 받는 생성자, 2개 받는 생성자, 3개 받는 생성자… 처럼 받는 매개변수의 개수를 모든 필드 수까지 점층적으로 늘려가는 형식의 패턴을 의미한다.

 <br>

 선택적 매개변수가 한두 개 있다면 크게 문제될 건 없겠지만, 10개, 20개가 된다면 그만큼 생성자가 필요하기 때문에 가독성과 유지보수성이 크게 저해된다.

 <br>

**자바 빈스 패턴**

```java
public class User {
    private String name;
    private String email;
    private int age;
    private String phNum;
    private String address;

    public User() {}

    public void setName(String name) { this.name = name; }

    public void setEmail(String email) { this.email = email; }

    public void setAge(int age) { this.age = age; }

    public void setPhNum(String phoneNumber) { this.phNum = phNum; }

    public void setAddress(String address) { this.address = address; }
}
```

<br>

 자바 빈스 패턴은 빈 객체를 생성하고 **setter**를 통해 필드를 초기화하는 형식의 패턴을 의미한다.

<br>

하지만, 모든 필드를 초기화하기 위해서는 **setter**를 여러 번 호출해야하며, 객체의 모든 필드가 초기화되기 전까지 **일관성이 무너진 상태로 유지**된다. 따라서, 자바 빈즈 패턴에서는 **불변 클래스를 만들 수 없**으며, 스레드 안정성을 얻기 위해서는 개발자가 추가적인 작업을 해주어야 한다. 

<br>

## 빌더 패턴

빌더 패턴은 **점층적 생성자 패턴**과 **자바 빈스 패턴**의 장점을 각각 취해 탄생한 패턴이다. 

<br>

```java
public class User {

    private final String name;
    private final String email;
    private final int age;
    private final String phoneNumber;

    public static class Builder {

        private final String name;
        private final String email;
        
        private int age = 0;
        private String phoneNumber = "";

        public Builder(String name, String email) {
            this.name = name;
            this.email = email;
        }

        public Builder age(int val) {
            age = val;
            return this;
        }

        public Builder phoneNumber(String val) {
            phoneNumber = val;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }

    private User(Builder builder) {
        name = builder.name;
        email = builder.email;
        age = builder.age;
        phoneNumber = builder.phoneNumber;
    }
}
```

<br>

빌더 패턴이 적용된 클래스를 살펴보자.

<br>

 클래스 내부에 정적 클래스 **Builder**가 정의되어 있고 필수 필드와 선택 필드를 각각 유지하고 있다. 필수적 필드에 대한 생성자와 각 선택자 필드를 설정하는 메소드가 있고, 외부 클래스의 **private 생성자를 호출**하는 **build** 메소드로 구성되어 있다. 

 <br>

그리고 아래와 같이 사용할 수 있다.

```java
User user = new User.Builder("김철수", "kim@example.com")
    .age(25)
    .phoneNumber("010-1234-5678")
    .build();
```

<br>

 매개변수의 순서에 구애받지 않으며 각 선택적 필드를 설정하는 메소드들은 자기 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다.

 <br>

이러한 방식을 **플루언트 API**, **메서드 연쇄**라고 한다. 

<br>

<aside>

**📌 불변(immutable)**

---

어떤 변경도 허용하지 않는다는 뜻으로, 주로 변경을 허용하는 가변과 구분하는 용도로 사용한다.

</aside>

<br>

<aside>

**📌 불변식(invariant)**

---

프로그램이 실행되는 동안, 또는 정해진 기간 동안 반드시 만족해야 하는 조건을 의미한다. 즉, 변경을 허용할 수는 있지만 특정 조건을 만족해야 한다는 뜻이다.  

</aside>

<br>

### 계층적으로 설계된 클래스와 빌더 패턴

아래는 추상 클래스는 추상 빌더를 구체 클래스는 구체 빌더를 갖게 설계한 코드이다.

<br>

**User → 추상 클래스**

```java
public abstract class User {
    final String name;
    final String email;
    final int age;
    
    abstract static class Builder<T extends Builder<T>> {
        String name;
        String email;
        int age = 0;
        
        public T name(String val) {
            name = Objects.requireNonNull(val);
            return self();
        }
        
        public T email(String val) {
            email = Objects.requireNonNull(val);
            return self();
        }
        
        public T age(int val) {
            age = val;
            return self();
        }
        
        abstract User build();
        
        protected abstract T self();
    }
    
    User(Builder<?> builder) {
        name = builder.name;
        email = builder.email;
        age = builder.age;
    }
}
```

<br>

**BasicUser → 구체 클래스**

```java
public class BasicUser extends User {
    private final String phoneNumber;
    
    public static class Builder extends User.Builder<Builder> {
        private String phoneNumber = "";
        
        public Builder phoneNumber(String val) {
            phoneNumber = val;
            return this;
        }
        
        @Override
        public BasicUser build() {
            return new BasicUser(this);
        }
        
        @Override
        protected Builder self() {
            return this;
        }
    }
    
    private BasicUser(Builder builder) {
        super(builder);
        phoneNumber = builder.phoneNumber;
    }
}
```

<br>

 추상 클래스에서는 추상 빌더를, 구체 클래스에서는 구체 빌더를 정의하고 있다.
 
<br>

 자바는 파이썬처럼 **self** 타입을 지원하지 않기 때문에 이러한 한계를 우회하기 위하여 **시뮬레이트한 셀프 타입 관용구**를 사용한다. 각 하위 클래스가 추상 메서드인 **self** 구현하여 자신의 정확한 타입을 반환하도록 강제한다. 
 
<br>

 **self** 메소드를 통해 **부모 클래스에서 정의된 메서드들이 하위 타입의 빌더를 반환**하게 되므로, 추상 클래스와 구체 클래스의 Builder 메서드들을 순서에 관계없이 연쇄적으로 호출할 수 있다. 

<br>

<aside>

**📌 공변 변환 타이핑**

---

하위 클래스의 메서드가 상위 클래스가 정의한 반환타입이 아닌, 그 하위 타입을 반환하는 것

→ User의 build 메소드는 반환 타입이 User 이지만, BasicUser의 build 메소드는 BasicUser 반환

</aside>
