# item5

## 🔒 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

---

많은 클래스가 하나 이상의 자원에 의존합니다.

이러한 자원을 정적 유틸리티, 싱글턴으로 명시하는 것은 좋지 않습니다.

여러가지의 대응할 방법이 필요합니다.

- 어떠한 방법이 있을까요
    - 필드에서 final을 제거하고 다른 사전으로 교체하는 메서드를 추가하는 방법
        - 오류를 내기 쉬우며, 멀티 스레드 환경에서는 쓸 수 없습니다.
        - 사용하는 자원에 따라 동작이 달라지는 환경에서는 정적 유틸리티, 싱글턴 방식이 적합하지 않습니다.

    - 클래스가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원을 사용해야합니다.

  → 바로 **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**입니다.

    - 이는 의존 객체 주입의 한 형태입니다.

    ```java
    public class Spellchecker {
    		private final Lexicon dictionary;
    		// 의존 객체 주입
    		public Spellchecker(Lexicon dictionary) {
    				this.dictionary = Objects.requireNonNull(dictionary);
    		}
    		public boolean isValid(String word) { ... }
    		public List<String> suggestions(String typo) { ... }
    }
    ```

    - 딱 하나의 dictionary 자원만 사용하지만 어떤 환경에서든 잘 작동합니다.
    - 또한 불변을 보장해 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있습니다.
    - 의존 객체 주입은 생성자, 정적 팩터리, 빌더에 모두 똑같이 적용할 수 있습니다.

- 이러한 패턴의 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있습니다.
    - 이때, **팩터리**란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체
    - 즉, 팩터리 메서드 패턴을 구현한 것
    - Supplier<T> 인터페이스가 팩터리를 표현한 완벽한 예

- **Supplier란?**
    - 자바 8에서 도입된 **함수형 인터페이스** 중 하나
    - 매개변수 없이 **T 타입의 객체를 반환**하는 역할

    ```java
    @FunctionalInterface
    public interface Supplier<T> {
        T get();
    }
    
    Supplier<List<String>> factory = ArrayList::new;
    List<String> list = factory.get(); // ArrayList 인스턴스 생성
    ```

- **bounded wildcard(한정적 와일드카드)와 타입 제한**
    - 팩터리 메서드에서 **T 타입을 제한**하면 더 안전하게 사용할 수 있습니다.
        - ex) **T는 특정 클래스나 그 하위 클래스만 가능**하게 제한할 수 있음

        ```java
        public <T extends Number> void process(Supplier<T> supplier) { ... }
        ```

        - 클라이언트는 **Number 또는 하위 타입(Integer, Double 등)의 객체를 생성하는 팩터리**만 넘길 수 있습니다.

- 의존 객체 주입이 유연성과 테스트 용이성을 개선
    - BUT, 코드를 어지럽게 만들기도 합니다.
    - 대거(Dagger), 주스(Guice), 스프링(Spring) 같은 의존 객체 주입 프레임워크를 사용해 해소