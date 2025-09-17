# Item10

<aside>

equals는 일반 규약을 지켜 재정의하라

</aside>

equals 메서드는 재정의하기 쉬워 보이지만 함정이 많다. 잘못 구현하면 프로그램이 이상하게 동작하거나 디버깅하기 어려운 버그를 만들어낸다. 따라서 언제 재정의해야 하고 어떻게 해야 하는지 정확히 알아야 한다.

### 1. When?

→ 값을 표현하는 클래스(Integer, String…)일 때

아래의 학생 클래스에서, 물리적으로 다른 두 객체 s1과 s2가 논리적으로는 같다.

```java
public class Student {
    private String name;
    private int studentId;
    
    // 다른 메모리에 있는 두 객체지만 논리적으로는 같을 수 있음
    Student s1 = new Student("김철수", 12345);
    Student s2 = new Student("김철수", 12345);
    // s1 == s2 는 false -> 다른 객체
**//** 이름과 학번이 같으면 같은 학생 ->  **s1.equals(s2)는 true여야 한다.**
}
```

이런 클래스들은 Map의 키나 Set의 원소로 사용될 때 논리적 동치성으로 비교되어야 한다. equals를 재정의하지 않으면 같은 학생 정보인데도 다른 객체로 취급되어 중복이 허용되거나 찾기에 실패한다.

그럼 *재정의를 하지 말아야 하는* 경우는…?

- 각 인스턴스가 고유한 경우
    - Thread처럼 동작하는 개체를 표현하는 클래스. 각 스레드는 고유하니까 물리적 동일성만으로 충분하다.
- 논리적 동치성이 의미 없는 경우
    - Random 클래스 같은 경우, 두 난수 생성기가 ‘같다’는 개념 자체가 의미 없음.
- 상위 클래스가 이미 적절히 구현한 경우
    - ArrayList는 AbstractList의 equals를 그대로 쓰면 된다.
- equals 메서드를 호출할 일이 없는 경우
    - private 클래스나 package-private 클래스
    - package-private → 어떠한 접근 지정자도 명시하지 않는 것. 같은 package 내에서만 접근 가능함을 의미.

### 2. How?

> **‘equals 규약’**
>
>
> → equals는 동치관계를 구현해야 함! 이 규약을 어기면 컬렉션 클래스들이 예상과 다르게 동작한다.
>
> 1. 반사성： null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
> 2. 대칭성： null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
     >
     >     → 대칭성이 깨지면 컬렉션에서 contains, remove 같은 메서드가 일관성 없게 동작한다.
>
> 3. 추이성： null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
> 4. 일관성： null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
> 5. null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

**구현 단계**

1. == 연산자를 사용해 입력 자기 자신의 참조인지 확인

   → 다를 확률이 높고 비교 비용이 낮은 필드부터 비교한다. 첫 번째 조건이 false면 나머지는 평가하지 않으니 순서가 성능에 큰 영향을 준다.

    - 참조 타입: `Objects.equals()` (null 안전)
    - float/double: `Float.compare()`, `Double.compare()` (NaN, -0.0f 등 특수값 처리)
    - 배열: `Arrays.equals()`
2. instanceof 연산자로 입력이 올바른 타입인지 확인
3. 입력을 올바른 타입으로 형변환
    - 2번의 확인
4. 입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사
    - 하나라도 다르면 false
5. 위의 equals 규약을 잘 지켰는지 검토하자

입력 타입이 Object가 아니라면 재정의가 아닌 다중정의라는데… 아이템52에서 다룬다고 하니 일단 넘어감.

<aside>

꼭 필요한 경우가 아니라면 equals를 손수 재정의하지 말자…

**IntelliJ에서의 사용법:**

1. 클래스 코드 안에서 커서를 둡니다.
2. **`Alt + Insert` (Windows/Linux)** 또는 **`Cmd + N` (Mac)** → **Generate 메뉴** 열기.
3. **`equals()` and `hashCode()`** 선택.
4. 비교 기준으로 삼을 필드를 체크하면, IntelliJ가 알아서 `equals()`와 `hashCode()`를 만들어 줍니다.

**주의점:**

- 생성된 코드는 사람이 직접 짠 것보다 일정하고 실수가 적습니다.
- 다만 AutoValue처럼 **필드가 바뀌면 자동 갱신되지는 않아서**, 필드 추가/삭제 후에는 다시 Generate해야 합니다.
</aside>