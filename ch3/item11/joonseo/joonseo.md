# Item11

> **equals를 재정의하려거든 hashCode도 재정의하라.**
<br>

## hashCode

먼저, **hashCode** 메서드는 해시 기반 컬랙션에서 객체를 저장하거나 검색할 때 사용하는 메서드이다. 따라서, equals를 재정의하는 경우 **hashCode**를 재정의하지 않으면, 해시 기반 컬렉션을 사용할 때 문제를 일으킬 가능성이 높다.

<br>

Object 클래스는 아래와 같은 규약을 정의하고 있다.


- **equals** 비교에 사용되는 필드가 변경되지 않았다면, 한 런타임 내에서는 **hashCode** 메서드는 몇 번을 호출해도 **일관성 있게 같은 값을 반환**해야 한다.
- equals(Object)가 두 객체를 같다고 판단했다면, **두 객체의 hashCode 또한 같은 값을 반환**해야 한다.
- equals(Object)가 두 객체를 다르다고 판단했어도, **hashCode까지 다른 값을 반환할 필요는 없**다. 하지만 다른 객체에 대해서는 다른 값을 반환하는 것이 해시테이블 성능상 좋다.

<br>

 **hashCode**를 재정의하지 않으면, 재정의한 **equals** 메서드에서는 **true**를 반환하지만, **hashCode**에서는 **false**를 반환할 수도 있는 상황이 발생한다. 

 <br>

## hashCode 재정의

```java
@Override
public int hashCode(){
	return 42;
}
```

<br>

 위와 같이 재정의하면 동일 클래스 타입에 대해 모든 인스턴스가 해시값 **42**를 갖게 된다. 이러면 일관성이야 지킬 수 있겠지만, **해시 기반 검색이 불가능**하므로 조회 속도가 현저히 떨어지게 된다. 즉, 해시 기반 컬랙션이 갖는 이점을 상실하게 되는 것이다.

 <br>

아래는 좋은 hashCode 메서드를 작성하는 요령이다.

```
1. int 변수 result를 선언한 후, c로 초기화 한다.
(c는 첫번째 핵심필드를 2.a 방식으로 계산한 해시코드이다)

2. 객체의 핵심필드 f에 대해 아래 작업을 수행한다.

2-a. 기본 타입 필드라면, Type.hashCode(f)를 수행한다.
		 참조 타입 필드이고 hashCode가 연쇄적으로 호출되는 상황이라면, 표준형의 hashCode를 수행한다.
		 (필드값이 null이라면, 0을 사용한다.)
		 배열 타입 필드라면, 핵심 원소 각각을 별도의 필드로 다룬다.
		 
2-b. 2-a에서 계산한 방식으로 result c를 갱신한다.
			-> result = 31 * result + c
```

<br>

좋은 hashCode 메서드는 아래와 같이 재정의할 수 있겠다.

<br>

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

 31을 곱하는 이유는 홀수이면서 소수이기 때문이다. 만약 짝수를 곱하게 되면 오버플로우 발생시, 정보를 잃게된다.

 <br>

 **Objects** 클래스는 위와 같은 기능을 제공하는 **hash** 메서드를 제공한다. 하지만, 입력 매개변수에 따라 이를 담는 배열이 만들어지고, 매개변수의 타입따라 언박싱과 박싱을 거쳐야하기 때문에 **일반적으로 더 느리다.** 

 <br>

```java
@Override
public int hashCode(){
	return Objects.hash(a, b, c)
}
```

<br>

 불변 클래스이면서 해시코드를 계산하는 비용이 크다(필드가 많거나, 박싱/언박싱이 필요한 케이스가 많은 경우)면 **캐싱기반의 지연초기화**를 고려해볼 수 있을 것 같다.

 <br>

```java
public class PhoneNumber {
    private final short areaCode, prefix, lineNum;
    private int hashCode; // 0으로 초기화 (캐시되지 않음을 의미)
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix = rangeCheck(prefix, 999, "prefix");
        this.lineNum = rangeCheck(lineNum, 9999, "line num");
        // hashCode는 0으로 초기화됨 (기본값)
    }
    
    @Override
    public int hashCode() {
        int result = hashCode;
        if (result == 0) { // 아직 계산되지 않았음
            result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            result = 31 * result + Short.hashCode(lineNum);
            hashCode = result; // 캐싱
        }
        return result;
    }
}
```

<br>

특정 필드를 지연초기화하려면, 해당 클래스의 **스레드 안정성을 보장**해야 하긴 하지만, 실제 해시값이 필요할 때까지 비용이 큰 초기화를 미룰 수 있고 캐싱까지 가능하니 이점이 있다.
