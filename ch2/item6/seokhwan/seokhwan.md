# Item 6

추가 일시: 2025년 9월 5일 오후 5:27
강의: Effective JAVA

## 🍀 Item 6: 불필요한 객체 생성을 피하라

---

동일한 기능의 객체를 매번 생성하는 것은 쓸데없는 인스턴스를 만들어 낼 수 있다.

```java
//잘못된 예
String s = new String("new instance");

//올바른 예
String s = "good";
```

빈번히 호출되는 메서드에 잘못된 예외 같은 코드가 있다면 쓸데없는 인스턴스가 만들어지게 된다.

매번 새로운 인스턴스를 만드는 대신 하나의 인스턴스를 사용하여, 같은 가상 머신내에서 이와 같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함을 보장할 수 있다.

**정적 팩터리 메서드 사용**

생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스에는 정적 팩터리 메서드를 사용하는 것도 좋은 방법이다. 

Boolean.valueOf(String) ← 새로운 인스턴스 생성 X

가변 객체의 경우에도 사용 중 변경되지 않는다면 재사용할 수 있다.

**캐싱**

생성 비용이 비싼 객체는 캐싱을 사용하여 성능을 향상 시킬수 있다.

```java
//정규식을 파싱하고 컴파일하는 비용이 큰 작업
public static boolean isValidEmailBad(String s) {
        return Pattern.compile("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$")
                     .matcher(s)
                     .matches();
    }

//다음과 같이 처리
private static final Pattern EMAIL_PATTERN = Pattern.compile("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$");

public static boolean isValidEmailGood(String s) {
    return EMAIL_PATTERN.matcher(s).matches();
}
```

다음과 같이 불변인 Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱해두고 이후 isValidEmailGood 메서드가 호출될 때마다 인스턴스를 재사용 할 수 있다.

이 경우 메서드가 반복적으로 호출되는 상황에서 성능 개선을 기대할 수 있다.

⚠️ 개선된 방식의 클래스가 초기화 된 후 메서드를 한 번도 호출하지 않는다면 쓸데없이 초기화된 것이다.

**오토박싱으로 인한 성능저하**

```java
private static long sum() {
	Long sum = 0L;
	for (long i = 0; i <= Integer.MAX_VALUE; i++)
		sum += i;
	
	return sum;
}
```

오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 똑같은 것은 아니다.

sum 변수를 Long으로 선언한 덕분에 sum에 i를 더해줄 때마다 불필요한 Long 인스턴스를 생성한다.

→ 박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 생기지 않도록 주의하자

책에서는 마지막으로 다음과 같이 설명한다.

> 생성비용이 아주 비싼 객체가 아니라면 객체 생성을 피하고자 객체 풀을 만들지 말아라
> 

필요 없는 객체를 반복 생성했을 때의 피해보다 < 방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해가 훨씬 크다.

‘객체 생성은 비싸지 피해야 한다‘로 오해하지 말고 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 그냥 객체 생성을 하라는 것이다.