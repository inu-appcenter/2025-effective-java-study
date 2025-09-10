# item1

## 🏭 생성자 대신 정적 팩터리 메서드를 고려하라

👍🏻 정적 팩터리 메서드가 생성자보다 좋은 장점 다섯가지

---

1. *이름을 가질 수 있다.*

   → **이름**을 통해 반환될 객체의 특성을 쉽게 묘사할 수 있습니다.

   → 한 클래스에 시그니처가 같은 생성자가 여러 개 필요하면, 정적 팩터리 메서드를 사용하고 이름에 차이를 잘 명시해야합니다.
   ( 이때, 시그니처는 **컴파일러가 해당 메서드를 식별할 수 있는 정보**를 말합니다. (메서드 이름, 매개변수 타입 or 순서 ))


2. *호출 될 때마다 인스턴스를 새로 생성하지는 않아도 된다.*

   → 불변 클래스는 인스턴스를 미리 만들어놓거나 새로 생성한 인스턴스를 캐싱해 재활용합니다.

   → 플라이웨이트 패턴도 이와 비슷한 기법
   ( 여기서 **폴라이웨이트 패턴**이란?)
    - 객체를 공유해서 메모리 사용량을 줄이는 구조 패턴
    - 같은 속성을 가진 객체는 여러 개 만들지 않고, 한 개만 만들어서 재사용

    ```java
    class AppCenter {
    		private final String name;    // 내부 상태: 공유 가능
    		private final int age;       // 내부 상태: 공유 가능
    }
    
    interface Study {
        void coding(int x, int y); // 외부 상태(x,y)를 받아서 출력
    }
    ```

   → 이런 같은 객체를 반환하는 클래스를 **인스턴스 통제 클래스**라고 합니다.

   [ 그래서 통제하는 이유가 무엇인가요? ]

   → 클래스를 싱글턴 or 인스턴스화 불가로 만들 수 있습니다.

   → 불변 값 클래스에서 동치인 인스턴스가 단 하나 뿐임을 보장할 수 있습니다.



3. *반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.*

   → 반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 가집니다.

   → 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크의 핵심

   → 클라이언트는 얻은 객체를 구현 클래스가 아닌 인터페이스만으로 다루게 됩니다. ( 좋은 습관 )

    ```java
    // 클라이언트가 ArrayList에 직접 의존
    List<String> list = new ArrayList<>(); 
    
    // 실제 반환은 ArrayList
    public static List<String> newList() {
        return new ArrayList<>();  // 내부 구현만 교체해도 클라이언트는 영향 없음
    }
    ```




4. *입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.*

   → 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없습니다.

   → 클라이언트는 팩터 리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없습니다.

    ```java
    interface Shape {
        void draw();
    }
    
    class Circle implements Shape {
        public void draw() { System.out.println("Drawing a Circle"); }
    }
    
    class Square implements Shape {
        public void draw() { System.out.println("Drawing a Square"); }
    }
    ```



5. *정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다*

   → 정적 팩터리 메서드는 반환 타입을 인터페이스(또는 추상 클래스)로 지정해두면, 실제 어떤 구현체를 반환할지는 런타임 시점에 정할 수 있습니다.

   [ **서비스 제공자 프레임워크** ]
   이 개념을 이용해서 만들어진 것이 바로 **서비스 제공자 프레임워크입니다.**

    1. 서비스 인터페이스 → 구현체의 동작을 정의
    2. 제공자 등록 API → 제공자가 구현체를 등록
    3. 서비스 접근 API → 클라이언트가 서비스의 인스턴스를 얻을 때 사용
    4. 서비스 제공자 인터페이스 → 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명

   EX) **JDBC**
    - Connection은 **서비스 인터페이스(계약)**, Driver는 **서비스 제공자(구현/공장)**, DriverManager.registerDriver는 **제공자 등록**, DriverManager.getConnection은 **서비스 접근 API**

    - 애플리케이션은 getConnection()만 호출, 어떤 드라이버가 실제 Connection을 반환할지는 프레임워크가 결정합니다 → 이게 서비스 제공자 프레임워크의 핵심 장점입니다.


---

👎🏻 정적 팩터리 메서드의 단점 두가지

---

1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

   → 구현 클래스들은 상속할 수 없습니다.

   → 이때,
   *“상속보다는 컴포지션 (*필드로 포함**)***을 사용하라”* 
    *”상속을 허용하면, 하위 클래스에서 setter나 가변 필드를 추가해 불변성을 깨버릴 수 있다.”*
   따라서, 오히려 장점으로도 볼 수 있습니다.



2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

   → 생성자처럼 API 설명에 명확히 드러나지 않습니다.


[ **정적 팩터리 메서드 명명 방식** ]

- **from**： ****매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드

    ```java
    public static MyPageResponse from(Member member) {
        return new MyPageResponse(
                member.getId(),
                member.getStudentId(),
                member.getDepartment().getFullName(),
                member.getStudentType().getDescription(),
                member.getCountry().getFullName(),
                member.getNickname(),
                member.getImageUrl()
        );
    }
    ```

- **of**： ****여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드

    ```java
    public static FeedbackResponse of(Long memberId, String content) {
        return new FeedbackResponse(memberId, content);
    }
    ```

- **valueOf**： ****from과 of의 더 자세한 버전
- **instance** 혹은 **getlnstance**：(매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.
- **create** 혹은 **newlnstance :** instance 혹은 getlnstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
- **getType**： getlnstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
- **newlype:** newlnstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
- **type *:*** getType과 newType의 간결한 버전