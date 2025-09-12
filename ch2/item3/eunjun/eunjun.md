# Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

1. private 생성자 + public static final 필드 방식의 싱글턴
    
    ```jsx
    public class Elvis {
     public static final Elvis INSTANCE = new Elvis(); // 생성자는 외부에서 호출 불가능, 접근만 가능
     private Elvis() { ... }
     public void leaveTheBuilding() { ... } 
    }
    ```
    

1. 정적 팩토리 방식의 싱글턴
    
    ```jsx
    public class Elvis {
     private static final Elvis INSTANCE = new Elvis();
     private Elvis() { ... }
     public static Elvis getlnstance { return INSTANCE; }
    }
    ```
    
    2번 방식의 장점은 다음과 같다.
    
    1. 원할 때 싱글턴을 유지하지 않을 수 있다.
        - 정적 팩토리 메서드 안에서 new를 이용하거나 스레드마다 다른 객체를 생성하게 할 수 있다.
        
    2. 정적 팩토리를 제네릭 싱글턴 팩토리로 만들 수 있다.
        
        ```jsx
        interface IdentityFunction<T> {
            T apply(T t);
        }
        
        public class GenericSingletonFactory {
            // 싱글턴 객체 (타입을 Object로 고정)
            private static final IdentityFunction<Object> IDENTITY = x -> x;
        
            // 제네릭 싱글턴 팩토리 메서드
            public static <T> IdentityFunction<T> identityFunction() {
                // 하나의 IDENTITY 객체를 필요에 따라 T 타입으로 변환
                return (IdentityFunction<T>) IDENTITY;
            }
        
            public static void main(String[] args) {
                // String용
                IdentityFunction<String> f1 = GenericSingletonFactory.identityFunction();
                System.out.println(f1.apply("hello")); // hello
        
                // Integer용
                IdentityFunction<Integer> f2 = GenericSingletonFactory.identityFunction();
                System.out.println(f2.apply(123)); // 123
        
                // Double용
                IdentityFunction<Double> f3 = GenericSingletonFactory.identityFunction();
                System.out.println(f3.apply(3.14)); // 3.14
            }
        }
        
        ```
        
    3. 정적 팩토리의 메소드 참조를 공급자로 사용할 수 있다.
        
        ```jsx
        // private 생성자 + public static final 방식
        public class Elvis {
            public static final Elvis INSTANCE = new Elvis(); // 공개된 인스턴스
            private Elvis() {} // 외부에서 new 못함
        
            public void sing() {
                System.out.println("🎵 I'm Elvis 🎵");
            }
        }
        
        --> 어떤 API가 supplier<Elvis>를 받는다고 하면, INSTANCE는 이미 객체이기 때문에
            Supplier<Elvis> sup = Elvis.INSTANCE::??? 같은 방식으로 메소드 참조가 불가능
            
        --> 때문에 supplier<Elvis> sup = () -> Elvis.INSTANCE; 같이 람다를 사용해야 함.
        
        // 정적 팩토리 메소드 방식
        public class Elvis {
         private static final Elvis INSTANCE = new Elvis();
         private Elvis() { ... }
         public static Elvis getlnstance { return INSTANCE; }
        }
        
        --> 이렇게 정적 팩토리 메소드를 사용하면 
            Supplier<Elvis> sup = Elvis::getInstance; 같이 메소드 참조 가능
        ```
        
        여기서 공급자, supplier란 함수형 인터페이스다. 함수형 인터페이스란 추상 메소드를 단 하나만 가지고 있는 인터페이스다. 자바 8부터 JDK가 자주 쓰이는 함수형 인터페이스 표준을 제공한다. 
        
        supplier 이외에도 consumer, function, operator, predicate 등이 있다. 이 덕분에 개발자들은 매번 함수형 인터페이스를 정의할 필요가 없고,  Optional, Stream 등 API를 일관적이게 작성할 수 있다.
        
        https://lasbe.tistory.com/68
        
        함수형 인터페이스가 쓰이는 이유는 람다를 사용하기 위해서다. 여기서 람다란 익명클래스를 더 쉽게 사용하기 위한 자바의 문법이다. 
        
        ```jsx
          // 자바의 supplier 코드
        	 @FunctionalInterface
        	 public interface Supplier<T> {
            T get();
           }
           
          
          // 현재 UniClub에서 사용된 코드, 람다 사용
          Club club = clubRepository.findById(clubId)
              .orElseThrow(() -> new CustomException(ErrorCode.CLUB_NOT_FOUND));
              
              
          // 위 코드를 익명클래스로 사용한 경우
           Club club = clubRepository.findById(clubId)
             .orElseThrow(new Supplier<CustomException>() {
                 @Override
                 public CustomException get() {
                     return new CustomException(ErrorCode.CLUB_NOT_FOUND);
                 }
             });
              
              
           // 익명클래스를 사용하지 않은 경우   
           Optional<Club> optionalClub = clubRepository.findById(clubId);
           Club club;
           if (optionalClub.isPresent()) {
               club = optionalClub.get();
           } else {
               throw new CustomException(ErrorCode.CLUB_NOT_FOUND);
           }
        ```
        
        람다를 한 번 더 간결하게 사용할 수 있도록 만든 자바의 문법이 메서드 참조다.
        
        ```jsx
          // UniClub 코드에서 권한을 가져오는 부분
          String authorities = authentication.getAuthorities().stream()
              .map(GrantedAuthority::getAuthority)  // 메서드 참조 사용(::)
              .collect(Collectors.joining(","));
        ```
        
    
    그러나, 방법 1, 2는 객체를 직렬화/역직렬화 하는 과정에서 문제가 생길 수 있다. 여기에서 말하는 직렬화는 객체 → 바이트 스트림으로의 변환이다. 네트워크 전송이나 디스크 저장에 사용된다. 역직렬화는 바이트 스트림 → 객체로의 변환이다.
    
    ```jsx
    class Person implements Serializable {
        String name;
        Person(String name) { this.name = name; }
    }
    
    Person p1 = new Person("Alice");
    
    // 직렬화 후 역직렬화
    Person p2 = deserialize(p1);
    
    System.out.println(p1 == p2); // false
    System.out.println(p1.name.equals(p2.name)); // true
    ```
    
    위 코드를 보면 싱글톤 객체를 역직렬화하는 과정에서 싱글톤이 아니게 된다는 것을 알 수 있다. 이를 해결하기 위해 인스턴스 필드를 transient 선언하고, readResolve()를 정의해 사용한다.
    
    항상 직렬화 ↔ 역직렬화 과정에서 같은 객체임을 보장하게 하려면 변환 과정에서 객체 고유 ID까지 저장해야 할텐데, 그렇게 된다면 자원을 잡아먹는 반면 직렬화와 역직렬화 과정에서 늘 같은 객체임을 보장해야 하는 경우가 많지 않아 그 책임을 개발자에게 준게 아닐까.. 라는게 내 생각
    
    https://jjingho.tistory.com/140 
    
    1. 열거타입 방식의 싱글턴 - 권장되는 방법
        
        
        ```jsx
        public enum Elvis {
            INSTANCE;  // 싱글턴 인스턴스
        
            public void sing() {
                System.out.println("🎵 I'm Elvis, the one and only!");
            }
        }
        
        public class Main {
            public static void main(String[] args) {
                Elvis elvis1 = Elvis.INSTANCE;
                Elvis elvis2 = Elvis.INSTANCE;
        
                System.out.println(elvis1 == elvis2); // true
                elvis1.sing();
            }
        }
        ```
        
        - 장점
            1. 싱글턴을 만드는 방법 1, 2와 다르게 직렬화를 위해 readResolve를 쓰는 등 추가적인 노력이 필요 없다.
            2. 방법 1, 2는 리플렉션으로 private 생성자를 호출하면 싱글턴이 아니게 되는 경우가 생기지만, enum은 리플렉션으로도 인스턴스 추가가 불가능하다.
            3. 코드가 간결하다.
    
    - 단점
        1. enum으로 만든 싱글턴은 다른 클래스를 상속받을 수 없다.
        2. 다른 코드에 비해 조금 부자연스러워 보일 수 있다.