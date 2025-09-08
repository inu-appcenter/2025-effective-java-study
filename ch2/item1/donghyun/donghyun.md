# 아이템 1

## 생성자 대신 정적 팩터리 메서드를 고려하라

### 장점

1. 메서드의 의미를 나타내는 이름을 가질 수 있다.
    
    ```java
    @Getter
    public class RequestAnnouncementDto {
    
        private String title;
        private String writer;
        private String content;
        private Boolean isEmergency;
    
        public static Announcement dtoToEntity(RequestAnnouncementDto requestAnnouncementDto) {
            return Announcement.builder()
                    .title(requestAnnouncementDto.getTitle())
                    .writer(requestAnnouncementDto.getWriter())
                    .content(requestAnnouncementDto.getContent())
                    .isEmergency(requestAnnouncementDto.getIsEmergency())
                    .build();
        }
    
    }
    ```
    
2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
    
    *싱글톤에 사용
    
    ```java
    public class Book {
    
        private static Book book = new Book(); // JVM에 의해서 클래스가 메모리 공간에 올라가는 순간
    
        private Book() {
        }
    
        public static Book getBookInstance(){
            return book;
        }
    }
    ```
    
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다. 
    - 생성자의 반환타입은 반드시 그 클래스의 타입이어야 한다.
    
    ```java
    Rate : 부모
      |
    Challenger, Diamond, Silver
    
    public interface Rate {
        int score();
        
        static Rate createChallenger(int score) {
            return new Challenger(score);
        }
        
        static Rate createDiamond(int score) {
            return new Diamond(score);
        }
        
        static Rate createSilver(int score) {
            return new Silver(score);
        }
        
        default String description() {
            return String.format("%s : %d", this.getClass().getSimpleName(), score());
        }
    }
    
    -------
    
    public class Main{
        public static void main(String[] args) {
            Rate challenger = Rate.createChallenger(2400);   
         == Rate challenger = new Challenger(2400);
        }
    }
    ```
    
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    
    ```java
    Animal - 부모
    Cat - Animal을 상속받은 자식
    Dog - Animal을 상속받은 자식
    
    public static Animal makeAnimal(String name) {
    	if (name == "개") {
    		return new Dog();
    	} else if (name == "고양이") {
    		return new Cat();
    	}
    }
    
    ```
    

 

1. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    - 컴파일 시점
        
        ```java
        public class DatabaseManager {
            public Connection createConnection(String dbType) {
                if ("mysql".equals(dbType)) {
                    return new MySQLConnection();     // MySQLConnection 클래스 필요
                } else if ("postgres".equals(dbType)) {
                    return new PostgreSQLConnection(); // PostgreSQLConnection 클래스 필요
                } else if ("oracle".equals(dbType)) {
                    return new OracleConnection();     // OracleConnection 클래스 필요
                }
                return null;
            }
        }
        ```
        
        컴파일 시점에 모든 구현 클래스가 존재해야 함
        
        새 DB 추가시 코드 수정 필요
        
    - 런타임 시점
        
        ```java
        Connection conn = DriverManager.getConnection(url);
        
        ------
        
        DriverManager.java
        public static Connection getConnection(String url) {
           for (Driver driver : registeredDrivers) {
               if (driver.acceptsURL(url)) {
                   return driver.connect(url, properties);
               }
           }
           throw new SQLException("No suitable driver");
        }
        ```
        
        컴파일 시점에는 Connection 인터페이스만 알면 됨
        
        구체적인 구현 클래스는 런타임에 결정
        
        새 DB 추가해도 코드 수정 불필요
        
        ### **런타임 vs 컴파일**
        
        컴파일 시점
        
        - **언제**: .java → .class 파일로 변환할 때
        - **무엇을 결정**: 문법 검사, 타입 검사, 메서드 존재 여부 등
        - **정적으로 결정됨**: 코드에 명시적으로 적힌 내용만 확인
        
        런타임 시점
        
        - **언제**: 프로그램이 실제로 실행될 때
        - **무엇을 결정**: 실제 객체 생성, 메서드 호출, 조건문 처리 등
        - **동적으로 결정됨**: 실행 중 상황에 따라 달라짐
    
    참고사항 : return driver.connect(url, properties); 그럼 이 때 Driver connect도 컴파일 시험에 확인되는 것이 아닌가하는 의문이 든다. 하지만 MySQLDriver, Connection, Statement 같은 mysql 코드는 서비스 애플리케이션과 별도로 컴파일이 된다.  즉 컴파일 시점에서는 서비스 애플리케이션에서는 Connection 타입만 알면 되고 Mysql은 Driver의 connect 메서드만 만들면 된다. 그리고 런타임 시점(즉 프로그램을 실행할 때) 둘이 만난다.
    

### 단점

1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
    
    ```java
    class ImmutablePerson {
        private ImmutablePerson(String name);
        public static ImmutablePerson of(String name);
    }
    
    class Employee extends ImmutablePerson {
        public Employee(String name, String company) {
            super(name);
        }
    }
    ```
    
    - 자바의 문법에서 상속을 받는 자식의 생성자 가장 첫번째 줄은 부모 생성자를 super로 호출하는 것인데 부모에서 생성자 대신 정적 메서드만 public으로 하고 부모 생성자는 private로 하면 자식 생성자는 컴파일 오류가 발생함
    
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
    
    ```java
    public static DatabaseConnection get() { }      // 무엇을 get?
    -> public static DatabaseConnection getInstance() { }
    ```