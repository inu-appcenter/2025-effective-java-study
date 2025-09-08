# Item05

> ***자원을 직접 명시하지 말고 의존 객체 주입을 사용하라***

<br>

## 의존 객체 주입

 **의존 객체 주입**은 인스턴스를 생성할 때, 생성자에 필요한 자원을 매개변수로 넘겨주는 방식이다. 클래스가 내부적으로 자원을 할당하는 대신 외부에서 필요한 의존성을 받아 사용한다. 

 <br>

```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) {
        return dictionary.contains(word);
    }
}
```

<br>

SpellChecker 클래스를 보면 dictionary 필드를 생성자를 통해 초기화하고 있다. 그리고 아래처럼 Lexicon을 인터페이스로 정의하게 되면, 상황에 따라 유연하게 적절한 구현체를 사용하는 방식으로 활용할 수 있다.

<br>

```java
public interface Lexicon {
    boolean contains(String word);
}

public class EnglishDictionary implements Lexicon {
    @Override
    public boolean contains(String word) {
        return true;
    }
}

public class KoreanDictionary implements Lexicon {
    @Override
    public boolean contains(String word) {
        return true;
    }
}
```

<br>

이 방식은 테스트가 가능하며(Mocking) OCP 원칙까지 잘 지킬 수 있는 좋은 방법이다. 

<br>

## 자원 팩터리를 넘겨주면

의존 객체 주입의 변형으로 **생성자에 자원 팩터리를 넘겨주는 방식**이 있다. 여기서 자원 팩토리를 넘겨준다는 말의 의미는 단순히 객체를 넘겨주는 것이 아니라 **객체를 생성하는 팩터리를 주입**한다는 의미이다. 

<br>

**Supplier<T>** 인터페이스는 객체 생성 팩토리의 대표적인 예시인데, 아래 코드를 보며 살펴보자.

<br>

```java
public class SpellChecker {
    private final Supplier<Lexicon> dictionaryFactory;
    
    public SpellChecker(Supplier<T extends Lexicon> dictionaryFactory) {
        this.dictionaryFactory = Objects.requireNonNull(dictionaryFactory);
    }
    
    public boolean isValid(String word) {
        Lexicon dictionary = dictionaryFactory.get(); // 매번 새로운 인스턴스 생성
        return dictionary.contains(word);
    }
}
```

<br>

위와 같이 생성자에 **Supplier<T extends Lexicon>**을 매개변수로 받고 아래처럼 활용할 수 있다. 단, 팩터리를 주입받는 메서드는 한정적 와일드카드 타입으로 매개변수를 제한해야 한다.

<br>

<aside>

**📌 한정적 와일드카드 타입**

---

확정되지 않은 타입의 범위를 특정 타입의 하위 또는 상위로 제한하여 API의 유연성을 높이는 기법.

</aside>

<br>

```java
// 람다 표현식으로 팩터리 전달
SpellChecker checker = new SpellChecker(() -> new EnglishDictionary());

// 메서드 참조로 팩터리 전달
SpellChecker checker = new SpellChecker(EnglishDictionary::new);

// 단, 여기서 EnglishDictionary 클래스는 Lexicon을 상속한다.
```
