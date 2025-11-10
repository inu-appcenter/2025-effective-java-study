<aside>

추상 클래스보다는 인터페이스를 우선하라

</aside>

자바는 다중 구현을 위해 인터페이스와 추상 클래스를 제공한다. 자바 8부터 인터페이스도 디폴트 메서드를 제공할 수 있게 되면서, 둘 다 구현 코드를 가질 수 있게 됐다. 이게 뭐가 다르냐면…

**추상 클래스의 제약**: 추상 클래스를 상속 받았다면 무조건 그 클래스의 하위 타입이 되어야 한다. 자바는 단일 상속만 지원하기 때문에, 이미 다른 클래스를 상속받고 있다면 새로운 추상 클래스를 끼워넣을 방법이 없다.

**인터페이스의 유연함**: 인터페이스가 요구하는 메서드만 구현하고 규약을 지키면, 어떤 클래스를 상속했든 상관없이 그 타입으로 취급된다.

```java
// 이미 Animal을 상속받은 상태
public class Dog extends Animal implements Comparable<Dog>, Runnable {
    // 문제없이 여러 인터페이스 구현 가능
}

// 추상 클래스는?
public class Dog extends Animal, AbstractComparable { // 컴파일 에러!
    // 불가능...
}
```

이제 인터페이스의 장점을 추상 클래스와 비교하여 알아보자.

### 기존 클래스에 새 기능 추가

메서드 몇 개 추가하고 implements만 붙이면 끝이다.

실제로 `Comparable`, `Iterable` 같은 인터페이스가 추가됐을 때, 자바 표준 라이브러리의 수많은 기존 클래스들이 이를 구현했다고 한다.

```java
// 기존 클래스에 인터페이스 추가
public class Cat {
    private String name;
    // ... 기존 코드
}

// 나중에 비교 기능이 필요해졌다면?
public class Cat implements Comparable<Cat> {
    private String name;
    // ... 기존 코드
    
    @Override
    public int compareTo(Cat other) {
        return this.name.compareTo(other.name);
    }
}
```

**추상 클래스의 경우**: 기존 클래스 위에 새로운 추상 클래스를 끼워넣으려면, 그 추상 클래스가 계층 구조상 공통 조상이어야 한다. 문제는 이게 적절하지 않은 상황에서도 강제로 모든 하위 클래스가 상속받게 된다는 것이다. 따라서 계층 구조에 혼란이 생긴다.

### 믹스인(Mixin) 정의

믹스인이란, 클래스의 주된 기능에 선택적 기능을 혼합(mixed in)하는 것이다.

```java
// 주된 타입: Animal
public class Dog extends Animal implements Comparable<Dog> {
    // Dog의 주된 기능
    @Override
    public void eat() { ... }
    
    // 믹스인: "나는 비교 가능한 Dog야"
    @Override
    public int compareTo(Dog other) {
        return this.age - other.age;
    }
}
```

`Comparable`은 ‘이 클래스의 인스턴스들끼리 순서를 정할 수 있다’는 선택적 기능을 제공한다. 추상 클래스로는 이런 믹스인을 정의할 수 없다. 클래스는 그 부모가 둘일 수 없고, 계층 구조에 믹스인을 삽입할 합리적인 위치가 없기 때문이다.

```java
// 방법 1: 최상위에 넣기?
class Animal extends Comparable { } // ❌ 
// → 모든 동물이 비교 가능해야 하나? 
// 비교가 필요 없는 동물도 강제로 구현해야 함

// 방법 2: 중간에 넣기?
class Mammal extends Animal, Comparable { } // ❌ 
// → 다중 상속 불가! 컴파일 에러

// 방법 3: 말단에 넣기?
class Dog extends Mammal, Comparable { } // ❌
// → 마찬가지로 다중 상속 불가!
```

### 계층구조가 없는 타입 프레임워크 생성

계층은 여러 개념을 구조적으로 표현하는 데에 효과적이다. 다만 현실에는 계층을 엄격하게 구분하기 어려운 개념들이 존재한다.

```java
public interface Singer {
    AudioClip sing(Song s);
}

public interface Songwriter {
    Song compose(int chartPosition);
}

// 싱어송라이터는?
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();
    void actSensitive();
}

// 구현도 자유롭게
public class Artist implements SingerSongwriter {
    // Singer, Songwriter의 모든 메서드 구현
}
```

