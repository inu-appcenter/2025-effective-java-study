# Item 1

<aside>

## 생성자 대신 정적 팩터리 메서드를 고려하라

</aside>

정적 팩터리 메서드(static factory method)

- 이름 그대로, static method로 인스턴스 반환하는 방법

## 정적 팩터리 메서드가 생성자보다 우수한 5가지 요소

### 1. 이름을 가질 수 있다

- ‘메서드’이기 때문에, 추가적인 설명 없이도 적절한 네이밍만으로 그 의미를 묘사할 수 있다.

    ```
    public class Book {
        private String name;
        private String author;
        private String publisher;
    
        // 생성자
        public Book(String author) {
            this.author = author;
        }
    
        // 정적 펙토리 메서드
        public static createBookWithAuthor(author) {
            return new Book(author);
        }
    }
    ```

    ```
    Book book1 = new Book("Appcenter"); 
    // Appcenter가 어떤 변수인지 알기 어렵다. author일까 publisher일까
            
    Book book2 = Book.createBookWithAuthor("Appcenter"); 
    // Appcenter가 author에 해당한다는 것을 한눈에 알 수 있다.
    ```


- 이름만 잘 짓는다면 하나의 클래스 내에서 동일한 시그니처를 가져도 불편함 없이 사용 가능하다. 이름만으로 그 차이가 드러나기 때문에.

### 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다

- 객체를 매번 새로 만들 필요가 없으므로 캐싱이나 미리 만들어둔 객체를 재사용한다.
    - 클래스 내부에 static final로 미리 선언한 객체를 반환
- 동일한 요청마다 같은 객체를 반환하니 메모리 및 성능에서 이점이 있다.
- 자주 사용되는 값에 대해 "싱글턴"처럼 동작하므로, a == b 비교가 논리적 동등성(equals)이 아닌 참조 동등성(동일 인스턴스)만으로도 동작한다.안에 있는 걸 컨트

    <aside>

  util 클래스

  근데 우리는 스프링이 해주니까

    </aside>


### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

- 팩터리 메서드의 반환 타입은 고정되지만, 실제 반환하는 객체의 구체 클래스에는 제한이 없다

    ```java
    interface Animal {}
    class Dog implements Animal {}
    class Cat implements Animal {}
    
    public static Animal newAnimal(String type) {
        if (type.equals("dog")) return new Dog();
        else return new Cat();
    }
    
    Animal a = newAnimal("dog"); // Dog 객체
    Animal b = newAnimal("cat"); // Cat 객체
    ```

- 위 코드에서, 메서드의 반환 타입은 Animal이다. Dog나 Cat을 몰라도, Anlmal 타입의 객체 받는다는 것만 알면 된다.
    - 메서드 내부에서는 type에 따라 Dog나 Cat 객체를 반환하고, 둘 다 Animal을 구현했으니 Animal 타입으로 업캐스팅 되어 반환된다.
    - 메서드를 호출하는 사용자는 오직 Animal의 인터페이스만을 사용한다. 추후 newAnimal(”cow”)가 Cow 객체를 반환하도록 수정되어도, 클라이언트 코드는 수정할 필요가 없다.
        - → Animal이라는 공통 타입만 알면 되니 유연하고 변경에 강한 코드


### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다

- 입력에 따라 여러 하위 타입 객체를 조건에 맞게 골라 반환할 수 있다.
- 교재 예시: 원소의 수에 따라 두 가지 하위 클래스 중 하나의 인스턴스를 반환하는 EnumSet
    - 사용자는 EnumSet이라는 상위 타입만 알면 되고, 내부적으로 다른 객체를 반환한다. (성능 개선, 확장성 및 안정성 보장)

    <aside>

  디자인 패턴을 생각하고 하면 쓰게 될 것 같음

  중요한데 우리가 아직 안 쓰는

    </aside>


### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

- 구현 클래스가 나중에 등록되어도

    <aside>

  추상화+다형성을 버무려서 생각하면 맞말

  근데 왜 정적팩터리메서드?

  만약, 일반 생성자나 일반 인스턴스 메서드를 사용했다면 실제 객체를 생성해야하며, 클라이언트는 구체적 구현체를 알고 있어야 하는 단점이 있다.

  생성자 → 컴파일 시점에 있어야 함
  정.팩.메 → 쓸 때만 있으면 됨 (ex. db)

    </aside>


## 단점

### 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터 리 메서드만 제공하면 하위 클래스를 만들 수 없다.

- 상황에 따라 장점으로도, 단점으로도 작용한다. 보안 측면에서 우수한 대신 상속을 통한 확장은 제한된다.

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

- 어떤 팩터리 메서드가 있는지 직관적으로 알기 어렵다.
    - 생성자는 new 클래스명(…)만 기억하면 쉽게 찾는다.
- API 문서나 네이밍 규칙을 참고하지 않고서는 해당 클래스가 어떤 메서드로 객체를 생성하는지 알 수 없다.
    - 문서화가 잘 되어 있지 않거나, 네이밍 컨벤션이 일관되지 않으면 더욱 어렵다.