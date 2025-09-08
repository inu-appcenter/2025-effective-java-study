# 아이템 6

## 불필요한 객체 생성을 피하라

**String 객체 생성 문제**

```java
String s = new String("bikini"); // 매번 인스턴스 생성
String s = "bikini"; // 같은 문자열인 기존 인스턴스를 리턴
```

**정적 팩토리 메서드**

```java
Boolean b = new Boolean("true");
Boolean b = Boolean.valueOf("true");
```

**캐싱**

```java
static boolean isRomanNume rat(St ring s) {
 return s.matches("^?=.)M*(C[MD] |D?C{0,3})"
 + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$")； 
}

String.matches() 내부 동작:

매번 호출될 때마다 Pattern.compile() 실행
정규표현식을 유한 상태 머신으로 컴파일
컴파일된 Pattern으로 매칭 수행
사용 후 Pattern 객체 버림 (가비지 컬렉션 대상)

---

public class RomanNumerals {
	 private static final Pattern ROMAN = Pattern.compile(
	 "^(?=. )M*(C[MD]|D?C{0,3})"
		 + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
	static boolean isRomanNume ral(St ring s) {
	 return ROMAN.matcher(s).matches()；
	} 
}

클래스 로딩 시 한 번만 Pattern 컴파일
static final로 캐싱하여 재사용
매번 호출 시 컴파일된 Pattern만 사용
```

유한상태 머신이란 정규식을 통해 문자열을 비교하는 것을 하나의 설계로 만들어서 비교를 실행하는 것을 말함

**오토박싱**

```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;  // 매번 오토박싱 발생
    }
    return sum;
}

순서
sum을 long으로 언박싱
i, sum을 덧셈
덧셈 결과를 Long으로 오토박싱
그 객체를 sum에 할당

---

private static long sum() {
    long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}
```