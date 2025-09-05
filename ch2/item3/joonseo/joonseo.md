# Item03

> private 생성자나 열거 타입으로 싱글턴임을 보장하라

<br>

## 싱글톤이란

**싱글톤**은 인스턴스로 단 하나만 생성될 수 있는 클래스를 의미한다. 하지만, 인터페이스 없이 단일 싱글톤 클래스로 유지하면 테스트하기가 어려워진다.

<br>

 테스트를 진행할 경우 인터페이스가 없으면, 컴파일 시점에는 구체 클래스가 테스트에 강하게 결합되어 다른 구현체로 교체할 수 없기 때문이다. 

<br>

```java
public interface UserService {
    void saveUser(User user);
}

public class UserServiceImpl implements UserService{

	private static final UserServiceImpl userServiceImpl = new UserServiceImpl();
	
	private UserServiceImpl(){
	
	}
	
	@Override
	public void saveUser(User user){
		// 구현
	}

}

```

<br>

따라서, 위와 같이 인터페이스와 구현체를 분리해야만 Mocking이 가능하다.

<br>

## 싱글톤을 만드는 방법

### public static final 필드 사용하기

 이 방법에는 두가지 조건이 따르는데, **첫번째**로 생성자의 접근 제어자를 **private**로 제한하여 필드 초기화의 경우를 제외하고 생성자의 호출을 제한한다. **두번째**로 인스턴스 필드를 **public**으로 열어 다른 패키지나 클래스에서 접근할 수 있게 한다.

 <br>

```java
public class UserServiceImpl {

	public static final UserServiceImpl instance = new UserServiceImpl();
	
	private UserServiceImpl(){
	
	}
	
	public void saveUser(User user){
	}

}
```

<br>

 단, 접근에 대한 권한이 특별하게 주어진 API에 대해서는 접근 자체를 막을 방법은 없다. 하지만, 내부적으로 생성자가 두 번 호출될 때 예외를 반환하게 설계하여 이를 방지할 수 있다.

 <br>

**public static final** 필드를 사용하면 아래와 같은 장점을 가져갈 수 있다.

- 클래스가 싱글턴임을 API 내에서 명확하게 확인할 수 있다.
- 간결하다.

<br>

### 정적 팩터리 메서드 활용하기

이 방법에서는 인스턴스 필드까지 private으로 제한하지만, 이 필드에 접근할 수 있는 정적 팩터리 메서드를 제공하여 인스턴스에 접근할 수 있게 한다.

<br>

```java
public class UserServiceImpl {

	private static final UserServiceImpl instance = new UserServiceImpl();
	
	private UserServiceImpl(){
	
	}
	
	public static UserServiceImpl getInstance(){
	}

}
```

<br>

getInstatance는 **필드 초기화시 할당된 instance를 고정적으로 반환**하므로 이 또한 싱글턴을 유지할 수 있는 방법 중 하나이다. 

<br>

정적 팩터리 메서드를 활용하면 아래와 같은 장점이 있다.

- 요구사항이 변경되어 싱글턴이 아니게 되어도 내부 구현만 변경하여 유연하게 확장할 수 있다.
- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
- 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.

<br>

<aside>

🚨 위 두 방식으로 구성된 싱글턴들은 **모든 인스턴스 필드를 일시적이라고 선언**하고 **readResolve** 메소드를 정의해야 역직렬화 과정에서 새로운 인스턴스가 생기는 것을 방지할 수 있다!

</aside>

<br>

### 열거 타입 사용하기

 열거 타입으로 싱글턴을 만들면, 더 간결하고 추가되는 코드 없이 싱글턴을 보장할 수 있다. 또한, 복잡한 직렬화 과정이나 리플렉션 공격에서도 완벽하게 막아준다.

 <br>

```java
public enum UserServiceImpl {
	INSTANCE;
}
```

<br>

단, 만들고자 하는 싱글턴이 **Enum** 외에 다른 클래스를 상속해야 한다면, 해당 방식은 사용할 수 없다.
→ 이미 **enum** 타입은 **java.lang.Enum**을 상속하고 있기 때문이다.
