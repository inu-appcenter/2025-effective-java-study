# Item06

> ***불필요한 객체 생성을 피하라***

<br>

## 재사용은 빠르고 세련되다.

 아래 코드는 호출될 때마다 새로운 **String** 객체를 만들어 할당한다. 한번 호출되고 마는 코드면 문제가 없지만, 이 코드가 반복문 안에 들어가 있다면 같은 String 객체가 몇백 개 만들어질 수도 있다. 

 <br>

```java
String s = new String("bikini");
```

<br>

객체에 대한 참조가 잦다면 또는 완전히 통제할 자신이 없다면 아래 방식을 지향하자. 또한, 이전 아이템인 정적 팩터리 메서드를 활용해도 좋다.

<br>

```java
String s = "bikini"
```

<br>

 아래 **isRomanNumeral** 메소드를 살펴보자. 이 메소드가 하는 일은 매개변수로 주어진 문자열 s를 주어진 정규 표현식과 일치하는지 검사하고 그 결과를 반환한다.

 <br>

```java
static boolean isRomanNumeral(String s) {
    return s.matches("(?=.)M*(C[MD]|D?C{0,3})" + 
                     "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

<br>

 하지만, 자세히 살펴보면 **String.matches** 메서드는 내부적으로 **Pattern** 인스턴스를 매번 생성하고 해당 메소드가 종료되면 가비지 컬렉션의 대상이 되어 자원의 낭비가 발생하게 된다. 

 <br>

 따라서, 아래와 같이 정규표현식을  정적 필드로 캐싱하여 클래스 로딩 시 한번만 생성되게 하면 자원의 낭비를 줄일 수 있다.

 <br>

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
        "(?=.)M*(C[MD]|D?C{0,3})" + 
        "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

<br>

 아래 코드도 불필요한 자원 낭비가 발생하고 있다.

```java
private static long sum() {
    Long sum = 0L; 
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i; 
    return sum;
}
```

<br>

 합계를 나타내는 **sum**은 래퍼 클래스인 **Long**으로 선언되었지만, 더해지는 수 **i**는 원시 타입 **long**으로 선언되었다. 따라서, `sum += i`가 실행될 때마다 **Long** 객체가 생성되고 이 역시 불필요한 자원 낭비이다.

 <br>

## 재사용에 주의해야 하는 경우

 재사용은 보통 성능상 이점을 가져다 줄 수도 있지만, 경우에 따라 의도하지 않은 방향으로 코드가 실행되게 할 수도 있다. 

 <br>

<aside>

**🧑🏻‍💻 어댑터(Adapter)**

---

**어댑터(Adapter)**는 실제 작업은 뒷단 객체에 위임하고 자신은 제 2의 인터페이스 역할을 해주는 객체이다.어댑터는 뒷단 객체 하나만 관리하면 되기 때문에, 객체 하나당 어댑터 하나씩만이 필요하다.

</aside>

<br>

**Map** 인터페이스의 **keySet** 메서드는 **Map** 객체 안의 키를 담은 `Set 뷰`를 반환한다. 즉, 존재하는 키값을 통해 새로운 객체를 반환하는 것이 아닌, 원본의 정보가 담긴 Set을 반환하는 것이다. 

<br>

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);

Set<String> keys1 = map.keySet();
Set<String> keys2 = map.keySet();
```

<br>

따라서, 뷰로 반환된 **keys1**, **keys2**에 대해 수정, 추가, 삭제 연산이 일어나면 실제 **map**에도 영향을 주기 때문에 의도하지 않은 방향으로 동작할 수 있다.
