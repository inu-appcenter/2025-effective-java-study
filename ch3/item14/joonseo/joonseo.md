# Item14

> ***Comparable을 구현할지 고려하라***

<br>

## compareTo

**Comparable** 인터페이스의 **compareTo**는 **Object**의 **equals**와 매우 유사하지만, 동치성 비교를 넘어 순서까지 비교할 수 있다. 즉, 이를 구현한 클래스는 순서(크기, 알파벳순 등)가 있는 필드가 존재한다는 것이다. 

<br>

### compareTo 일반 규약

<br>

compareTo 메서드를 작성하는 일반 규약은 다음과 같다.

<br>

- 구현 객체와 비교할 수 없는 타입의 객체가 주어지면, **ClassCastException**을 반환한다.
  
<br>

- **Comparable**을 구현한 클래스는 모든 x와 y에 대해 **x.compareTo(y) == -y.compareTo(x)**를 보장해야 한다.
  
<br>

- **Comparable**을 구현한 클래스는 **추이성**을 보장해야 한다. 즉, **x.compareTo(y) > 0**이고 **y.compareTo(z) > 0**이면, **x.compareTo(z) > 0**을 만족해야 한다.
  
<br>

- 필수는 아니지만, **x.compareTo(y) == 0**은, **x.equals(y) == true** 의 결과와 같아야 한다.

<br>

**equals** 메서드의 일반 규약에서 언급한 **반사성**, **대칭성**, **추이성**을 충족해야 함을 명시하고 있다.

<br>

**compareTo** 메서드는 각 필드가 동치인지를 확인하는 것이 아니라 그 **순서**를 확인하기 위해 사용된다. 만약, 해당 필드가 **객체 참조 필드**라면 그 객체의 **compareTo**를 재귀적으로 호출해서 비교해야 한다. 

<br>

**Comparable**을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면, 비교자 **Comparator**를 대신 사용한다. 자바가 제공하는 비교자를 사용할 수도 있고, 쓰임에 따라 직접 구현할 수도 있다.

<br>

아래는 자바가 제공하는 비교자를 사용하는 예시이다. 

```java
private final class CaseInsensitiveString 
			implement Comparable<CaseInsenstiveString>{ // CaseInsensitiveString만 비교 
		
		public int compareTo(CaseInsensitiveString cis){
				return String.CASE_INSENSITIVE_OREDER.compare(s, cis.s);
		}
```

<br>

## Comparator

자바 8부터는 **Comparator** 인터페이스가 비교자 생성 메서드와 함께 연쇄 방식으로 비교자를 생성 & 적용할 수 있게 되었다. 적용전, 후 예시를 살펴보자.

<br>

**적용전**

```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode); // 가장 중요한 필드
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix); // 두 번째로 중요한 필드
        if (result == 0)
            result = Short.compare(lineNum, pn.lineNum); // 세 번째로 중요한 필드
    }
    return result;
}
```

<br>

**적용후**

```java
private static final Comparator<PhoneNumber> COMPARATOR =
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

<br>

**Comparator**을 사용한 코드가 가독성이나 유지보수 측면에서 좋아보이지만, 성능상으로 10퍼센트 정도 느리다고 한다.

<br>

**Comparator**는 **int**, **long**, **double**에 대한 **보조 생성 메서드**를 제공한다. **short** 타입은 **int**의 보조 생성 메서드인 **comparingInt**와 **thenComparingInt 메서드**를, **float** 타입은 **comparingDouble**과 **thenComparingDouble 메서드를** 사용하면 된다. 

<br>

**comparing** 메서드는 2가지 버전으로 오버로딩되어 있다.

<br>

**키 추출자만 받아 자연적 순서를 사용**

```java
public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
        Function<? super T, ? extends U> keyExtractor)
{
    Objects.requireNonNull(keyExtractor);
    return (Comparator<T> & Serializable)
        (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
}
```

<br>

**키 추출자, 키 비교자 둘다 받아 정의된 비교자의 순서를 사용**

```java
public static <T, U> Comparator<T> comparing(
        Function<? super T, ? extends U> keyExtractor,
        Comparator<? super U> keyComparator)
{
    Objects.requireNonNull(keyExtractor);
    Objects.requireNonNull(keyComparator);
    return (Comparator<T> & Serializable)
        (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),
                                          keyExtractor.apply(c2));
}
```

<br>

마지막으로, 비교 연산을 직접 수행하기 보다 각 타입의 래퍼 클래스가 제공하는 **compare** 메서드를 사용하자. 오버플로우를 방지하고 정확한 부동 소수점 계산을 제공한다.

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();  // 오버 플로우 발생 가능
    }
};
```

<br>

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

<br>

```java
static Comparator<Object> hashCodeOrder = 
    Comparator.comparingInt(o -> o.hashCode());
```

```java

```
