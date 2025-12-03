# 아이템 86

# Serializable을 구현할지는 신중히 결정하라

## 직렬화란 무엇인가

프로그램의 객체를 → 파일이나 네트워크로 보낼 때 바이트 스트림으로 변환해서 보내는것

바이트 스트림 : 객체를 파일저장, 네트워크 전송을 위해 이진 데이터로 바꾸는것

```java
class Person implements Serializable { <- 이것만 추가하면 됨 
    String name = "홍길동"; 
    int age = 25;
}
```

## 하는 방법

```java
User user = new User("홍길동", 30, "hong@example.com");

try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream("user.ser"))) {
    
    oos.writeObject(user);
    
    System.out.println("직렬화 완료: " + user);
    
} catch (IOException e) {
    e.printStackTrace();
}
  
 ------------
 
 try-with-resources 문법으로 파일에 객체를 쓰기가 끝났으면 자동으로 닫아줌(사용한 리소스들을 반환한다는 뜻)
 FileOutputStream("user.ser") : user.ser이란 파일에 쓸 수 있는 도구를 의미
 writeObject(user) : user 객체를 해당 user.ser이란 파일에 쓴다는 것을 의미
 -> 요것이 직렬화의 전체 로직
```

## 근데 왜 DB에 저장할 때는 직렬화를 하지 않는냐?

DB는 객체를 직렬화가 아닌 미리 구성한 테이블에 하나씩 데이터를 넣는다고 생각하면됨

```java
CREATE TABLE users (
    id INT,
    name VARCHAR(100),
    age INT,
    email VARCHAR(100)
);

INSERT INTO users VALUES (1, '홍길동', 25, 'hong@example.com');

이렇게 테이블을 미리 구성하면
객체의 필드를 테이블의 컬럼에 맞추어 넣는것임
-> 요것이 SQL 프로토콜

이 원리는 네트워크를 타는 rds에서도 마찬가지
객체 -> jpa -> sql 문자열 -> 네트워크 -> rds 이 순서대로 rds에서 동작
```

## 언제 사용하나?

여러 예시가 있지만 지금 제가 개인 프로젝트에서 직렬화를 썼는데 그 예시를 들자면

```java
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class OrderCreatedEvent implements Serializable {

    private Long orderId;
    private Long userId;
    private List<Long> cartItemIds;
    private LocalDateTime createdAt;
    private int totalPrice;

    public static OrderCreatedEvent of(Long orderId, Long userId, List<Long> cartItemIds, int totalPrice) {
        return new OrderCreatedEvent(orderId, userId, cartItemIds, LocalDateTime.now(), totalPrice);
    }
}

간단히 소개하면
MSA환경을 구축하고 주문 서비스에서 Order(주문)을 하면 결제 서비스에 주문이 생성되었다고 알려줘야 하는데 그것을 
RabbitMQ를 통해서 알려줌
알려줄 때 위 객체 OrderCreatedEvent를 주문 서비스에서 생성하고 rabbitmq를 publish하고
결제 서비스는 해당 rabbitmq를 subscribe해서 주문 생성 이벤트를 수신함
이게 네트워크를 타기 때문에 바이트 스트림으로 변화해야 하므로 직렬화했음

근데 이번 아이템 공부하면서 알게 된 거는 이렇게 보낼려면 JSON으로 바꿔서 보내는게 
어떤 프로그래밍 언어든 읽을 수 있고, 버전 관리가 더 쉽다고 합니다.

// json으로 할 시 추가해야할 application.yml 설정
spring:
  rabbitmq:
    listener:
      simple:
        message-converter: jackson2JsonMessageConverter
```

## 장점

1. 객체 저장
    
    객체가 메모리에 저장되면 프로그램 종료시 사라진다
    
    객체를 파일로 저장하면 이는 디스크에 저장되기 때문에 나중에  다시 불러오는게 가능
    
2. 객체 전송
    
    네트워크를 통해 다른 컴퓨터로 객체를 보내고 싶으면 직렬화를 통해 바이트 스트림으로 바꿔서 보내야함
    
3. 객체를 복사하기 위해
    
    깊은 복사를 할 때 사용
    
    깊은 복사는 객체 내의 값들은 같지만 다른 주소에 있는 완전히 다른 객체를 만들기 위함인데 이게 직렬화/역직렬화를 통해 가능
    

## 단점

1. 부모 클래스에서는 사용x
    
    부모 클래스에 Serializable를 붙이면 모든 자식 클래스에 강제 직렬화됨
    
    이를 회피할려면 자식 클래스의 필드에 transient를 추가해야함
    
    ```java
    private transient Connection dbConnection; -> 이렇게
    ```
    
    → 해결법 : 그래서 할거면 자식 클래스에 해야함
    
2. 여러 버전이 있는 경우는 사용하면 안됨
    
    버전(객체의 형태)가 달라지면 직렬화 호환성이 깨짐
    
    ```java
    // 버전 1인 user 객체
    public class User implements Serializable {
        private static final long serialVersionUID = 1L;
        
        private String name;
        private int age;
    }
    
    조건 : 이 객체를 직렬화해서 파일에 저장함
    
    // 버전 2인 user 객체
    public class User implements Serializable {
        private static final long serialVersionUID = 2L;
        
        private String name;
        private int age;
    		private Address address;
    }
    
    옛날 Address 필드가 없는 user객체를 읽을려고 하면 Address가 null이 되므로
    NPE가 발생
    
    -> 해결법(JSON)
    @JsonProperty(defaultValue = "")를 하면 없어도 기본값이 ""이 되므로 해결
    그리고 JSON은 여러 프로그래밍 언어를 커버할 수 있어서 더 범용성이 있음
    ```
    
3. 보안 문제
    
    해커가 직렬화한 바이트를 직접 조작할 수 있기 때문에 문제가 됨
    
    → 해결법 : 암호화
    

## 요약

1. 직렬화 = 객체를 저장/전송 가능하게 바꾸는 것
2. 쉽게 만들 수 있지만, 나중에 바꾸기 엄청 어려움
3. 정말 필요한지 고민하고, 필요하면 신중하게 설계할 것