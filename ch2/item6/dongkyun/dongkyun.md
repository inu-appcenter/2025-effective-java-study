# item6

## 🏃🏻 불필요한 객체 생성을 피하라

---

- 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많습니다.

```java
// 실행 될 때마다 String 인스턴스 생성
String s = new StringC'bikini");

// 하나의 인스턴스를 사용한다.
String s = "bikini";
```

- 이 방식을 사용한다면 같은 가상 머신 안에서 이와 똑같은 문자열 리터 럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장됩니다.

- 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 전혀 그렇지 않습니다.
    - 즉, 생성자 대신 팩터리 메서드를 사용하는 것이 좋습니다.
- 불변 객체만이 아니라 가변 객체라 해도 사용 중에 변경되지 않을 것임을 안다면 재사용할 수 있습니다.
- 생성 비용이 아주 비싼 객체는 캐싱하여 재사용해야 합니다.

```java
static boolean isRomanNume rat(St ring s) {
		return s.matches("시?=.)M * (C[MD] |D?C{0,3})"
				+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$")；
}
```

- **String.matches**는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만,
  성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않습니다.
    - 내부에서 만드는 Pattern 인스턴스는 한 번 쓰고 버려져 가비지 컬렉션의 대상이 됩니다.
    - Pattern은 입력받은 정규 표현식에 해당하는 유한 상태 머신을 만들어 인스턴스 생성 비용이 높습니다.
    - 이를 성능하기 위해선, Pattern 클래스를 초기화 시 캐싱해두고 재사용해야합니다.

    ```java
    public class RomanNumerals {
    			private static final Pattern ROMAN = Pattern.compile(
    				"(?=. )M* (C [MD] |D?C{0,3})"
    				+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$")；
    		static boolean isRomanNumeral(String s) {
    				return ROMAN.matcher(s).matches()；
    		}
    }
    ```


- 오토 박싱 (auto boxing)
    - 기본 타입과 박싱된 기본 타입을 섞어서 쓸 때 자동으로 상호 변환해주는 기술

    ```java
    private static long sum() {
    		Long sum = 0L;
    		for (long i = 0; i <= Integer.MAX_VALUE; i++)
    				sum += i;
    		return sum;
    }
    ```

    - sum 변수를 long이 아닌 Long으로 선언하여 불필요한 Long 인스턴스가 2^31개나 만들어집니다.
    - 박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의해야합니다.

- 요즘의 JVM에서는 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일이 크게 부담되지 않습니다.
  프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일입니다.