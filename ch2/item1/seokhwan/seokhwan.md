# Item 1

추가 일시: 2025년 9월 4일 오후 1:44
강의: Effective JAVA

## 🍀 Item 1: 생성자 대신 정적 팩터리 메서드를 고려하라

---

```java
public static User from(String name, Long age) {
	return User.builder()
					.name(name)
					.age(age)
					.gender(Gender.MALE)
					.build();
}
```

책에서는 다섯가지 장점을 제시한다.

첫 번째, 이름을 가질 수 있다.

→ 정적 팩터리 메서드의 가장 큰 장점이라고 생각한다.

생성자 BigInteger(int, int, Random) 가 반환하는 객체가 어떤 특성을 가지는지 설명하기 어렵다.

이 때, BigInteger.probablePrime(int, int, Random)을 사용하면 ‘값이 소수인 BigInteger를 반환한다’는 의미를 확실하게 전달한다.

생성자는 하나의 시그니처가 하나의 생성자만 만들 수 있지만, 정적 팩터리 메서드를 사용한다면 시그니처가 같은 생성자를 각각의 메서드명을 통해 여러개 활용할 수 있다.

~~(입력 파라미터의 순서를 바꾸어 여러개의 생성자를 추가하는 것은 추후에 어떤 역할을 할지 모르게 만드는 멍청한 짓이라고 설명한다.)~~

두 번째, 호출 될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

→ 생성자 대신 정적 팩터리 메서드를 사용할 때의 이점으로 적절한가? 라는 생각이 든다.

불필요한 객체 생성을 하지 않아(Boolean.valueOf(boolean)) 생성비용이 큰 객체가 요청되는 상황에 성능을 올려준다는 사실에 대해서는 반박하지 않는다. 

다만, 정적메서드를 사용함으로써 추가적으로 얻을 수 있는 이득이지 ‘생성자 대신 정적 팩터리 메서드를 사용하는 근본적인 이유’는 아닌 것 같다는 생각이다.

- 세 번째, 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
    
    ```java
    // 생성자: 항상 선언된 타입의 인스턴스만 반환
    public class Animal {
        public Animal() { } // Animal 인스턴스만 반환 가능
    }
    
    // 정적 팩터리: 하위 타입 반환 가능
    public class Animal {
        public static Animal createDog() {
            return new Dog(); // Dog는 Animal의 하위 타입
        }
        public static Animal createCat() {
            return new Cat(); // Cat은 Animal의 하위 타입
        }
    }
    ```
    
- 네 번째, 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    
    ```java
    // 생성자: 불가능
    public class EnumSet<E extends Enum<E>> {
        public EnumSet(Collection<E> c) {
            // 항상 EnumSet 인스턴스만 생성 가능
        }
    }
    
    // 정적 팩터리: 가능
    public static <E extends Enum<E>> EnumSet<E> of() {
        if (elements.length <= 64) {
            return new RegularEnumSet<>(elements); // 작은 경우
        } else {
            return new JumboEnumSet<>(elements);   // 큰 경우
        }
    }
    ```
    
- 다섯 번째, 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    
    ```java
    // 생성자: 컴파일 시점에 클래스가 반드시 존재해야 함
    public class Connection {
        public Connection() { } // Connection 클래스가 반드시 존재해야 함
    }
    
    // 정적 팩터리: 런타임에 클래스를 찾아서 반환 가능 (서비스 제공자 프레임워크)
    public class DriverManager {
        public static Connection getConnection(String url) {
            // 런타임에 적절한 구현체를 찾아서 Connection 반환
        }
    }
    ```
    

등의 추가적인 이점을 추가적으로 제시하지만, 이름을 통해 반환 객체의 특성을 명확히 하는 것이 가장 중요한 활용 요소라 생각된다.

단점으로 두가지를 제시하는데

첫 번째, 상속을 하려면 public이나 protected생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다. 는 개인적으로 오히려 장점으로 다가온다

두 번째,  정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

→ 과거에는 어땠을지 모르겠으나 요즘에는 IDE 코드 자동완성기능이 너무 잘 되어있어서 큰 불편함은 아니라고 생각된다.