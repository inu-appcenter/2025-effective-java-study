# Item 10. equals는 일반 규약을 지켜 재정의하라

자바에서 객체를 비교할 때 ==를 사용하면 동일성(객체 참조값을 비교), equals를 사용하면 동등성을 사용한다고 보면 된다. primitive type은 값을 그대로 비교하기에 ==를 사용해도 된다.

```jsx
class User {
    String studentId;

    User(String studentId) {
        this.name = name;
    }
}

public class EqualsStudy {
    public static void main(String[] args) {
        User u1 = new User("202101535");
        User u2 = new User("202101535");

        System.out.println("u1 == u2 ? " + (u1 == u2));          // false
        System.out.println("u1.equals(u2) ? " + u1.equals(u2));  // equals를 재정의해줬다면 true
    }
}

```

equals를 사용해야 하는 경우는 위 코드와 같다. 한 학교에 학번은 고유한 값이라고 할 때, 학번이 같으면 학생도 같을 것이다. User라는 객체 자체를 비교하고 싶다고 ==을 사용해 비교하면, u1과 u2는 참조값이 달라 false를 반환할 것이다. 객체 참조값이 아니라 학번값으로 User를 비교하고 싶다면, ==가 아니라 equals를 학번을 비교하도록 오버라이딩 해야한다.

```jsx
public boolean equals(Object obj) {
 return (this == obj);
 }
```

Object에서 기본적으로 제공하는 equals는 ==와 다를 것이 없으므로, 동등성을 이용해 비교하고 싶은 객체가 있다면 반드시 equals를 재정의해줘야 한다. String, Integer 등 자바에서 제공하는 객체들은 equals를 오버라이딩이 돼 있어 개발자들이 사용할 수 있다.

다음과 같은 경우는 equals를 재정의하지 않는 것이 좋다.

- 각 인스턴스가 본질적으로 고유한 경우
    
    → 값을 표현하는 객체가 아니라, 스레드같이 동작하는 객체
    
- 인스턴스의 논리적 동치성을 검사할 일이 없는 경우

- 상위 클래스에서 재정의한 equals를 하위 클래스에서도 사용할 수 있는 경우

- 클래스가 private거나 default이고, equals를 호출할 일이 없는 경우
    
    → equals를 실수로라도 호출되는 걸 막고싶다면 예외를 던지도록 구현
    

equals 메서드를 재정의할 때는 다음을 만족해야 한다.

- 반사성
    
    → 객체는 자기 자신과 같아야 한다.(x.equals(x)는 항상 true)
    
- 대칭성
    
    → 두 객체가 서로에 대해 동일하다고 판단되면 양방향으로 동일해야 한다.(x.equals(y)가 true면 y.equals(x)도 true)
    
- 추이성
    
    → 객체 1이 객체 2와 동일, 객체 2가 객체 3과 동일하면 객체 1과 객체 3은 동일해야 한다.
    
- 일관성
    
    → 두 객체의 상태가 변경되지 않는 한 equals 메소드는 항상 동일한 값을 반환해야 한다.
    
- null 아님
    
    → null이 아닌 모든 참조값 x에 대해 x.equals(null)은 false다.
    

equals 메소드 구현 방법은 다음과 같다.

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다. (double, float같은 경우에는 compare 정적메소드 이용)
5. equals() 구현 후 hashCode() 구현

```jsx

public class PhoneNumber {
    private final short areaCode, prefix, lineNum;

// intellj getClass expression
    @Override
    public boolean equals(Object o) {
        if (o == null || getClass() != o.getClass()) return false;
        PhoneNumber that = (PhoneNumber) o;
        return areaCode == that.areaCode && prefix == that.prefix && lineNum == that.lineNum;
    }
    
// intellij instanceof expression
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof PhoneNumber that)) return false;
        return areaCode == that.areaCode && prefix == that.prefix && lineNum == that.lineNum;
    }
    
// effective java
@Override
public boolean equals(Object o) {
    if (o == this) return true; 
    if (!(o instanceof PhoneNumber)) return false; 
    PhoneNumber pn = (PhoneNumber) o;
    return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
}

```

that 문법은 캐스팅과 instanceof를 합친 문법이다. intellij instanceof expression과 effective java의 equals는 사실상 같다고 보면 된다.

effective java의 equals에서는 if (o == this)를 최적화하는 용도로 썼지만, intellij에서는 사용하지 않았다.

equals를 instanceof로 구현하면 자식 클래스의 인스턴스도 허용해 유연하게 사용할 수 있고, getClass로 구현하면 완전히 같은 타입의 인스턴스 이외에는 전부 허용하지 않아 견고하게 사용할 수 있다. 다만 instanceof를 사용한다고 해도 자식 클래스에서 새로운 필드가 생긴다면 equals 규약을 만족시킬 방법은 없다. 이를 우회하는 방법으로 상속 대신 컴포지션을 사용하는 방법이 있다.