# Item 11

추가 일시: 2025년 9월 13일 오전 3:00
강의: Effective JAVA

## 🍀 Item 11: equals를 재정의하려거든 hashCode도 재정의하라

---

**equals를 재정의한 클래스 모두에서 hashCode도 재정의해야한다.**

Object 명세의 규약에는 다음같은 내용이 있다.

*equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.*

논리적으로 같은 객체는 같은 해시코드를 반환해야한다.

하지만 Object의 기본 hashCode 메서드는 규약과 달리 서로 다른 값을 반환한다.

때문에 equals를 재정의한다면 hashCode도 재정의 해주어야한다.

### hashCode를 재정의 하는 방법

---

1. int 변수 result를 선언한 후 값 c로 초기화한다.
    1. 이때 c는 해당 객체의 첫번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드이다.
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    1. 해당 필드의 해시코드 c를 계산한다.
        1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스이다.
        2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 호출한다. 계산이 더 복잡해질 것 같다면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다.
        3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계싼한 다음, 단계 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0 추천)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
    2. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다.
        
        result = 31 * result + c;
        
3. result를 반환한다.

글로 설명하기에는 다소 난해한 감이 있다.

하지만 기본적인 공식이 정해져있다.

```java
@Override
public int hashCode() {
    int result = 첫번째필드해시코드;
    result = 31 * result + 두번째필드해시코드;
    result = 31 * result + 세번째필드해시코드;
    //...반복
    return result;
}
```

**필드 타입별 처리법**

기본 타입

```java
// int, long, double 등
int age = 25;
int hash = Integer.hashCode(age);   // Integer.hashCode(25)

boolean married = true;
int hash = Boolean.hashCode(married);  // Boolean.hashCode(true)
```

참조 타입

```java
// String, 커스텀 객체 등
String name = "김철수";
int hash = Objects.hashCode(name);  // null 안전!

// null인 경우 자동으로 0 반환
String nullName = null;
int hash = Objects.hashCode(nullName);  // 0
```

배열

```java
// 배열 전체가 중요한 경우
int[] scores = {90, 85, 88};
int hash = Arrays.hashCode(scores);

// 배열의 일부만 중요한 경우
// → 중요한 원소들을 개별 필드처럼 처리
```

더 간단한 방법

```java
@Override
public int hashCode() {
    return Objects.hash(name, age, married);
}
```

해당 방법은 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱을 거쳐야 하기 때문에 성능에 민감하지 않은 상황에서만 사용해야한다.

❓ hash 코드에 31이 쓰이는 이유 → 31이 홀수이면서 소수이기 때문에 충돌을 줄여준다. (관례)

성능 향상을 위해 해시코드를 계산할 때 핵심 필드를 생략하는 것은 좋지 않은 선택이다.

이는 해시 품질을 저하시키고, 해시 테이블의 성능을 떨어뜨릴 수 있다.

해시 계산 비용보다 해시 충돌로 발생할 수 있는 비용이 훨씬 크다.

```java
// 나쁜 예시: 성능을 위해 핵심 필드 생략
public class Person {
    private String firstName;   // 핵심 필드
    private String lastName;    // 핵심 필드  
    private int age;           // 핵심 필드
    private String address;    // 복잡한 필드
    
    @Override
    public int hashCode() {
        // 성능을 위해 address만 사용 (잘못된 선택!)
        return Objects.hash(address);
    }
}

Person p1 = new Person("김", "철수", 25, "서울시 강남구");
Person p2 = new Person("이", "영희", 30, "서울시 강남구");
Person p3 = new Person("박", "민수", 35, "서울시 강남구");

// 모두 같은 해시코드! (같은 주소)
System.out.println(p1.hashCode()); // 12345
System.out.println(p2.hashCode()); // 12345  
System.out.println(p3.hashCode()); // 12345
```

*hashCode가 반환하는 값의 생성 규칙을 API사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.*