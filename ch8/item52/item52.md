# 아이템 52

## 아이템 52: 다중정의는 신중히 사용하라

## 컴파일 vs 런타임

컴파일 vs 런타임

컴파일 : 고급언어(코드) 를 기계어로 번역하는 과정(문법 오류 등 검사)

런타임 : 프로그램 실행 후에 발생하는 과정(메모리 부족 …)

## 다중정의 vs 재정의

### 재정의(Overriding)

```java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override 
    String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override 
    String name() { return "샴페인"; }
}

List<Wine> wines = List.of(
    new Wine(),  // 힙 메모리의 0x01에 저장
    new SparklingWine(), // 힙 메모리의 0x02에 저장
    new Champagne() // 힙 메모리의 0x03에 저장
);

for (Wine wine : wines) {
    System.out.println(wine.name()); // 2번째 아이템의 경우 0x02를 갔더니 SparklingWine이여서 
																	    // SparklingWine.name을 호출
}

-----------------------

포도주
발포성 포도주
샴페인
```

런타임에 wine.name()을 할 때 실제 객체 타입으로 결정

### 다중정의(Overloading)

```java
public class CollectionClassifier {

    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }
    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections  = {
                new HashSet<>(),
                new ArrayList<>(),
                new HashMap<>().values()
        };

        for (Collection<?> c : collections) {
            System.out.println(classify(c));
        }    
    }
}

-----------

그 외
그 외
그 외
```

컴파일 시점에 c의 선언 타입이 Collection이므로 classify를 할 때 매개변수가 Collection타입인 classify를 호출

### 해결방법

**방법 1: instanceof 사용**

```java
public static String classify(Collection<?> c) {
    return c instanceof Set ? "집합" :
            c instanceof List<?> ? "리스트" : "그 외";
}

public static void main(String[] args) {
    Collection<?>[] collections  = {
            new HashSet<>(),
            new ArrayList<>(),
            new HashMap<>().values()
    };

    for (Collection<?> c : collections) {
        System.out.println(classify(c));
    }    
}
```

**방법 2: 메서드 이름을 다르게**

```java
public static String classifySet(Set<?> s) {
    return "집합";
}

public static String classifyList(List<?> lst) {
    return "리스트";
}

public static String classifyCollection(Collection<?> c) {
    return "그 외";
}
```

### 실전 예시 ObjectOutputStream

나쁜 예시(다중정의)

```java
class BadOutputStream {
    void write(boolean b) { }
    void write(int i) { }
    void write(long l) { }
    void write(float f) { }
    void write(double d) { }
}
```

### 좋은 예시(이름 구분 메서드)

```java
class ObjectOutputStream {
    void writeBoolean(boolean b) { }
    void writeInt(int i) { }
    void writeLong(long l) { }
    void writeFloat(float f) { }
    void writeDouble(double d) { }
    
    // write, read의 이름이 짝을 지어서 있음
    boolean readBoolean() { }
    int readInt() { }
    long readLong() { }
}
```

## 오토박싱(wrapper-primitive 자동 형변환)으로 인한 혼란

목록(-3,-2,-1,0,1,2)에서 숫자 0, 1, 2를 삭제하고 싶은 상황

```java
public class MainAutoBoxing {

    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);   // -3, -2, -1, 0, 1, 2
            list.add(i);  // -3, -2, -1, 0, 1, 2
        }

        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }

        System.out.println(set + " " + list);
    }
}

-------

[-3, -2, -1] [-2, 0, 2]

-------

// list의 경우 삭제 방법(index 기준)
[-3, -2, -1, 0, 1, 2]
remove(0) → [-2, -1, 0, 1, 2]  // 인덱스 0 제거
remove(1) → [-2, 0, 1, 2]      // 인덱스 1 제거
remove(2) → [-2, 0, 2]         // 인덱스 2 제거

---------

/**
 * Removes the element at the specified position in this list (optional
 * operation).  Shifts any subsequent elements to the left (subtracts one
 * from their indices).  Returns the element that was removed from the
 * list.
 *
 * @param index the ***index*** of the element to be removed
 * @return the element previously at the specified position
 * @throws UnsupportedOperationException if the {@code remove} operation
 *         is not supported by this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= size()})
 */
E remove(int index);

```

