> ***표준 함수형 인터페이스를 사용하라***

<br>

## 함수형 인터페이스
 **함수형 인터페이스**란 **1개의 추상 메서드를 갖는 인터페이스**를 말한다. 여러 개의 메서드가 있다고 하더라도, 추상 메서드가 단 1개라면 함수형 인터페이스라고 할 수 있다.

<br>

 함수형 인터페이스를 정의하고 사용할 것이라면, **@FunctionalInterface** 어노테이션을 명시하자. 해당 인터페이스의 코드나 명세를 읽을 사람들에게 이 인터페이스는 람다용으로 사용될 것임을 명시하며, 유지보수 과정에서 다른 개발자가 실수로 메서드를 추가하는 사고를 방지할 수 있기 때문이다.

<br>

```java
@FunctionalInterface
public interface CustomFunctionalInterface<T> {
    boolean appCenter(T t);

    default void introduce(){
        System.out.println("Hello. This is AppCenter!");
    }

    static void study(){
        System.out.print("Hello. It's time to study!");
    }
}
```

<br>

## 표준 함수형 인터페이스

 저자는 함수형 인터페이스를 직접 만들어서 사용하기 보다, 표준 함수형 인터페이스를 사용하라고 한다. 책에서 언급한 **Map**을 캐시로 사용하는 예시를 한번 살펴보자. 

<br>

**함수형 인터페이스를 직접 정의**

```java
@FunctionalInterface
public interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}

Map<String, String> cache = new ModernLinkedHashMap<>(
    (map, eldest) -> map.size() > 100
);
```

<br>

**표준 함수형 인터페이스 사용**

```java
Map<String, String> cache = new ModernLinkedHashMap<>(
    (map, eldest) -> map.size() > 100
);
```

위 예시는 표준 함수형 인터페이스 **BiPredicate**를 사용한 예시이다. 이미 표준 함수형 인터페이스가 잘 구축되어 있기 때문에 용도에 맞게 적절하게 활용하면 된다.

<br>

### Function<T, R>

**T** 타입의 인수를 받아 **R** 타입의 값을 반환하는 함수형 인터페이스이다. 

<br>

**시그니처**

```java
R apply(T t);
```

<br>

**활용**

```java
List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
```

<br>

 **String::length**는 **String** 타입을 인수로 받아 **Integer** 타입을 반환하는 **length** 메서드의 참조를 사용한 것으로 함수형 인터페이스 `Function<String, Integer>`을 활용한 것이다.

<br>

### Predicate<T>

**T** 타입의 인수를 받아 **boolean** 타입을 반환하는 함수형 인터페이스이다.

<br>

**시그니처**

```java
boolean test(T t)
```

<br>

**활용**

```java
List<String> filtered = names.stream()
    .filter(s -> s.startsWith("G"))
    .collect(Collectors.toList());
```

<br>

### Consumer<T>

**T** 타입의 인수를 받아 아무것도 반환하지 않는 함수형 인터페이스이다.

<br>

**시그니처**

```java
void accept(T t)
```

<br>

**활용**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(System.out::println);
```

<br>

### Supplier<T>

인수를 받지 않고 **T**를 반환하는 함수형 인터페이스이다.

<br>

**시그니처**

```java
T get()
```

<br>

**활용**

```java
Chapter chapter = chapterRepository.findById(chapterId)
    .orElseThrow(() -> new RestApiException(CustomErrorCode.CHAPTER_NOT_FOUND));
```

<br>

### UnaryOperator<T>

**T** 타입의 인수를 받아 **T** 타입 값을 반환하는 함수형 인터페이스이다.

<br>

**시그니처**

```java
T apply(T t)
```

<br>

**활용**

```java
List<String> names = new ArrayList<>(Arrays.asList("alice", "bob"));
names.replaceAll(String::toUpperCase);
```

<br>

### BinaryOperator<T>

T 타입의 인수를 2개 받아 T 타입 값을 반환하는 함수형 인터페이스이다.

<br>

**시그니처**

```java
T apply(T t1, T t2)
```

<br>

**활용**

```java
(name, country) -> "이름: " + name + "국적: " + country
```

<br>

## 변형

위에서 설명한 함수형 인터페이스들은 기본 타입을 위한 다양한 변형도 제공한다. 

<br>

 각 기본 인터페이스는 기본 타입 **int**, **long**, **double**에 대해 `IntFunction<T>` 와 같은 변형을 제공하는데, **int** 타입의 입력 변수를 받아 **T**를 반환하는 식이다. 

<br>

```java
@FunctionalInterface
public interface IntFunction<R> {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    R apply(int value);
}

```

<br>

 만약, 반환값도 기본 타입이라면 **SrcToResult**의 형식을 따른다. 예를 들어, **long**을 입력으로 받아 **int**로 반환하는 **Function**은 **LongToIntFunction**이 되는 느낌이다. 

<br>

```java
@FunctionalInterface
public interface LongToIntFunction {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    int applyAsInt(long value);
}
```

<br>


 마지막으로 입력이 참조 객체이며 반환값이 기본 타입이라면 **ToResult**의 형식을 따른다. 만약, 입력이 **int[]**이고 반환값이 **Long**이라면, **ToLongFunction<int[]>**가 된다.

<br>

```java
@FunctionalInterface
public interface ToLongFunction<T> {

    /**
     * Applies this function to the given argument.
     *
     * @param value the function argument
     * @return the function result
     */
    long applyAsLong(T value);
}
```

<br>

 정말 마지막으로 **Supplier** 중 유일하게 **boolean** 타입을 반환하는 **Supplier**는 **BooleanSupplier**로 따로 정의 되어 있다.

<br>

```java
@FunctionalInterface
public interface BooleanSupplier {

    /**
     * Gets a result.
     *
     * @return a result
     */
    boolean getAsBoolean();
}
```

<br>

## 직접 구현해야 하는 경우

 만약, 필요한 함수형 인터페이스가 없다면 직접 구현하는 것이 맞다. 대신 구현하기 전 아래 조건에 부합하는지 따져보자.

<br>

- 자주 사용되며 이름 자체로 동작을 정확히 기술할 수 있다.
- 반드시 따라야 하는 규약이 존재한다.
- 유용한 디폴트 메서드를 제공할 수 있다.
