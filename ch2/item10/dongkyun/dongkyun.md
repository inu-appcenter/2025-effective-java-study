# item10

## 🟰 equals는 일반 규약을 지켜 재정의하라

---

- 재정의하지 않는 것이 최선인 조건들
    1. *각 인스턴스가 본질적으로 고유하다.*
        1. 값을 표현하는게 아닌 동작하는 개체를 표현하는 클래스

           → ex) Thread

    2. *인스턴스의 ‘논리적 동치성(logical equality)’을 검사할 일이 없다.*
        1. java.util.regex.Pattern은 equals를 재정의하여 같은 정규표현식을 나타내는지 검사하는 방법도 존재
        2. BUT, 필요 없을 경우, 기본 equals만으로 해결
    3. *상위 클래스에서 재정의한 **equals**가 하위 클래스에도 딱 들어맞는다.*
        1. 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 사용
        2. List 구현체들은 AbstractList로부터
        3. Map 구현체들은 AbstractMap으로부터
    4. *클래스가 private이거나 package-private이고 **equals** 메서드를 호출할 일이 없다.*

        ```java
        // 방어
        @Override public boolean equals(Object o) {
        		throw new AssertionError(); // 호출 금지!
        }
        ```


`🤔 그럼 도대체 언제 정의해야 하는데요?`

- 객체 식별성 (물리적으로 같은지)가 아닌 논리적 동치성을 확인해야 함
+ 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때 !

  → ex) 값 클래스 (Integer, String, …)

  → 값이 같은 지를 비교하기위해 equals가 논리적 동치성을 확인하도록 재정의하면
  기대에 부응, Map의 키와 Set의 원소로 사용 가능

    - But, 값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어질 일이 없다면 재정의 필요 ❌

      → 이를 `인스턴스 통제 클래스`라고 부름
      → ex) Enum


- equals 메서드 재정의 시 따라야하는 일반 규약들

  > equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.
  >
    1. *반사성**(reflexivity)***： ****null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
    2. *대칭성**(symmetry)***： ****null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equal 이 x) 도 true 다.
    3. *추이성**(transitivity)***： ****null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
    4. *일관성**(consistency)***： ****null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
    5. ***null-**아님*: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

  → ⚠️ 수많은 클래스는 전달 받은 객체가 equals 규약을 지킨다고 가정하고 동작
  → 규약을 어기면 안됨

- **동치관계**란?
    - 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산이다

      → 이 부분집합을 동치류(equivalence class; 동치 클래스)라 한다

- 동치 관계를 만족시키기 위한 다섯 가지 요건
    1. *반사성*
       : 객체는 자기 자신과 같아야 한다.
       2. *대칭성*
       : 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.

        ```java
        // 대칭성 위배!
        (QOverride public boolean equals(Object o) {
        		if (o instanceof CaselnsensitiveString)
        				return s.equalslgnoreCase(
        						((CaselnsensitiveString) o).s);
        		if (o instanceof String) // 한 방향으로만 작동한다!
        				return s.equalslgnoreCase((St ring) o);
        		return false;
        }
        ```

        - cis.equals(s)는 true를 반환
        - s.equals(cis)는 false를 반환

          → ⚠️ 대칭성 위배
          → String의 equals는 CaseInsensitiveString의 존재를 모른다

        ```java
        List<CaseInsensitiveString> list = new ArrayListo();
        list.add(cis);
        ```
        
        - list.contains(s)를 호출하면 false 반환 or true or RuntimeException
            
            → **equals** 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.
            
    3. *추이성*
    : 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 
    첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻
        
        ```java
        // p1.equals(p2)와 p2.equals(p3)는 true
        // p1.equals(p3)가 false
        ColorPoint pl = new ColorPoint(1, 2, Color.RED);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
        ```
        
        → 구체 클래스를 확장해 새로운 값을 추가하면서 **equals** 규약을 만족시킬 방법은 존재하지 않는다.
        
        ```java
        @Override public boolean equals(Object o) {
        		if (o == null 11 o.getClass() != getClassO)
        				return false;
        		Point p = (Point) o;
        		return p.x = x && p.y = y;
        }
        ```
        
        `🤔 그럼 equals가 같은 구현 클래스의 객체와 비교할 때만 true를 반환하게 하면 되지 않나?`
        
        → Point의 하위 클래스는 어디서든 Point로써 활용될 수 있어야 한다.
        → ❌ 그러지 못한다. 
        
        - *리스코프 치환 원칙 (Liskov substitution principle)*
            - 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.
            - 따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.
        
        - *컬렉션 구현체에서 주어진 원소를 담고 있는지를 확인하는 방법*
            - 대부분의 컬렉션은 이 작업에 equals 메서드를 이용
        - 구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법이 하나 있다.
            
            → 상속 대신 컴포지션을 사용하라
            
            - Point를 상속하는 대신 Point를 ColorPoint의 private 필드로 두고
            - ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰(view) 메서드를 public으로 추가하는 방식
                
                ```java
                public class ColorPoint {
                		private final Point point;
                		...
                		public Point asPointO {
                				return point;
                		}
                		...
                }
                ```
                
                ex) java.sql.Timestamp
                → java.util.Date를 확장한 후 nanoseconds 필드를 추가
                → 대칭성을 위배
                → Date와 섞어 쓸 때 주의 필요
                
    4. *일관성*
    : 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 
    앞으로도 영원히 같아야 한다는 뜻
        - 클래스가 불변이든 가변이든 **equals**의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.

            ex) java.net.URL의 equals는 네트워크를 통하고 항상 결과가 같다고 보장할 수 없다.
            → 😧 문제
            
            → 이런 문제를 피하려면 equals는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야 한다
            
    5. *null-아님*
    : 모든 객체가 null과 같지 않아야 한다는 뜻
        - equals는 건네받은 객체를 적절히 형변환한 후 필수 필드들의 값을 알아내야 한다.
            - 그러려면 형변환에 앞서 instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사해야 한다
            - equals가 타입을 확인하지 않으면 잘못된 타입이 인수로 주어졌을 때 
            ClassCastException을 던져서 일반 규약을 위배하게 된다

- 🪜 양질의 equals 메서드 구현 방법 단계
    1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    2. **instanceof** 연산자로 입력이 올바른 타입인지 확인한다.
    3. 입력을 올바른 타입으로 형변환한다.
    4. 입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사한다

- 비교 연산자
    - float, double → ==
    - 참조 타입 필드 → equals
    - float, double → .compare()
      → 부동 소수점을 다뤄야하여 특별 취급
      → .equals()도 가능하지만 오토 박싱으로 성능상 단점
    - 배열 → 모든 원소가 핵심 필드라면 Arrays.equals 메서드들 중 하나를 사용
    - null → Objects.equals()로 NullPointerException 방지

- 비교 비용
    - 최상의 성능을 위해서 해당 필드를 `먼저 비교`
        - 다를 가능성이 큰
        - 비교하는 비용이 싼

- 주의사항
    - **equals**를 재정의할 땐 **hashCode**도 반드시 재정의하자
    - 너무 복잡하게 해결하려 들지 말자
      → 필드들의 동치성만 검사해도 규약을 어렵지 않게 지킬 수 있다.
    - Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자

- AutoValue 프레임워크
    - 애너테이션 하나만 추가하면 equals 테스트 코드를 알아서 작성해준다.