### 해결

강제 형변환

```java
for (int i = 0; i < 3; i++) {
    set.remove(i);
    list.remove(Integer.valueOf(i)); // index가 아니라 element를 기준으로 삭제
}

/**
 * Removes the first occurrence of the specified element from this list,
 * if it is present (optional operation).  If this list does not contain
 * the element, it is unchanged.  More formally, removes the element with
 * the lowest index {@code i} such that
 * {@code Objects.equals(o, get(i))}
 * (if such an element exists).  Returns {@code true} if this list
 * contained the specified element (or equivalently, if this list changed
 * as a result of the call).
 *
 * @param o ***element*** to be removed from this list, if present
 * @return {@code true} if this list contained the specified element
 * @throws ClassCastException if the type of the specified element
 *         is incompatible with this list
 * (<a href="Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if the specified element is null and this
 *         list does not permit null elements
 * (<a href="Collection.html#optional-restrictions">optional</a>)
 * @throws UnsupportedOperationException if the {@code remove} operation
 *         is not supported by this list
 */
boolean remove(Object o);

```

## 람다에서의 문제점

```java
    public static void main(String[] args) {
//        new Thread(System.out::println).start(); // 성공, Thread의 생성자는 단 하나
        ExecutorService exec = Executors.newCachedThreadPool();
        exec.submit(System.out::println); // 컴파일 오류
        // Reference to 'println' is ambiguous, both 'println()' and 'println(boolean)' match
    }
    
// submit 다중정의
<T> Future<T> submit(Callable<T> task);
Future<?> submit(Runnable task);

// println도 다중정의됨
void println(String x);
void println(int x);
void println(Object x);
```

### 해결방법

```java
// 명시적으로 타입 지정
exec.submit((Runnable) System.out::println);

// 또는 람다로 명확히
exec.submit(() -> System.out.println());

exec.submit((Callable<Void>) () -> {
    System.out.println("Hello");
    return null;
});
    
```

## **생성자 다중정의**

### **안전한 예시**

```java
public class ArrayList<E> {
    public ArrayList(int initialCapacity) {
    }
    
    public ArrayList(Collection<? extends E> c) {
    }
}

// 사용
new ArrayList<>(10);                    // int 생성자 호출
new ArrayList<>(Arrays.asList(1, 2, 3)); // Collection 생성자 호출
```

### 불안한 예시

```java
public class Confusing {
    public Confusing(String s) { 
        System.out.println("String 생성자 호출");
    }
    
    public Confusing(CharSequence cs) { 
        System.out.println("CharSequence 생성자 호출");
    }
}

String str = "hello";
new Confusing(str);  // String 생성자 호출

-----------------

public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence,
               Constable, ConstantDesc {
-----------------

public static void main(String[] args) {
  Set<?> set = new HashSet<>();
  List<?> list = new ArrayList<>();
  Collection<?> collection = new HashMap<>().values();

  System.out.println(classify(set));        
  System.out.println(classify(list));     
  System.out.println(classify(collection)); 
}

---------------------

집합
리스트
그 외
```

파라미터가 가장 구체적인 메서드 선택

public final class String implements CharSequence

**7. 포워딩으로 일관성 보장**

```java
public final class String {
    // 자바 4 시절부터 존재
    public boolean contentEquals(StringBuffer sb) {
        return contentEquals((CharSequence) sb);  
    }
    
    // 자바 5에서 추가
    public boolean contentEquals(CharSequence cs) {
    }
}
```

특수한 버전이 일반적인 버전으로 작업을 위임하여 일관성 보장

## 핵심 정리

### 다중정의를 피해야 하는 경우

1. **매개변수 개수가 같을 때**
2. **가변인수 사용 시**
3. **함수형 인터페이스를 같은 위치(파라미터)에 받을 때**

### 안전한 다중정의 조건

1. **매개변수 타입이 근본적으로 다를 때(형변환 막기)**
2. **불가피한 경우 포워딩으로 동작 일관성 보장**

### 대안

- **메서드 이름을 다르게 짓기**
- **정적 팩터리 메서드 사용(생성자는 이름을 다르게 못 작성하기 때문에 정적팩토리 메서드로 생성자를 한 번 감싸서 사용)**