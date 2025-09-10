# 아이템 4

## 인스턴스화를 막으려거든 private 생성자를 사용하라

### 유틸리티 클래스란?

정적 메서드와 정적 필드만을 담은 클래스로, 인스턴스를 만들 필요가 없는 클래스

예시

```java
Math.max(10, 20);
StringUtils.isEmpty(str);
```

**유틸리티 클래스를 인스턴스화하면 생기는 문제점**

불필요한 인스턴스 생성 허용

→ 이유 : 기본 생성자를 클래스에 작성하지 않아도 클래스에 생성자가 하나도 없으면 컴파일러가 자동으로 기본 생성자를 생성하기 때문

**추상클래스도 하면 안되는 이유**

추상클래스를 상속하면 자식 클래스가 부모인 추상클래스의 private를 제외한 나머지 필드를 접근할 수 있기 때문

**올바른 해결 방법 = private 생성자**

```java
public class UtilityClass {
    private UtilityClass() {
        throw new AssertionError(); // 클래스 내에서 생성했을 경우 오류발생
    }
   
}
```

**외부 차단** : private 생성자

**내부 차단** : throw new AssertionError

**상속 차단** : 자식 클래스가 부모 클래스의 필드를 사용하기 위해선 자식의 생성자에서 부모의 생성자를 호출해야 하는데 private이므로 불가능

```java
public class UtilityClassChild {

	public UtilityClassChild() {
		super() -> 생성자의 가장 첫번째 줄에는 super() 즉 부모의 생성자를 호출하는데
		부모의 생성자가 private이므로 super()가 불가능
	}

}
```