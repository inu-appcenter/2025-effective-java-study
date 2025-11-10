<aside>

표준 예외를 사용하라

</aside>

## 왜 그래야 하나

자바 라이브러리가 제공하는 표준 예외는 다른 개발자들에게 이미 익숙한 규약이기 때문이다. 직관적이고, 코드 리뷰나 유지보수에도 편하다.

## 자주 재사용되는 예외들

### **IllegalArgumentException**

- 메서드에 전달된 인수 자체가 문제일 때

```java
public void setAge(int age) {
    if (age < 0) {
        // 인수 값 자체가 잘못됨
        throw new IllegalArgumentException("나이는 음수일 수 없습니다: " + age);
    }
    this.age = age;
}

public void processItems(List<Item> items) {
    if (items.isEmpty()) {
        // 빈 리스트를 전달한 것 자체가 문제
        throw new IllegalArgumentException("처리할 항목이 없습니다");
    }
}
```

### **IllegalStateException**

- 객체의 현재 상태 때문에 메서드를 실행할 수 없을 때

```java
public class ShoppingCart {
    private boolean checkedOut = false;
    private List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        if (checkedOut) {
            // 이미 결제된 장바구니에는 아이템을 추가할 수 없음
            // 인수(item)는 문제없지만, 객체 상태가 문제
            throw new IllegalStateException("이미 결제가 완료된 장바구니입니다");
        }
        items.add(item);
    }
    
    public void checkout() {
        if (items.isEmpty()) {
            // 장바구니가 비어있는 상태가 문제
            throw new IllegalStateException("장바구니가 비어있습니다");
        }
        checkedOut = true;
    }
}
```

### **IllegalArgumentException vs IllegalStateException**

둘 중 무엇을 던져야 할지 애매한 경우도 있다.

일반적으로, 인수 값이 무엇이었든 어차피 실패했을 거라면 `IllegalStateException`, 그렇지 않으면 `IllegalArgumentException`을 던진다.

```java
public class CardDeck {
    private List<Card> cards;
    
    public List<Card> dealCards(int count) {
        // 덱에 10장이 남았는데 20장을 요청한 경우?
        // IllegalArgumentException? IllegalStateException?
        
        if (count > cards.size()) {
            // 인수 값이 무엇이든 실패했을 것 -> IllegalStateException
            // 인수 값이 적절했다면 성공했을 것 -> IllegalArgumentException
        }
    }
}
```

위 예시에서는 덱의 상태(남은 카드 수)가 문제이므로 `IllegalStateException`이 더 적합하다. 만약 요청한 카드 수가 음수라면 `IllegalArgumentException`을 던져야 한다.

### **NullPointerException**

- null 체크
- null 체크는 `IllegalArgumentException`이 아닌 `NullPointerException`을 던지는 게 관례다.

```java
public void registerAnimal(Animal animal) {
    // ❌ 이거 말고
    if (animal == null) {
        throw new IllegalArgumentException("동물 정보는 필수입니다");
    }
    
    // ✅ 이게 관례
    if (animal == null) {
        throw new NullPointerException("동물 정보는 필수입니다");
    }
    
}
```

### **UnsupportedOperationException**

- 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때
- 주로 인터페이스 메서드 일부를 구현할 수 없을 때 사용한다.

```java
public class ReadOnlyAnimalList implements List<Animal> {
    private final List<Animal> animals;
    
    @Override
    public boolean remove(Object o) {
        throw new UnsupportedOperationException("읽기 전용 리스트입니다");
    }
}
```

- `IndexOutOfBoundsException`, `ConcurrentModificationException`은 대부분 프레임워크나 JDK가 알아서 던져주므로, 직접 던질 일은 거의 없다.

  ### IndexOutOfBoundsException

  시퀀스의 허용 범위를 벗어난 값이 전달될 때 사용한다.

    ```java
    public Animal getAnimal(int index) {
        if (index < 0 || index >= animals.size()) {
            throw new IndexOutOfBoundsException("인덱스: " + index + ", 크기: " + animals.size());
        }
        return animals.get(index);
    }
    ```

  ### ConcurrentModificationException

  단일 스레드용 객체를 여러 스레드가 동시에 수정하려 할 때 던진다. 동시 수정을 완벽히 검출하기는 어렵지만, 잠재적 문제를 알리는 역할을 한다.

    ```java
    public void processAnimals() {
        for (Animal animal : animals) {
            if (someCondition) {
                // 반복 중 컬렉션을 수정하면 ConcurrentModificationException 발생
                animals.remove(animal); 
            }
        }
    }
    ```


## 직접 재사용하지 말아야 할 예외들

`Exception`, `RuntimeException`, `Throwable`, `Error`는 직접 재사용하지 말자. 여러 예외의 상위 클래스에 해당되는 이러한 예외들은, 여러 성격의 예외를 포괄하고 있다. 안정적인 테스트가 불가능하니 추상 클래스처럼 생각하는 편이 좋다.

## 커스텀 예외는 언제 사용할까

```java
// ❌ 불필요한 커스텀 예외
public class InvalidAgeException extends IllegalArgumentException {
    public InvalidAgeException(int age) {
        super("Invalid age: " + age);
    }
}
throw new InvalidAgeException(age);

// ✅ 표준 예외로 충분
throw new IllegalArgumentException("나이는 0 이상이어야 합니다: " + age);

// ✅ 비즈니스 예외는 커스텀이 나을 수 있음
public class InsufficientBalanceException extends RuntimeException {
    private final Money required;
    private final Money current;
    
    // 도메인 특화된 정보를 담을 때는 커스텀 예외 고려
}
```

**대부분의 경우 표준 예외로 충분하다.** 커스텀 예외는 정말 필요할 때만 만들자. 이때 핵심은, ‘익숙한 것’을 사용해야 한다는 점이다. 다른 팀원들이 코드를 보자마자 무엇에 대한 예외인지 바로 알 수 있도록 하는 것이 중요하다.