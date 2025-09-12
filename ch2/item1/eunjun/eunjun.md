# Item 1. 생성자 대신 정적 팩토리 메서드를 고려하라

- 장점
    1. 이름을 가질 수 있다.
        
        ```jsx
        // 일반 생성자, 직관적으로 의미를 알기 힘듦
        User u1 = new User("홍길동", true);  
        User u2 = new User("홍길동", false); 
        
        // 정적 팩토리 메서드, 이름을 통해 의도를 알 수 있음
        public class User {
            private String name;
            private boolean admin;
        
            private User(String name, boolean admin) {
                this.name = name;
                this.admin = admin;
            }
        
            public static User createAdmin(String name) {
                return new User(name, true);
            }
        
            public static User createGuest(String name) {
                return new User(name, false);
            }
        }
        
        // 호출
        User admin = User.createAdmin("홍길동");
        User guest = User.createGuest("철수");
        
        ```
        
    2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
        
        ```jsx
        public class User {
            private static final User DEFAULT = new User("default");
        
            private String name;
            private User(String name) { this.name = name; }
        
            public static User of(String name) {
                if (name.equals("default")) {
                    return DEFAULT;  // 이미 만들어둔 객체 재사용
                }
                return new User(name);  // 필요시만 새로 생성
            }
        }
        ```
        
        메서드 안에 생성자가 포함이 되는 구조이므로, 다른 로직을 넣어 캐싱을 구현할 수도 있다.
        
    3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
        - 자바 8 이전에는 인터페이스에 정적 메서드를 선언할 수 없었다. 그렇기 때문에 이름이 “Type”인 인터페이스를 반환하는 정적 메서드가 필요하면 “Types”라는 인스턴스화 불가인 동반클래스를 만들어 그 안에 정의하는 것이 관례였다.
            
            ```jsx
            public interface Type {
                // 구현체의 API만 정의
            }
            
            public final class Types {
                private Types() {} // 다른 곳에서 사용 불가능하도록 생성자 private
            
                public static Type of(...) {
                    return new SomeTypeImpl(...);
                }
            }
            
            ```
            
            → 왜 이게 불가능했을까? 
            
            static은 객체가 아니라 클래스 자체에 속해서(동적 바인딩 X, 정적 바인딩), 부모 클래스를 상속받은 자식 클래스에서 오버라이딩할 수 없다. 인터페이스는 구현이 담겨있으면 안된다는 철학때문에 인터페이스에 static을 담을 수 없었으나, 매번 동반클래스를 작성하는 등 번거로움이 있었기에 변경됐다.
            
    4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
        
        ```jsx
        // 생성자를 사용하면 명시적으로 다른 객체를 사용해야 함
        Shape s1 = new Circle(); 
        Shape s2 = new Square(); 
        
        // 정적 팩토리 메서드를 사용하면 인자에 따라 다른 클래스(또는 하위 타입) 객체 반환 가능
        public interface Shape {}
        
        class Circle implements Shape {}
        class Square implements Shape {}
        
        class ShapeFactory {
            public static Shape of(String type) {
                if (type.equals("circle")) return new Circle();
                else if (type.equals("square")) return new Square();
                throw new IllegalArgumentException("unknown type");
            }
        }
        
        // 호출
        Shape s1 = ShapeFactory.of("circle");
        Shape s2 = ShapeFactory.of("square");
        
        ```
        
    5. 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
        
        → 동현님, 준서형 자료 참고
        
- 단점
    1. 상속하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위클래스를 만들 수 없다.
        
        → 애초에 정적 팩토리 메서드만 제공하는 클래스는 상속 전용이 아니라 해당 메소드들만을 위한, 즉 컬렉션같은 클래스 아닌가? 이게 왜 단점이 되는지 잘 이해가 안감.
        
    2. 정적 팩토리 메서드는 프로그래머가 찾기 어렵다.
        
        → 이것도 단점인지 잘 모르겠음. 생성자도 매개변수가 다양해진다면 코드나 문서를 봐야하는 건 똑같을텐데, 오히려 정적 팩토리 메서드를 추가하는게 귀찮다는 단점이 더 크지않나?