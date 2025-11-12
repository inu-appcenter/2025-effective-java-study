# item26

## 🚨 로 타입은 사용하지 말라

---

### 📒 용어

- `제네릭 클래스` or `제네릭 인터페이스`

  → 클래스와 인터페이스 선언에 타입 매개변수가 쓰이는 것


제네릭 클래스와 제네릭 인터페이스를 통틀어 제네릭 타입이라고 한다.

각각의 제네릭 타입은 일련의 매개변수화 타입을 정의한다.

```java
ex) List<String>

먼저 클래스 이름이 나오고 꺾쇠괄호 안에 실제 타입 매개변수들을 나열한다.
위는 원소의 타입이 String인 리스트를 뜻하는 매개변수화 타입이다.
String이 타입 매개변수 E에 해당하는 실제 타입 매개변수이다.
```

제네릭 타입을 하나 정의하면, 그에 딸린 `로 타입(raw type)`도 함께 정의된다.

→ 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.

ex) List<E>의 로 타입은 List

이는, 제네릭이 도래하기 전 코드와 호환되도록 하기 위한 궁여지책

제네릭이 없던 시절

```java
코드 26-1 컬렉션의 로 타입 - 따라 하지 말 것!
// Stamp 인스턴스만 취급한다.
private final Collection stamps = ...;
```

실수로 다른 타입을 넣어도 아무 오류 없이 컴파일 된다.

```java
// 실수로 동전을 넣는다.
stamps, add (new Coin(...)); // "unchecked call" 경고를 내뱉는다.
```

다시 꺼내기 전에는 오류를 알아채지 못한다.

```java
코드 26-2 반복자의 로 타입 - 따라 하지 말 것!
for (Iterator i = stamps.iteratorO; i.hasNext(); ) {
		Stamp stamp = (Stamp) i.next(); // ClassCastException을 던진다.
		stamp.cancel();
}
```

즉, 제네릭을 활용하자

```java
코드 26-3 매개변수화된 컬렉션 타입 - 타입 안전성 확보!
private final Collection<Stamp> stamps = ...;
```

컴파일이 아무 경고 없이 됐다면, 문제가 없음을 보장한다.

```java
Test.java:9: error: incompatible types: Coin cannot be converted
to Stamp
		stamps.add(new Coin());
```

✅ 컴파일러는 컬렉션에서 원소를 꺼내는 모든 곳에 보이지 않는 형변환을 추가하여
절대 실패하지 않음을 보장한다.

### ‼️ 로 타입을 쓰면 안되는 이유

로 타입을 쓰면 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 된다.

🤔 그럼 왜 있냐?

→ 호환성

→ 제네릭과 기존의 코드를 마이그레이션 하며, 기존의 코드들도 호환해야 했기 때문이다.

List 같은 로 타입은 사용해서는 안 되나,
List<Object>처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다.

List는 아예 발을 뺀 것, List<Object>는 모든 타입 허용 의미를 명확히 전달

> **List<Object>** 같은 매개변수화 타입을 사용할 때와 달리
**List** 같은 로 타입을 사용하면 타입 안전성을 잃게 된다
>

```java
코드 26-4 런타임에 실패한다. - unsafeAdd 메서드가 로 타입（List）을 사용
public static void main（String[] args） {
		List<String> strings = new ArrayListo（）;
		unsafeAdd(strings, Integer.valueOf(42));
		String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다.
}

private static void unsafeAdd(List list, Object o) {
		list.add(o);
}
```

이 코드는 컴파일은 되지만 로 타입 인 List를 사용하여 다음과 같은 경고가 발생 한다.

```java
Test.java:10: warning: [unchecked] unchecked call to add(E) as a
member of the raw type List
		list.add(o);
```

❌ 이를 실행하면, strings.get(0)의 결과를 형변환하려 할 때 ClassCastException을 던진다.
→ Integer를 String으로 변환하려 시도했기 때문

> ‼️ 자바의 **제네릭은 컴파일 타임 전용입니다.**

즉, 런타임에는 타입 정보가 “소거(type erasure)”되어 사라집니다.
→ 런타임 시점에는 List<String>이나 List<Integer>나 **그냥 같은 ArrayList 객체

✅ “로 타입”으로 선언하면, 제네릭 타입 정보는 컴파일러가 완전히 무시**

“런타임에는 제네릭 타입 정보가 지워지므로(type erasure), add 시점엔 예외가 안 나고
get 시점에 캐스팅 예외가 발생한다.”
>