가수이면서 작곡가인 싱어송라이터를 예로 들 수 있다. 이 경우 가수 클래스, 작곡가 클래스, 싱어송라이터 클래스로 n개의 속성에 따라 2^n개의 조합을 클래스로 만들게 된다. 이런 현상을 조합 폭발(combinatorial explosion)이 일어나고, 거대한 계층 구조에는 공통 기능을 정의한 타입도 없어서 중복 코드가 넘쳐나게 된다.

### 래퍼 클래스와의 조합

인터페이스는 래퍼 클래스 관용구와 함께 쓰면 기능을 안전하고 강력하게 향상시킬 수 있다. 타입을 추상 클래스로 정의하면? 상속밖에 방법이 없다. 상속으로 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기 쉽다.

```java
// 인터페이스 + 래퍼 클래스: 유연하고 안전
public class InstrumentedSet<E> implements Set<E> {
    private final Set<E> s;  // 위임
    private int addCount = 0;
    
    public InstrumentedSet(Set<E> s) {
        this.s = s;
    }
    
    @Override
    public boolean add(E e) {
        addCount++;
        return s.add(e);
    }
    // ... 나머지 메서드는 단순 위임
}
```

### 디폴트 메서드로 편의 제공

인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 디폴트 메서드로 제공하고 수고를 덜 수 있다.

```java
public interface Animal {
    void makeSound();  // 추상 메서드
    
    // 디폴트 메서드로 편의 제공
    default void introduce() {
        System.out.println("I am an animal that goes:");
        makeSound();
    }
}

// 구현 클래스는 makeSound만 구현하면 됨
public class Dog implements Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
    // introduce()는 자동으로 사용 가능
}
```

**디폴트 메서드의 제약**:

- `equals`, `hashCode`, `toString` 같은 Object 메서드는 디폴트 메서드로 제공 불가
- 인스턴스 필드를 가질 수 없음
- public이 아닌 정적 멤버를 가질 수 없음 (private 정적 메서드는 예외)
- 내가 만들지 않은 인터페이스에는 디폴트 메서드 추가 불가

### **골격 구현(Skeletal Implementation) 클래스**

인터페이스와 추상 클래스의 장점을 모두 가져가는 방법이다.

**역할 분담**:

- **인터페이스**: 타입 정의, 필요하면 디폴트 메서드 제공
- **골격 구현 클래스**: 나머지 메서드 구현

```java
// 1. 인터페이스 정의
public interface Animal {
    void eat();
    void sleep();
    void makeSound();
}

// 2. 골격 구현 클래스 (관례: AbstractXxx)
public abstract class AbstractAnimal implements Animal {
    // 기반 메서드는 추상으로
    @Override
    public abstract void makeSound();
    
    // 나머지는 구현 제공
    @Override
    public void eat() {
        System.out.println("Eating...");
    }
    
    @Override
    public void sleep() {
        System.out.println("Sleeping...");
    }
}

// 3. 구현 클래스는 골격 구현 확장만 하면 됨
public class Dog extends AbstractAnimal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
    // eat(), sleep()는 이미 구현됨
}
```

**골격 구현 작성 방법**:

1. 인터페이스를 살펴 다른 메서드 구현에 사용되는 **기반 메서드** 선정 → 골격 구현의 추상 메서드
2. 기반 메서드로 구현 가능한 메서드 → 디폴트 메서드로 제공 (단, Object 메서드 제외)
3. 남은 메서드 → 골격 구현 클래스에 구현

**골격 구현을 확장할 수 없는 경우 → 시뮬레이트한 다중 상속**

ex. 이미 다른 클래스를 상속받음

다중 상속의 장점(골격 구현 활용)을 얻으면서, 단점(복잡한 상속 구조)은 피한다.

```java
// 이미 다른 클래스를 상속받은 상황
// 기반 메서드만 구현 (외부 클래스로 위임)
public class MyList extends SomeOtherClass implements List<String> {
    
    // 골격 구현을 private 내부 클래스로 활용
    private class ListHelper extends AbstractList<String> {
        @Override
        public String get(int index) {
            // MyList의 구현 위임
            return MyList.this.getElement(index);
        }
        
        @Override
        public int size() {
            return MyList.this.getSize();
        }
    }
    
    private final ListHelper helper = new ListHelper();
    
    // List 메서드들을 내부 클래스에 위임
    @Override
    public String get(int index) {
        return helper.get(index);
    }
    
    @Override
    public int size() {
        return helper.size();
    }
    // ... 나머지 메서드들도 위임
}
```

### 정리

- 다중 구현용 타입은 인터페이스가 최적
- 복잡한 인터페이스라면 골격 구현을 함께 제공하자

<aside>

추상 클래스를 사용해야 하는 경우는?

</aside>