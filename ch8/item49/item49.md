# item 49

## 🩺 매개변수가 유효한지 검사하라

---

메서드와 생성자의 입력 매개변수에 대한 제약 조건은 반드시 문서화 해야하며, 메서드 실행 전 검사해라.

🚫 메서드 몸체가 실행된 이후에는 많은 문제가 발생할 수 있다.
→ 실행되기 전, 잘못된 값에 대해서는 `예외 처리`가 필수이다.

만약, 검사에 실패한다면 **실패 원자성**을 어기게 될 수 있다.

public과 protected 메서드는 잘못된 매개변수에 대한 예외를 문서화 해야한다.
→ @throws 자바독 태그를 사용해라

이때, 매개변수의 제약을 문서화 한다면, 제약을 어길 시 발생하는 예외도 함께 기술할 필요가 있다.

ex)

```java
/**
* （현재 값 mod m） 값을 반환한다. 이 메서드는
* 항상 음이 아닌 Biginteger를 반환하다는 점에서 remainder 메서드와 다르다.
*
* @param m 계수(양수여야 한다.)
* @return 현재 값 mod m
* @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
*/
public Biginteger mod(Biginteger m) {
		if (m.signum() <= 0)
				throw new ArithmeticException("계수(m)는 양수여of 합니다. " + m);
		... // 계산 수행
}
```

위와 같이 @throws를 통해 명시하면 된다.

위의 메서드를 한 번 보자.
이 메서드는 m이 null일 때, NPE를 던지지만,  null에 대한 얘기는 어디에도 없다.

🤔 잘못된 것일까?
이건 의도적인 설계이다.

BigInteger의 클래스를 가보면, 클래스-level 주석에 아래와 같은 규칙이 존재한다.

```java
 // BigInteger
 * <p>All methods and constructors in this class throw
 * {@code NullPointerException} when passed
 * a null object reference for any input parameter.
```

> 이 클래스의 모든 public 메서드는 null 인수를 허용하지 않으며,
null이 전달되면 NullPointerException을 던진다.
>

즉, 클래스 전체의 규칙으로 훨씬 더 일관성 있고 깔끔한 주석이 되었다.

+ Nullable 등 애너테이션을 쓸 수도 있지만, 자바 표준에는 없으므로
  공식 문서에는 자바독을 명시하는 것이 선호된다.

✨ 자바 7에 추가된 **java.util.Objects.requireNonNull** 메서드는 유연하고 편하다.
→ null 검사를 수동으로 하지 않아도 된다.
→ 원하는 예외 메시지도 지정이 가능하다.
→ 입력 값을 그대로 반환하여, 값 사용과 null 체크가 동시에 가능하다.

```java
코드 49-1 자바의 null 검사 기능 사용하기
this.strategy = Objects.requireNonNul(strategy, "전략");
```

⭐️ 자바 9에는 Objects에 범위 검사 기능도 더해졌다.
→ checkFromlndexSize, checkFromToIndex, checkindex라는 메서드들이다.
→ 예외 메시지를 다루지 못하고, 리스트와 배열 전용이고, 양 끝단 값은 다루지 못하는 제약들이 존재한다.
→ 제약만 만족한다면 아주 유용하다.

```java
1. Objects.checkIndex(int index, int length)
-> index가 [0, length) 범위 안에 있는지 확인
-> 유효하면 그대로 index 반환
-> 유효하지 않으면 IndexOutOfBoundsException 발생

2. Objects.checkFromToIndex(int fromIndex, int toIndex, int length)
-> [fromIndex, toIndex) 구간이 [0, length) 범위 안에 있는지 확인
-> 유효하면 그대로 fromIndex 반환
-> 잘못된 구간이면 IndexOutOfBoundsException

3. Objects.checkFromIndexSize(int fromIndex, int size, int length)
-> fromIndex부터 size만큼의 범위가 전체 길이 안에 있는지 확인
-> 유효하면 fromIndex 반환
-> 잘못된 경우 IndexOutOfBoundsException
```

🔎 public 메서드가 아니라면 단언문(`assert`)을 사용해 매개변수 유효성을 검증할 수 있다.

```java
코드 49-2 재귀 정렬용 private 도우미 함수
private static void sort(long a[], int offset, int length) {
		assert a != null;
		assert offset >= 0 && offset <= a.length;
		assert length >= 0 && length <= a.length - offset;
		... // 계산 수행
}
```

단언문을 사용시,
1. 실패하면 AssertionError를 던진다.
2. 런타임에서 아무런 효과도, 성능 저하도 없다.

‼️ 메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 더 신경 써서 검사해야 한다

💈 생성자는 “나중에 쓰려고 저장하는 매개변수의 유효성을 검사하라”는 원칙의 특수한 사례다.
생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는 데 꼭 필요하다.

매개변수 유효성 검사에도 예외는 있다.
→ 바로, 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때,
계산 과정에서 암묵적으로 검사가 수행될 때다.

ex) 객체 리스트를 정렬할 때,
비교를 하던 중 잘못된 타입일 경우 예외가 발생한다.
즉, 굳이 비교하기 전, 검사할 필요가 없다.
🙄 BUT, 또 암묵적 유효성 검사에 너무 의존하면,  실패 원자성이 깨진다.

때로는, API 문서에서 던지기로 한 예외와 실제 계산 중 발생된 예외가 다를 수도 있다.
→ 이때는, 예외 번역 관용구 (아이템 73)을 사용해 API 문서에 기재된 예외로 번역해야 한다.

📁 정리

❌ 이 글의 중점은 “매개변수에 제약을 둬라”가 아니다.

✅ ”어떤 매개변수든 처리할 수 있도록 범용적으로 설계해라”이다.