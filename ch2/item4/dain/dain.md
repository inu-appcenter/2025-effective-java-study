# Item4

<aside>

인스턴스화를 막으려거든 private 생성자를 사용하라

</aside>

### 인스턴스화를 왜 막아야 할까?

유틸리티 클래스에 대해 생각해보자. **`java.lang.Math`**, **`java.util.Arrays`** 처럼 정적 메서드만 포함하고 있으며 인스턴스는 필요 없다. 그런데 **생성자를 명시하지 않으면 컴파일러가 기본 public 생성자를 자동 생성**해버린다. 이렇게 되면 외부에서 인스턴스를 생성할 수 있게 되고, 의도에서 벗어나게 된다.

### 추상 클래스로 만든다면?

여전히 문제가 있다. 하위 클래스를 인스턴스화하면 되니까~

그래서 어떻게 해결하느냐… 답은 간단하다. **클래서 내부에 직접 private 생성자를 선언하면 된다.**

생성자 안에 **`throw new AssertionError()`**를 두면 내부에서 생성자를 호출하는 실수도 방지할 수 있다.

```java
public class AnimalUtils {
    // 인스턴스 생성을 막는 private 생성자
    private AnimalUtils() {
        throw new AssertionError("인스턴스 생성을 막습니다.");
    }

    public static void feed(String animal) {
        System.out.println(animal + "에게 먹이를 줍니다.");
    }

    public static void wash(String animal) {
        System.out.println(animal + "를 씻깁니다.");
    }
}
```

이제 **`new AnimalUtils()`**를 시도하면 AssertionError가 발생한다.

private이니 당연히 상속도 불가능하다.

<aside>

super 불가능하면(일반클래스)

어차피 상속 못 받는 거 아닌가 new못하는데

추상클래스도 super를 호출하나?

</aside>