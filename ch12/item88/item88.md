# item 88

## 🛡️ readObject 메서드는 방어적으로 작성하라

---

### **⚠️ 불변 클래스를 직렬화하면 생기는 문제**

```java
코드 88-1 방어적 복사를 사용하는 불변 클래스
public final class Period {
		private final Date start;
		private final Date end;
		/**
		* @param start 시작 시각
		* @param end 종료 시각; 시작 시각보다 뒤여야 한다.
		* @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
		* @throws NullPointerException start나 end가 null이면 발행한다.
		*/
		public Period(Date start, Date end) {
				this.start = new Date(start.getTime());
				this.end = new Date(end.getTime());
				if (this.start.compareTo(this.end) > 0)
						throw new IllegalArgumentExceptior"
								start + "가 " + end + "보다 늦다.");
		}
		public Date start() { return new Date(start.getTime()); }
		public Date end() { return new Date(end.getTime()); }
		public String toString() { return start + " - " + end; }
		... // 나머지 코드는 생략
}
```

→ 불변을 유지하기 위해 **생성자와 getter에서 방어적 복사**를 수행

- 이 상태에서 그냥 implements Serializable만 붙여서 직렬화 가능하게 만들면,
    - 겉보기엔 기본 직렬화를 써도 괜찮아 보인다.

🚫 하지만 문제는 **역직렬화(readObject)** 에 있다.

- readObject는 사실상 바이트 스트림을 인자로 받는 생성자 역할이다.
- 따라서,
    - 인수(바이트 스트림이 표현하는 필드 값)가 유효한지 검사해야 하고
    - 필요한 경우 방어적 복사도 해야 한다.
- 이걸 안 하면, 공격자가 “정상 생성자로는 만들 수 없는 객체 상태”를 만든 바이트 스트림을 직접 던져서

  클래스의 `불변식`을 깨버릴 수 있다.


### 1️⃣ 문제: 불변식(시작 ≤ 종료) 깨기

기본 직렬화만 사용하는 경우,

- Period에 그냥 implements Serializable만 추가하고 아무것도 안 했다고 가정
- 다음이 가능해진다.
    - 정상적인 Period 인스턴스를 직렬화해서 나온 바이트 배열을
      사람이 직접 편집해서, start > end 상태가 되도록 조작

ex)

- serializedForm이라는 바이트 배열 리터럴 안에 `조작된 직렬화 데이터`가 들어 있음
- deserialize(serializedForm)를 호출해서 객체를 만들면
- 시작 시각보다 종료 시각이 더 앞선 이상한 Period가 만들어지게 된다.

즉,

> **일반 생성자**에선 유효성 검사 때문에 절대로 만들 수 없는 상태인데,
`직렬화/역직렬화`를 이용하면 그런 객체를 만들 수 있다.
>

→ 기본 직렬화를 그대로 쓰면 readObject가 아무 방어도 안 하는 상황으로 불변식이 깨진 인스턴스를 만들 수 있다.

✨ 어떻게 개선하는가?

→ readObject에서 불변식 검사

```java
// 코드 88-3 유효성 검사를 수행하는 readobject 메서드 - 아직 부족하다!
private void readObject(ObjectInputStream s)
        throws IOException, ClassNotFoundException {
    s.defaultReadObject(); // 기본 역직렬화로 필드 채우기

    // 불변식 검사
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```

readObject에서 직렬화된 값에 대해 불변식 검사를 해줘야 기본적인 불변식 위반 공격을 막을 수 있다.

BUT, 불변식(기간의 순서)은 지켰지만, 불변 자체가 깨질 여지가 있다.

### **2️⃣ 문제: 가변 공격**

→ Period가 더 이상 `불변`이 아니게 만드는 공격이다.

- 정상적인 Period를 하나 직렬화한 후
    - 그 **바이트 스트림 뒤에** `이전 객체 참조`를 추가해서
      내부 start, end 필드로 가는 참조를 추가로 붙일 수 있다.

그런 스트림을 역직렬화할 때,

1. Period 인스턴스가 하나 만들어짐
2. 이어서 읽는 두 개의 Date 객체는 사실 Period 내부 필드 start, end에 대한 **참조**가 됨

