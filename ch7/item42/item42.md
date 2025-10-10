<aside>
💡

익명 클래스보다는 람다를 사용하라

</aside>

### 예전엔…

자바에서 함수를 값처럼 다루고 싶을 때, 단일 추상메서드 인터페이스의 익명 클래스를 만들어 사용하곤 했다고 한다. 이런 방식은 코드가 장황하고 가독성이 떨어졌다. 아래가 그 예시다. 

> **익명 클래스:**
내부 클래스의 일종. 클래스의 선언과 객체의 생성을 동시에 하기 때문에 단 한 번만 사용될 수 있고, 오직 단 하나의 객체만을 생성할 수 있는 일회용 클래스.
> 

```java
// 익명 클래스 방식 - 낡은 기법
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

이 코드에는 Comparator라는 인터페이스가 있고, 이것을 익명 클래스가 구현한다. 실제로 하는 일은 문자열 길이를 비교하는 것뿐이다. 

자바 8부터는 이런 단일 추상 메서드 인터페이스를 **함수형 인터페이스**라 부르며, 람다로 간결하게 표현할 수 있게 되었다.

```java
// 람다 방식 - 간결하고 명확
Collections.sort(words, 
    (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

타입 추론 덕분에 매개변수 타입도 생략할 수 있다. 컴파일러가 문맥을 통해 `(String s1, String s2)`임을 알아낸다. (타입 추론 규칙은 너무 복잡하고, 잘 알지 못해도 상관없다고 한다.)

타입을 명시해야 코드가 더 명확하다면 매개변수 타입을 직접 명시하자. 그 외의 경우에는 전부 생략하면 된다. 

```java
타입 명시해야 하는 예시
```

비교자 생성 메서드와 sort 메서드를 사용하면 더욱 간결하다.

```java
// 비교자 생성 메서드 comparingInt 사용
Collections.sort(words, comparingInt(String::length));

// List.sort 메서드 사용 (Java 8+)
words.sort(comparingInt(String::length));
```

### 열거 타입

열거 타입의 상수별 동작도 람다로 깔끔하게 구현 가능하다.

예전엔 열거 타입에서 상수마다 다른 동작을 구현하기 위해 상수별 클래스 몸체를 사용했다. 역시 장황한 느낌을 준다.

```java
// 상수별 클래스 몸체 방식 
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };
    
    private final String symbol;
    
    Operation(String symbol) {
        this.symbol = symbol;
    }
    
    public abstract double apply(double x, double y);
}
```

람다를 활용하면 이렇게나 간결하다!~ 

각 상수의 동작을 람다로 표현해 생성자에 전달하고, 필드에 저장해두었다가 apply에서 호출하는 식이다. 

```java
// 람다를 활용한 방식
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);
    
    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }
    
    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

(여기서 `DoubleBinaryOperator`는 자바가 제공하는 함수형 인터페이스로, double 두 개를 받아 double을 반환한다.)c

람다가 간결하고 좋다는 사실 정도야 이미 알고 있었으리라 생각된다. 그러니 람다의 한계와 사용 시의 주의점에 주목하자.

### 람다를 사용하지 않아야 하는 경우

1. **코드가 길거나 복잡할 때**
    
    람다는 이름이 없고, 문서화도 못 한다.  코드 자체로 동작이 명확히 설명되지 않고, 줄 수가 많아지면 가독성이 심각하게 떨어진다. 람다는 한 줄일 때 가장 좋고, 길어봐야 세 줄이라고 한다. 세 줄이 넘어간다면 람다를 사용하지 않는 방법을 고려하자. 
    
    예를 들어, 복잡한 로직을 메서드로 추출하면 이름을 통해 의도를 명확히 드러낼 수 있고, 재사용도 가능하며, 테스트하기도 쉽다. 
    
    ```java
    // Bad - 이런 긴 람다는 읽기 어렵고 디버깅도 힘들다...
    list.stream()
        .filter(item -> {
            if (item.getCategory() == null) {
                return false;
            }
            String category = item.getCategory().toLowerCase();
            if (!category.contains("electronics") && !category.contains("books")) {
                return false;
            }
            BigDecimal price = item.getPrice();
            if (price.compareTo(new BigDecimal("10.00")) < 0) {
                return false;
            }
            if (price.compareTo(new BigDecimal("1000.00")) > 0) {
                return false;
            }
            return item.getStock() > 0;
        })
        .collect(Collectors.toList());
    
    // Good - 복잡한 로직은 메서드로 추출하자 
    list.stream()
        .filter(this::isValidItem)
        .collect(Collectors.toList());
    
    private boolean isValidItem(Item item) {
        if (item.getCategory() == null) {
            return false;
        }
        
        String category = item.getCategory().toLowerCase();
        boolean isValidCategory = category.contains("electronics") 
                                || category.contains("books");
        
        BigDecimal price = item.getPrice();
        boolean isValidPrice = price.compareTo(new BigDecimal("10.00")) >= 0
                             && price.compareTo(new BigDecimal("1000.00")) <= 0;
        
        return isValidCategory && isValidPrice && item.getStock() > 0;
    }
    ```
    
2. **추상 클래스 인스턴스가 필요할 때**
    
    람다는 함수형 인터페이스에서만 작동한다. 추상 클래스의 인스턴스를 만들고자 한다면 익명 클래스를 써야 한다.
    
    ```java
    // 추상 클래스 정의
    abstract class Animal {
        protected String name;
        
        public Animal(String name) {
            this.name = name;
        }
        
        abstract void makeSound();
        
        public void introduce() {
            System.out.print("I'm " + name + ". ");
            makeSound();
        }
    }
    
    // 람다로는 불가능 - 컴파일 에러!
    // Animal cat = () -> System.out.println("Meow");  
    
    // 익명 클래스를 사용해야 함
    Animal cat = new Animal("Kitty") {
        @Override
        void makeSound() {
            System.out.println("Meow");
        }
    };
    
    cat.introduce();  // I'm Kitty. Meow
    ```
    
3. **여러 추상 메서드가 있는 인터페이스**
    
    람다는 단 하나의 추상 메서드만이 존재하는 함수형 인터페이스에서 사용 가능하다. 추상 메서드가 여러 개라면 익명 클래스를 써야 한다.
    
    ```java
    // 여러 추상 메서드를 가진 인터페이스
    interface MouseListener {
        void mouseClicked(MouseEvent e);
        void mousePressed(MouseEvent e);
        void mouseReleased(MouseEvent e);
    }
    
    // 람다로는 불가능 - 어느 메서드를 구현하는지 모호함
    // MouseListener listener = (e) -> System.out.println("Clicked");  // 컴파일 에러!
    
    // 익명 클래스로 구현
    MouseListener listener = new MouseListener() {
        @Override
        public void mouseClicked(MouseEvent e) {
            System.out.println("Mouse clicked at: " + e.getX() + ", " + e.getY());
        }
        
        @Override
        public void mousePressed(MouseEvent e) {
            System.out.println("Mouse pressed");
        }
        
        @Override
        public void mouseReleased(MouseEvent e) {
            System.out.println("Mouse released");
        }
    };
    ```
    
4. **자기 자신을 참조해야 할 때** 
람다에서 this는 람다를 감싸는 바깥 인스턴스를 가리킨다. 반면 익명 클래스의 this는 익명 클래스 자신을 가리킨다. 함수 객체가 자신을 참조해야 한다면 익명 클래스를 써야 한다.
    
    ```java
    public class SelfReferenceExample {
        private String outerField = "Outer";
        
        public void demonstrateThis() {
            // 람다에서의 this는 바깥 클래스(SelfReferenceExample)를 가리킴
            Runnable lambdaRunnable = () -> {
                System.out.println("Lambda this.outerField: " + this.outerField);
                // System.out.println(this);  // SelfReferenceExample 인스턴스 출력
            };
            
            // 익명 클래스에서의 this는 익명 클래스 자신을 가리킴
            Runnable anonymousRunnable = new Runnable() {
                private String innerField = "Inner";
                
                @Override
                public void run() {
                    System.out.println("Anonymous class innerField: " + this.innerField);
                    System.out.println("Anonymous class outerField: " + 
                        SelfReferenceExample.this.outerField);  // 바깥 참조는 이렇게
                }
            };
            
            lambdaRunnable.run();      // Lambda this.outerField: Outer
            anonymousRunnable.run();    // Anonymous class innerField: Inner
                                       // Anonymous class outerField: Outer
        }
    }
    ```
    

### 람다 사용 시 주의사항 - 직렬화 금지

람다를 직렬화하는 일은 극히 삼가야 한다고 소개한다. 익명클래스에도 해당되는 문제인데, 직렬화 형태가 구현별로 다를 수 있다고 한다(JVM에 따라 구현 방법이 달라서). 단순히 동작하고 안 하고의 문제가 아니라, 개발 환경에선 잘 돌아가던 것이 운영 환경에서 터질 수도 있다 하니 하지 말라는 건 하지 말도록 하자…

### Q. 람다를 직렬화하는게 어떤 행위죠 코드상으로??

```java
public static void main(String[] args) throws Exception {
    File file = Files.createTempFile("lambda", "ser").toFile();
    try (ObjectOutput oo = new ObjectOutputStream(new FileOutputStream(file))) {
        Runnable r = () -> System.out.println("Can I be serialized?");
        oo.writeObject(r);
    }

    try (ObjectInput oi = new ObjectInputStream(new FileInputStream(file))) {
        Runnable  r = (Runnable) oi.readObject();
        r.run();
    }
}

Exception in thread "main" java.io.NotSerializableException: Test$$Lambda$18/0x0000000301001a08
	at java.base/java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1187)
	at java.base/java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:350)
	at Test.main(Test.java:9)
```
