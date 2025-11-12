# Item33

> ***타입 안전 이종 컨테이너를 고려하라***

<br>

### 타입 안전 이종 컨테이너란

**Map**이나 **Set** 자료구조를 사용하게 되면, **한가지 타입의 원소만 삽입**할 수 있다. 이런 제약사항을 극복하기 위한 패턴이 바로 **타입 안전 이종 컨테이너 패턴**이다.

<br>

원소의 **클래스 타입**을 **Key**로, **값**을 **Value**로 넣는 방식으로 여러가지 타입의 원소들을 한개의 **Map**에 넣을 수 있다. 아래 예시를 살펴보자.

```java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();
    
    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(type, instance);
    }
    
    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

<br>

이걸 보자마자 든 생각은 **각 클래스 타입 당 하나의 원소만 저장할 수 있는거 아닌가**였다. 과일 바구니에 과일을 종류별로 하나씩 담는건데, 어떤 상황에 효율적일지 잘 상상이 가지 않았다.

<br>

키로 **Class<?>** 타입을 사용하고 있다. **Class<?>** 는 타입을 담을 수 있는데, 아래 예시를 살펴보자

```java
Class<?> key1 = String.class;    // Class<String>
Class<?> key2 = Integer.class;   // Class<Integer>
Class<?> key3 = Boolean.class;   // Class<Boolean>

// String.class와 같은 리터럴을 타입토큰이라고 함!
```

<br>

이전 아이템에서 **비한정적 와일드카드 타입(?)**을 사용하면, 해당 객체에 대한 **쓰기**가 불가능하다고 했다. 근데 어떻게 가능한걸까?

<br>

비한정적 와일드카드 타입에 대한 쓰기가 불가능했던 이유는 아래의 예처럼 어떤 타입이 들어올지 알 수 없었기 때문이다. 하지만, **Map**의 **Key** 경우에는 **Class** 타입이라는 건 알 수 있기 때문에 문제될 것이 없다. 이런 상태를 **중첩**되어있다고 한다.

<br>

```java
// 삽입, 삭제 불가!
List<?> list = new ArrayList<>()

// Map의 Key가 와일드카드 타입을 포함하는 것은 가능
// Class 타입이 키로 사용될거라는 것은 알 수 있기 때문에
Map<Class<?>, Object> map; 
```

<br>

### 주의사항

**로타입을 사용하면 안정성이 떨어진다.**

만약, 악의적인 클라이언트가 로타입을 키로 해서 잘못된 값을 넣게 되면, 그 값을 다시 꺼내 캐스팅하는 과정에서 문제가 생긴다.

<br>

```java
Class stringClass = String.class;  
f.putFavorite(stringClass, 123);
```

<br>

 위 코드가 가능한 이유는 **putFavorite** 메서드에 파라미터가 바인딩될 때, 로타입이 들어오면 **putFavorite(Class<Object>, Object)** 형태로 타입이 추론되어 **Map** 객체가 오염된다.

<br>

 따라서 클라이언트가 제공한 **type**과 **instance**의 타입이 일치하는지 확인하는 절차가 필요하다. Java의 컬렉션 라이브러리 중 **checkedSet**, checkedList, **checkedMap**은 이러한 검증 절차를 반영한 컬렉션을 반환한다.

<br>

**실체화 불가 타입은 사용할 수 없다.**

**List<String>**이나 **List<Integer>**와 같은 타입들은 실체화가 불가능하며, 런타임에는 모두 **List.class** 타입으로만 표현된다.

<br>

이를 우회하기 위해 **슈퍼 타입 토큰**(Super Type Token) 방식을 사용할 수도 있지만, 이 방법도 완벽하지는 않아 주의해서 사용해야 한다.

<br>

### 한정적 타입 토큰

위 예시 **Favorites**에서 사용하는 타입 토큰은 비한정적이다. 어떤 **Class** 타입이든 받아들일 수 있기 때문이다.

<br>

한정적 타입 토큰을 통해 **Favorites**에서 받아들일 타입을 제한할 수 있다. 애너테이션 API에서 이를 적극 활용하고 있는데 아래 예시를 보자.

```java
public <T extends Annotation>
T getAnnotation(Class<T> annotationType)
```

<br>

런타임에 **Class<?>** 타입 객체가 있고, 이를 **Class<? extends Annotation>**을 인수로 받는 메서드에 전달해야 하는 상황을 가정해보자. 어떻게 안전하게 전달할 수 있을까?

<br>

단순 캐스팅은 비검사 경고가 발생하므로 안전하지 않다. 다행히도 **Class** 클래스는 **asSubclass** 메서드를 제공한다. 이 메서드는 호출한 **Class** 객체를 인수로 전달된 클래스의 하위 클래스로 캐스팅하며, 캐스팅에 실패하면 **ClassCastException**을 던진다. 따라서 타입 검사까지 수행하여 안전하게 타입 변환을 보장한다.

<br>

```java
public <T extends Annotation> T getAnnotation(Class<?> unknownType) {
    Class<? extends Annotation> annotationType = 
        unknownType.asSubclass(Annotation.class);
    
    return getAnnotation(annotationType);
}
```
