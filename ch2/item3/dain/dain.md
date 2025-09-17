# Item3

<aside>

private 생성자나 열거 타입으로 싱글턴임을 보증하라

</aside>

싱글턴 패턴은 클래스의 인스턴스를 하나만 생성하여 어디서든 동일 객체를 공유하도록 한다.

<aside>

왜 싱글턴을 써야 할까?

하나만 놓고 구현을 하자

</aside>

다만 이 경우 테스트에 어려움이 생긴다. 내부 인스턴스를 외부에서 임의로 교체하거나 변경할 수 없으니, 테스트 환경에서 원하는 동작을 하도록 가짜 객체를 끼워 넣기 어렵다.

아래는 싱글턴의 구현 방식이다.

```java
public class Animal {
    public static final Animal INSTANCE = new Animal(); // 컴파일 시점 

    private Animal() {
        // 생성자를 private으로 감춤
    }

    public void speak() {
        System.out.println("동물동물");
    }
}

```

1. public static final 필드로 싱글턴 제공
    - Animal.INSTANCE는 클래스가 처음 로딩될 때 한 번 생성된다.
    - 인스턴스가 공용으로 사용되고, 새 인스턴스를 만들 수 없다.

      → 직관적이고 간단함. 싱글턴이라는 점이 명확히 드러남.


```java
public class Animal {
    private static final Animal INSTANCE = new Animal(); 

    private Animal() {
    }

    public static Animal getInstance() {
        return INSTANCE;
    }

    public void speak() {
        System.out.println("동물동물");
    }
}

```

1. 정적 팩터리 메서드
    - API 변경 없이도 더이상 싱글턴이 아니도록 만들 수 있다.
    - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
        - 싱글톤 + 제네릭 → 다양한 타입의 객체를 단일 인스턴스로 관리 및 반환 가능
    - 메서드 참조를 공급자(supplier)로 사용할 수 있다.
        - 메서드 참조를 통해 값이나 객체를 생성하여 제공하는 역할을 하는 Supplier 인터페이스를 구현할 수 있다


    이 방식은 조금 더 유연하지만, 싱글턴 클래스를 직렬화 후 다시 역직렬화한다면 그 과정에서 새로운 인스턴스가 생성되면서 유일성이 깨질 위험이 있다. 항상 기존 인스턴스를 반환하게 하려면 readResolve() 메서드를 추가해야 한다.
    
    ```java
    private Object readResolve() {
        return INSTANCE;
    }
    ```
    
    이렇게 하면 진짜를 반환하고, 가짜는 가비지 컬렉터로 넘긴다. 
    
    <aside>
    
    ```java
    직렬화
    
    // 바이트로 변환해서 저장
    oos.writeObject(elvis1);
    
    역직렬화
    // 바이트를 다시 객체로 복원, 이 때 새로운 Elvis가 만들어짐
    Elvis elvis2 = (Elvis) ois.readObject();
    
    따라서 elvis1 != elvis2
    
    하지만 readResolve를 구현하면 
     private Object readResolve() {
        return INSTANCE;  // 가짜가 아니라 진짜를 반환함
    }
    
    readResolve는 자동으로 역직렬화할 때 JVM이 실행해줌
    ```
    
    </aside>


이 책에서 가장 권장하는 것은 **열거(enum) 타입**을 활용하는 아래의 방식이다.

```java
public enum Animal {
    INSTANCE;

    public void speak() {
        System.out.println("동물동물");
    }
}
```

열거 타입은 위에서 언급한 직렬화 등의 문제를 방지하면서도 간결하다.

(단, 다른 클래스를 상속해야 할 경우 사용이 불가하다.)