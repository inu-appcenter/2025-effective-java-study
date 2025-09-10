# Item 4

추가 일시: 2025년 9월 5일 오후 5:26
강의: Effective JAVA

## 🍀 Item 4: 인스턴스화를 막으려거든 private 생성자를 사용하라

---

생성자를 명시하지 않으면 컴파일러는 자동으로 기본 생성자를 생성한다.

매개변수를 받지 않는 public 생성자가 만들어지며, 의도치 않게 인스턴스화 될 수 있다.

```java
public class UtilityClass {
	
	public static final String name = "이름";
	
	public static String getName(String name) {
		return name;
	}
}
```

다음과 같은 코드가 자동 생성된다.

```java
public UtilityClass() {}
```

❓ 그렇다면 추상 클래스로 만들면 될까?

→ 하위 클래스를 만들어 인스턴스화 하는것을 막을 수 없다. 사용자는 이를 상속해서 사용하라는 것으로 오해 할 수 있다.

컴파일러가 기본 생성자를 만드는 경우는  → 명시된 생성자가 없는 경우

즉, private 생성자를 추가하여 클래스의 인스턴스화를 막을 수  있다.

```java
public class UtilityClass {
	
	public static final String name = "이름";
	
	//private 생성자로 기본 생성자가 만들어지는 것을 방지
	private UtilityClass() {
		throw new AssertionError();
	}
	
	public static String getName(String name) {
		return name;
	}
}
```

private으로 만들어진 생성자이므로 클래스 외부에서 접근 할 수 없다.