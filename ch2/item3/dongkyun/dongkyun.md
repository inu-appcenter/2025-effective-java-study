# item3

## 📌 private 생성자나 열거 타입으로 싱글턴임을 보증하라

---

- **싱글턴(singleton)**이란? 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말합니다.



- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있습니다.
    - 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문입니다.



- 싱글턴을 만드는 두 가지 방법
  모두 생성자는 private으로 감추고 접근할 수 있는 수단으로 public static 멤버를 만듭니다.

    1. public static 멤버가 final 필드인 방식
        
        ```java
        public class Elvis {
        		public static final Elvis INSTANCE = new Elvis();
        		private Elvis() { ... }
        		public void leaveTheBuilding() { ... }
        }
        ```
        
        - public static final 필드를 초기화할 때 딱 한 번만 호출됩니다.
            - 만들어진 인스턴스가 하나 뿐임이 보장됩니다.
        - 단 한가지 예외는, 권한이 있는 클라이언트의 리플렉션 API인AccessibleObject.setAccessible을 호출해 private 생성자를 호출 할 수 있습니다.
            - 이를 방어하려면, 두 번째 객체 생성 전 예외를 던지게 해야합니다.
        - 장점
            - 싱글턴임이 API에 명백히 드러납니다.
            - 간결합니다.
            
    2. 정적 팩터리 메서드를 public static 멤버로 제공
        
        ```java
        public class Elvis {
        		private static final Elvis INSTANCE = new Elvis（）;
        		private Elvis（） { ... }
        		public static Elvis getlnstance() { return INSTANCE; }
        		public void leaveTheBuilding（） { ... }
        }
        ```
        
        - Elvis.getInstance 는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis 인스턴
        스란 결코 만들어지지 않습니다. → ( 리플렉션 예외는 똑같이 적용됩니다. )
        - 장점
            - API를 바꾸지 않고도 싱글턴이 아니게 변경 가능합니다.
            - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있습니다.
            - 정적 팩터리 메서드 참조를 공급자(supplier)로 사용할 수 있습니다.

    ❌ 둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 
    단순히 Serializable을 구현한다고 선언하는 것만으로는 부족합니다. 
    모든 인스턴스 필드를 일시적 (transient)이라고 선언하고 readResolve 메서드를 제공해야 합니다.
    
    → 없을 경우 역직렬화 시점마다 새로운 인스턴스가 생성됩니다.
    
    ```java
    private Object readResolve() {
    		// •진짜, Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    		return INSTANCE;
    }
    ```


- 싱글턴을 만드는 세 번째 방법인 원소가 하나인 **열거 타입을 선언**

    ```java
    public enum Elvis {
    		INSTANCE;
    		public void leaveTheBuilding() { ... }
    }
    ```

    - 간결하고 간편합니다.
    - 복잡한 직렬화 상황이나 리플렉션 공격에도 제 2의 인스턴스 생성을 완전 막습니다.
    - 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법입니다.

  → 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다
  → 자바는 **다중 상속을 지원하지 않습니다** → 이미 Enum을 상속했기 때문에 다른 클래스를 상속 불가