# Item 10

추가 일시: 2025년 9월 13일 오전 2:59
강의: Effective JAVA

## 🍀 Item 10: equals는 일반 규약을 지켜 재정의하라

---

equals 메서드를 재정의하는 것은 상당한 리스크가 있다.

책에서는 다음과 같은 상황에서는 재정의 하지 않는 것을 권장하고있다.

**equals를 재정의 하지 않는 것을 권장하는 상황**

- 각 인스턴스가 본질적으로 고유한 상황
- 인스턴스의 ‘논리적 동치성’을 검사할 일이 없는 상황
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 들어맞는 상황
    - 구현체로부터 상속받은 equals를 그대로 사용하자
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없는 상황
    - 실수로 호출되는 상황을 막고 싶다면 오버라이드하여 AssertionError()를 반환

**그렇다면 언제 equals를 재정의 하나요?**

객체 식별성이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때

```java
public class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        Person person = (Person) obj;
        return age == person.age && 
               Objects.equals(name, person.name);
    }
}

// 사용 예시
Person p1 = new Person("김철수", 25);
Person p2 = new Person("김철수", 25);

System.out.println(p1.equals(p2)); // true! (논리적으로 같은 사람)
System.out.println(p1 == p2);      // false (여전히 다른 객체)
```

### equals 메서드를 재정의하기 위한 일반 규약

---

equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.

**반사성 (reflexivity)**

null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.

객체는 자기 자신과 같아야 한다.

해당 요건은 일부러 어기는 경우가 아니라면 만족시키지 못하기가 더 어려워보인다.

**대칭성 (symmetry)**

null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x) 도 true다.

두 객체는 서로에 대한 동치 여부에 똑같이 응답해야한다.

x.equals(y) 가 true를 반환한다면,  y.equals(x) 도 반드시 true를 반환해야한다.

일방향으로만 equals가 동작해서는 안 된다.

**추이성 (transitivity)**

null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.

아리스토텔레스의 3단 논법과 정확히 동일한 구조이다.

<aside>
☘️

대전제: A = B
소전제: B = C
결론: A = C

</aside>

<aside>
☘️

전제1: x.equals(y) == true
전제2: y.equals(z) == true
결론: x.equals(z) == true 

</aside>

구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.

**일관성 (consistency)**

null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

두 객체가 같다면 앞으로도 영원히 같아야 한다.

가변 객체는 비교 시점에 따라 서로 다를 수도 있지만, 불변 객체는 한번 다르면 끝까지 달라야 한다.

*클래스가 불변이든 가변이든 equlas의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.*

**Not-Null**

null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

모든 객체가 null과 같지 않아야 한다.

여기서 주의할 점은 null입력에 대해 NullPointerException을 던지지 말아야한다.

equals는 비교 결과를 알려주는 메서드이지 검증하는 메서드가 아니므로 안정성을 위해 false를 반환해야한다.

```java
@Override
public boolean equals(Object obj) {
    //null 체크 + 타입 체크 한 번에
    if (!(obj instanceof Person)) return false;
    
    Person other = (Person) obj;
    return Objects.equals(this.name, other.name);
}
```

instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사하는 방식이 권장된다.

### equals 메서드의 단계적 구현 방법

---

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신이 애응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사한다.

구현 후 세 가지를 자문해보자. 대칭적인가? 추이성이 있는가? 일관적인가?

### + 추가적인 고려사항

---

float와 double을 제외한 기본 타입 필드는 == 연산자로 비교하고, 참조 타입 필드는 equals 메서드로 비교한다.

float와 double 필드는 각각 정적 메서드인 .compare로 비교한다.

종종 null도 정상 값으로 취급하는 참조 타입 필드도 있다.

→ 다음과 같은 상황에 NullPointerException이 발생하지 않도록 Objects.equals로 비교하자.

최상의 성능을 위해 비용이 싼 객체부터 비교하자.

equals를 작성하고 테스트하는 일을 대신해주는 오픈소스를 활용하자.

→ google의 AutoValue 프레임 워크, 테스트코드는 IDE에 맡기는 것을 추천한다.