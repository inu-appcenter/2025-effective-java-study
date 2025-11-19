# item38

## ☝🏻 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

---

### 🔎 열거 타입의 확장

`열거 타입`은 거의 모든 상황에서 `타입 안전 열거 패턴`보다 우수하다.

BUT, 타입 안전 열거 패턴은 확장할 수 있는 반면, 열거 타입은 불가하다.
→ 열거한 값들을 가져온 후, 값을 더 추가 가능

대부분 열거 타입을 확장하는건 좋지 않다.
→
1. 확장한 타입의 원소는 기반 타입의 원소로 취급하지만, 그 반대는 성립하지 않는다
2. 기반 타입과 확장된 타입의 모든 원소를 순회할 방법이 없다
3. 확장성을 높이려면 고려할 요소가 늘어나 설계 복잡도가 올라간다

이게 무슨 말이냐 하면,

```java
// 1. 확장한 타입의 원소는 기반 타입의 원소로 취급하지만, 그 반대는 성립하지 않는다

// ExtendedFruit.ORANGE는 Fruit 으로 보면 “과일 중 하나”
// 하지만 Fruit.APPLE 은 ExtendedFruit 로는 보이지 않음
enum Fruit { APPLE, BANANA }
enum ExtendedFruit extends Fruit { ORANGE }
```

```java
// 2. 기반 타입과 확장된 타입의 모든 원소를 순회할 방법이 없다
Fruit.values() → 확장 타입의 원소(ORANGE)는 안 나옴
ExtendedFruit.values() → 기반 타입 원소(APPLE, BANANA)는 안 나옴
```

### 🧮 연산 코드

확장할 수 있는 열거 타입이 어울리는 쓰임이 최소한 하나는 있다.

→ 연산 코드(operation code 혹은 opcode)
→ 연산 코드의 각 원소는 특정 기계가 수행하는 연산을 의미

API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가할 수 있도록 열어줘야 할 때 필요하다.

✨ 열거 타입으로 이 효과를 내는 멋진 방법이 있다

→ 열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용하는 것

→ 연산 코드용 인터페이스를 정의하고, 열거 타입으로 구현
( 열거 타입이 그 인터페이스의 표준 구현체 역할)

```java
코드 38-1 인터페이스를 이용해 확장 가능 열거 타입을 흉내 냈다.
public interface Operation {
		double apply(double x, double y);
}

public enum BasicOperation implements Operation {
		PLUS("+") {
				public double apply(double x, double y) { return x + y; }
		},
		MINUS("-"){
				public double apply(double x, double y) { return x - y; }
		},
		TIMES("*") {
				public double apply(double x, double y) { return x * y; }
		},
		DIVIDE("/") {
				public double apply(double x, double y) { return x / y; }
		}；
		
		private final String symbol;
		
		BasicOperation(String symbol) {
				this.symbol = symbol;
		}
		
		@Override public String toString() {
				return symbol;
		}
}
```

✅ 열거 타입인 BasicOperation은 확장할 수 없지만, 인터페이스인 Operation은 확장할 수 있다.
→ 인터페이스를 연산의 타입으로 사용하면 된다.
→ 또 다른 열거 타입 구현체를 정의해 대체도 가능하다.

```java
코드 38-2 확장 가능 열거 타입
public enum ExtendedOperation implements Operation {
		EXP("^") {
				public double apply(double x, double y) {
						return Math.pow(x, y);
				}
		},
		REMAINDER("%") {
				public double apply(double x, double y) {
						return x % y;
				}
		}；

		private final String symbol;

		ExtendedOperation(String symbol) {
				this.symbol = symbol;
		}
		@Override public String toString() {
				return symbol;
		}
}
```

→ 이는 기존 연산을 쓰던 곳 어디던 사용 가능
→ 열거 타입에 따로 추상 메서드가 없어도 된다.

### ☄️ 확장된 열거 타입을 넘기는 방법

기본 열거 타입 대신, 확장된 열거 타입을 넘겨 확장된 열거 타입의 모든 원소를 사용하게 할 수도 있다.

```java
public static void main(String[] args) {
		double x = Double.parseDouble(args[0]);
		double y = Double.parseDouble(args[1]);
		test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(
				Class<T> opEnumType, double x, double y) {
		for (Operation op : opEnumType.getEnumConstants())
				System.out.printf("%f %s %f = %f%n",
													x, op, y, op.apply(x, y));
}
```

ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다.
→ 이때, class 리터럴은 `한정적 타입 토큰` 역할을 한다.

opEnumType 매개변수의 선언(<T extends Enum<T> & Operation> Class<T>)
→ Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻
→ 그래야 순회가 가능하며, 뜻하는 연산 수행이 가능하기 때문

두 번째 대안은
→ Class 객체 대신 `한정적 와일드카드 타입`인 Collection<? extends Operation>을 넘기는 방법이다.

```java
public static void main(String[] args) {
		double x = Double.parseDouble(args[0]);
		double y = Double.parseDouble(args[1]);
		test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Coflection<? extends Operation> opSet,
				double x, double y) {
		for (Operation op : opSet)
				System.out.printf("%f %s %f = %f%n",
						x, op, y, op.apply(xf y));
}
```

→ test 메서드가 더 유연해졌다.
→ 여러 구현 타입의 연산을 조합해 호출할 수 있게 되었다.

→ ❌ BUT, 특정 연산에서는 EnumSet과 EnumMap을 사용하지 못한다.

### ⚠️ 문제점

인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에도 한 가지 사소한 문제가 있다.

→ 열거 타입끼리 구현을 상속할 수 없다는 점이다.

아무 상태에도 의존하지 않는 경우에는 디폴트 구현(아이템 20)을 이용해 인터페이스에 추가하는 방법이 있다.

→ BUT, 연산 기호를 저장하고 찾는 로직이 BasicOperation과 ExtendedOperation 모두에 들어가야 한다.

→ 즉, 도우미 클래스나 정적 도우미 메서드로 코드 중복을 줄여야 한다.

### 📁 정리

자바 라이브러리도 이번 아이템에서 소개한 패턴을 사용한다.
ex) java.nio.file.LinkOption 열거 타입은 CopyOption과 OpenOption 인터페이스를 구현했다.

열거 타입 자체는 확장할 수 없지만,
✅ 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다.