❌ 공격자는 이 참조를 이용해서 Date를 마음껏 변경할 수 있고,
그 결과, Period 내부 상태도 바뀐다.

ex)

1. new Period(new Date(), new Date())를 직렬화해 ByteArrayOutputStream에 씀
2. 그 뒤에 **악의적인 이전 객체 참조 바이트**를 직접 추가
3. 이 스트림을 ObjectInputStream으로 읽어들일 때,
    - 먼저 Period p를 역직렬화
    - 그 다음 Date start, Date end 두 개를 더 읽는데,
      이 둘이 p의 내부 start, end와 같은 객체를 가리키게 됨
4. mp.end.setYear(78); 처럼 end를 수정하면
    - p의 내부 종료 시각도 같이 바뀌어버림 → Period는 더 이상 불변이 아님

즉,

> 역직렬화 시에 `내부 가변 필드로의 참조`가 외부로 새어나올 수 있다면,
불변 클래스도 가변처럼 변경 가능한 상태가 되어 보안 문제가 생긴다.
>

✨ 최종 개선

→ readObject에서 방어적 복사 + 불변식 검사

1. 방어적 복사
    - 외부에서 소유하면 안 되는 객체(특히 private 가변 필드)를 새로 복사해서 저장
2. 불변식 검사
    - 복사된 값들을 이용해 불변식을 확인

```java
// 코드 88-5 방어적 복사와 유효성 검사를 수행하는 readObject 메서드
private void readObject(ObjectInputStream s)
        throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 1. 가변 요소들을 방어적으로 복사
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 2. 불변식 검사
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```

‼ 주의

1. 방어적 복사를 불변식 검사보다 먼저 수행

   → 불변식 검사를 하기 전에 필드가 공격자가 조작한 객체를 직접 가리키고 있을 수 있음

2. Date의 clone()을 쓰지 않은 점

   → clone()은 신뢰하기 어려운 경우도 있고, 일반적으로 “복사 생성자 / 팩터리 기반 복사”를 권장하기 때문

3. start, end에서 final 제거 필요

   → readObject 안에서 다시 새 Date로 덮어써야 하기 때문

   → 아쉽긴 하지만, final을 유지하다가 불변이 깨지는 것보다는
   final을 포기하고 진짜 불변을 지키는 편이 낫다.


→ 이제 MutablePeriod 공격도 더 이상 통하지 않는다.

### **💬 기본 readObject를 써도 되는지 판단하는 요령**

> 🤔 transient가 아닌 모든 필드를 인수로 받는 public 생성자를 하나 추가해서,
>
>
> 아무 유효성 검사 없이 그대로 필드에 대입해도 괜찮은가?
>

→ 만약 답이 “아니오”라면,

- 기본 직렬화만 믿으면 안 되고 `커스텀 readObject`를 작성해야 한다.
    - 여기서 생성자에서 했어야 할 모든 유효성 검사와 방어적 복사를 수행
    - 혹은 `직렬화 프록시 패턴`(아이템 90)을 고려하는 것이 좋다.
        - 프록시를 통해서만 진짜 객체를 만들도록 하는 방식이라 더 안전해짐

### **☝🏻 readObject, 생성자의 또 다른 공통 규칙**

→ readObject도 생성자와 마찬가지로 `재정의 가능한 메서드`(override 가능한 메서드)를 호출하면 안 된다.

- 이유:
    - 상위 클래스에서 readObject(또는 생성자)가 재정의 가능한 메서드를 호출하고,
    - 하위 클래스가 그 메서드를 재정의했다면,
    - 하위 클래스의 상태가 *완전히 초기화/역직렬화되기 전에* 그 메서드가 실행될 수 있다.
- 그 결과:
    - 초기화 안 된 상태의 하위 클래스 필드가 사용되거나
    - 예기치 않은 동작, 심각한 버그로 이어질 수 있다.

### 📁 정리

> 안전한 readObject 작성 원칙
>
1. *private이어야 하는 객체 참조(특히 가변 객체) 필드는 방어적 복사*
2. *불변식 검사 필수*
3. *객체 그래프 전체에 대한 추가 검사가 필요하다면 ObjectlnputValidation 인터페이스를 사용*
4. *재정의 가능한 메서드는 호출 금지 (직접/간접적으로 호출하지 말 것)*