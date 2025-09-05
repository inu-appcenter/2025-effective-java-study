# Item04

> 인스턴스화를 막으려거든 private 생성자를 사용하라

<br>

## 인스턴스화를 원하지 않는다면

### private 생성자 사용하자.

 명시적으로 **private** 생성자를 사용하지 않는다면, 기본 생성자가 자동으로 생성되어 인스턴스화가 가능하다. 바로 아래와 같은 코드이다.

 <br>

```java
public class MathUtils{
	public static int add(int a, int b){
		return a+b;
	}
}
```

<br>

따라서, 아래와 같이 생성자의 접근 제어자를 명시적으로 **private**로 제한하자. 이렇게 하면 상속이 불가능하기 때문에 인스턴스화를 완벽하게 제어할 수 있다.

<br>

```java
public class MathUtils {

    private MathUtils() {
        throw new AssertionError();
    }
    
    public static int add(int a, int b) {
        return a + b;
    }
}
```

<br>

<aside>

📌 컴파일러는 ***명시된 생성자가 없을 때만 기본 생성자가 자동으로 생성***된다.

</aside>

<br>

### 추상 클래스는 완벽한 해결책이 아니다.

 추상 클래스는 인스턴스화가 불가능하지만, 이를 상속하는 클래스가 인스턴스화가 가능하다면 이것도 완벽한 해결책은 아니다. 

 <br>

```java
// 추상 클래스
public abstract class AbstractUtils {
    public static void doSomething() {}
}

// 추상 클래스를 상속하는 클래스
public class ConcreteUtils extends AbstractUtils {

}

ConcreteUtils instance = new ConcreteUtils(); // 인스턴스화가 가능하다
```
