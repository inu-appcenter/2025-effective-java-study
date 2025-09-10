# Item 2

추가 일시: 2025년 9월 5일 오후 5:24
강의: Effective JAVA

## 🍀 Item 2: 생성자에 매개변수가 많다면 빌더를 고려하라

---

정적 팩터리와 생성자는 선택적 매개변수가 많을 때 적절한 대응이 어렵다.

어떤 파라미터 값이 들어오는지에 따라 경우마다 정적 팩터리 메서드나 생성자를 새로 만들어야 한다.

### 점층적 생성자 패턴

프로그래머들은 점층적 생성자 패턴(telescoping constructor pattern)을 주로 사용했다.

```jsx
public class User {
	private final String name; //필수
	private final Long age; //필수
	private final String studentNumber; //선택
	private final String major; //선택

	public User(String name, Long age){
		this.name = name;
		this.age = age;
	}
	
	public User(String name, Long age, String studentNumber, String major){
		this.name = name;
		this.age = age;
		this.studentNumber = studentNumber;
		this.major = major;
	}
}
```

이런 방식의 점층적 생성자 패턴은 매개변수의 개수가 증가하면 클라이언트 코드를 작성하고 읽는데 어려움이 있다. 즉, 확장성에 문제가 있다.

### 자바빈스 패턴

또다른 방식으로 자바빈즈 패턴이 있다.

```jsx
public class User {
	private String name = "aaa"; //필수
	private Long age = -1; //필수
	private String studentNumber = ""; //선택
	private String major = ""; //선택

	public User() {}
	
	public void setName(String val) { name = val; }
	public void setAge(Long val) { age = val; }
	public void setStudentNumber(String val) { studentNumber = val; }
	public void setMajor(String val) { major = val; }
}
```

자바빈즈 패턴은 점층적 생성자 패턴과 비교하면 가독성이 뛰어난 것을 알 수 있다.

다만, 객체 하나를 만들기 위해 여러 개의 메서드를 호출해야 하고, 객체가 완전히 완성되기 전까지는 일관성(consistency)이 무너진 상태에 놓인다. → 디버깅도 어렵다.

일관성이 무너지는 문제 때문에 클래스를 불변으로 만들 수도 없다.

### 빌더 패턴

```jsx
public class User {
	private final String name;
	private final Long age;
	private final String studentNumber;
	private final String major;
	
	public static class Builder {
		//필수 매개변수
		private final String name;
		private final Long age;
		
		//선택 매개변수
		private final String studentNumber = "";
		private final String major = "";
		
		public Builder(String name, Long age) {
			this.name = name;
			this.age = age;
		}
		
		public Builder studentNumber(String val) {
			studentNumber = val;
			return this;
		}
		
		public Builder major(String val) {
			major = val;
			return this;
		}
		
		public Builder build() {
			return new User(this);
		}
	}
	
	private User(Builder builder) {
			name = builder.name;
			age = builder.age;
			studentNumber = builder.studentNumber;
			major = builder.major;
		}
}	
```

빌더 패턴을 보면 점층적 생성자 패턴과 자바빈즈 패턴의 장점만 취한것을 알 수 있다.

필요한 객체를 직접 만들지 않고, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다.

이후 빌더 객체가 제공하는 메서드로 원하는 선택 매개변수들을 설정한다.

보통 빌더는 생성할 클래스 안에 정적 클래스로 만들어두는 것이 일반적이다.

빌더의 세터 매서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출 할 수 있다.

이를 플루언트(fluent API) 혹은 메서드 연쇄(method chaining)라고 한다.

**계층적으로 설계된 클래스에서의 빌더 활용**

빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.

```jsx
public abstract class User {
	private final String name;
	private final Long age;
	
	abstract static class Builder<T extends Builder<T>> {
		String name;
		Long age;
		
		public T name(String name) {
			this.name = Objects.requireNonNull(name);
			return self();
		}
		
		public T age(Long age) {
			this.age = Objects.requireNonNull(age);
			return self();
		}
		
		abstract User build();
		
		// 하위 클래스는 이 메서드를 오버라이딩하여 this를 반환하도록 해야한다.
		protected abstract T self();
	}
	
	User(Builder<?> builder) {
		name = builder.name;
		age = builder.age;
	}
}
```

자바는 self타입이 없기 때문에 우회 방법을 시뮬레이트한 셀프타입 관용구를 사용한다.

추상 메서드 self를 통해 하위 클래스에서는 형변환 없이 메서드 연쇄를 지원한다.

```jsx
public class Student extends User {
	private final String studentNumber;
	private final String major;
	
	public static class Builder extends User.Builder<Builder> {
		private String studentNumber = "";
		private String major = "";
		
		public Builder studentNumber(String studentNumber) {
			this.studentNumber = studentNumber;
			return this;
		}
		
		public Builder major(String major) {
			this.major = major;
			return this;
		}
		
		@Override
		public Student build() {
			return new Student(this);
		}
		
		@Override
		public Builder self() {
			return this;
		}
	}
	
	private Student(Builder builder) {
		super(builder);
		studentNumber = builder.studentNumber;
		major = builder.major;
	}
}
```

빌더 패턴은 객체를 생성하기 위해 빌더부터 만들어야한다는 단점이 있다. 따라서 빌더 생성 비용을 소비한다.

점층적 생성자 패턴에 비해 코드가 길어~~(`@Builder` 어노테이션 쓰면 별차이 없지 않나?)~~ 매개변수가 4개 이상은 되어야 값어치를 한다.

다만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있어 애초에 빌더 패턴으로 시작하는 편이 좋다.