만약, 로 타입인 List를 매개변수화 타입인 List<Object>로 바꾸고 실행할 경우?

```java
Test.java:5: error: incompatible types: List<String> cannot be
converted to List<Object>
		unsafeAdd(strings, Integer.valueOf(42));
```

✅ 컴파일 자체를 막는다.

### ⁇ 비한정적 와일드카드 타입

원소의 타입을 몰라도 되는 로 타입을 쓰고 싶을 수도 있다.

```java
코드 26-5 잘못된 예 - 모르는 타입의 원소도 받는 로 타입을 사용했다.
static int numElementslnCommon(Set si, Set s2) {
		int result = 0;
		for (Object ol : si)
				if (s2.contains(ol))
						result++;
		return result;
}
```

이는 비한정적 와일드카드 타입을 사용하라.
→ 실제 타입 매개변수가 뭔지 신경 쓰고 싶지 않다면 물음표(?)를 사용하자.

ex) Set<E> → Set<?>

```java
코드 26-6 비한정적 와일드카드 타입을 사용하라. - 타입와전하며 유연하다.
static int numElementsInComnion（Set<?> si, Set<?> s2） { ... }
```

와일드카드 타입은 안전하고, 로 타입은 안전하지 않다.

→ 로 타입에는 아무 원소나 넣을 수 있고,
→ 와일드카드 타입에는 null 외에는 어떤 원소도 넣을 수 없다.

다른 원소를 넣으려 하면,

```java
Wildcard.java:13: error: incompatible types: String cannot be
converted to CAP#1
		c.add（"verbotenM）;
		
where CAP#1 is a fresh type-variable:
	CAP#1 extends Object from capture of ?
```

🚫 컴파일러는 컬렉션의 타입 불변식을 훼손하지 못하게 막았다.

→ 정확히는
어떤 원소도 Collection<?>에 넣지 못하게 했으며 꺼낼 수 있는 객체의 타입도 전혀 알 수 없게 했다.

→ 이 제약을 받아들일 수 없다면, `제네릭 메서드`(item 30)이나 `한정적 와일드카드 타입`(item 31)을 사용하라

### ⚠️ 예외

로 타입을 쓰지 말라는 규칙에도 소소한 예외가 몇 개 있다.

1. **class** 리터럴에는 로 타입을 써야 한다.

   → 자바 명세는 class 리터 럴에 매개변수화 타입을 사용하지 못하게 했다.

2. instanceof 연산자

   → 런타임에는 제네릭 타입 정보가 지워지므로 `instanceof 연산자`는
   비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다.

   다음은 제네릭 타입에 **instanceof**를 사용하는 올바른 예다.

    ```java
    코드 26-7 로 타입을 써도 좋은 예 - instanceof 연산자
    if (o instanceof Set) { // 로 타입
    		Set<?> s = (Set<?>) o; // 와일드카드 타입
    		...
    }
    ```

   위 처럼, 검사를 통과한 뒤에는 안전하게 **와일드카드 타입으로 형변환**해야 한다.
   → 검사 형변환(checked cast)으로 간주되어 컴파일러 경고가 발생하지 않음.


### 📁 정리

로 타입은 런타임에 예외가 발생할 수 있으니 사용하지 말아야 한다.

Set<Object>와 Set<?>는 안전하지만, 로 타입인 Set은 안전하지 않다.

| **한글 용어** | **영문 용어** | **예시** | **아이템** |
| --- | --- | --- | --- |
| 매개변수화 타입 | parameterized type | List<String> | 아이템 26 |
| 실제 타입 매개변수 | actual type parameter | String | 아이템 26 |
| 제네릭 타입 | generic type | List<E> | 아이템 26, 29 |
| 정규 타입 매개변수 | formal type parameter | E | 아이템 26 |
| 비한정적 와일드카드 타입 | unbounded wildcard type | List<?> | 아이템 26 |
| 로 타입 | raw type | List | 아이템 26 |
| 한정적 타입 매개변수 | bounded type parameter | <E extends Number> | 아이템 29 |
| 재귀적 타입 한정 | recursive type bound | <T extends Comparable<T>> | 아이템 30 |
| 한정적 와일드카드 타입 | bounded wildcard type | List<? extends Number> | 아이템 31 |
| 제네릭 메서드 | generic method | static <E> List<E> asList(E[] a) | 아이템 30 |
| 타입 토큰 | type token | String.class | 아이템 33 |