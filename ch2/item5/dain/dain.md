# Item5

<aside>

자원을 직접 명시하지 말고 의존 객체 주입를 사용하라

</aside>

아래처럼 정적 유틸리티 클래스를 사용한 코드가 있다.

```java
public class AnimalSoundChecker {
    private static final Dictionary dictionary = ...;  // 사전(자원)을 직접 내부에 고정

    private AnimalSoundChecker() {}  // 인스턴스 생성 방지

    public static boolean isValidSound(String sound) {
        // dictionary에 의존해 유효성 검사
    }
}

```

- 클래스 내부에 고정된 자원을 사용한다.
- 테스트용 사전을 쓰고 싶어도 변경 불가하다.

자원이 하나라고 가정하여 설계한 탓에 확장성과 유연성이 떨어진다.

그럼 싱글턴 패턴에서는 어떨까?

```java
public class AnimalSoundChecker {
    private final Dictionary dictionary;

    private AnimalSoundChecker(Dictionary dict) {
        this.dictionary = dict;
    }

    public static final AnimalSoundChecker INSTANCE = new AnimalSoundChecker(...);

    public boolean isValidSound(String sound) {
        // dictionary를 참조해 로직 수행
    }
}
```

- 유일한 인스턴스가 고정된 자원에 의존해 동작한다.
- 만약 다른 사전을 써야 한다면 인스턴스를 새로 만들 수 없고, 수정 불가능하다.
- 테스트 목적으로 가짜 사전을 넣기도 어렵다.

두 방식 모두 하나의 사전을 사용한다고 가정하고 있다. 책에서는 자원에 따라 동작이 달라지는 이러한 클래스에 ‘**의존 객체 주입(Dependency Injection)’**을 권장한다. **필요한 자원을 외부에서 주입받도록 설계**하자는 말이다.

```java
public class AnimalSoundChecker {
    private final Dictionary dictionary;

    // 생성자에서 사전을 주입받는다.
    public AnimalSoundChecker(Dictionary dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValidSound(String sound) {
        // 사전에 의존하여 유효성 검사
    }
}
```

- 필요한 사전을 클라이언트가 직접 넘길 수 있다. → 테스트용 사전을 포함하여 다양하게 주입 가능
- 여러 인스턴스를 생성해 사용하더라도 각각이 필요로 하는 자원을 사용할 수 있다.