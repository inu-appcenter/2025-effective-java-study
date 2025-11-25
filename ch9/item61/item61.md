# 아이템 61

# 아이템 61 — 박싱된 기본 타입보다는 기본 타입을 사용하라

## 기본 타입(primitive) vs 박싱된 기본 타입(wrapper)

### 기본 타입 (primitive)

- `int`, `long`, `double`, `boolean`
- 값(숫자) 자체를 저장함
- null 불가
- 빠르고 가벼움

### 박싱된 기본 타입 (wrapper)

- `Integer`, `Long`, `Double`, `Boolean`
- 객체(참조 타입)
- null 가능
- 느리고 메모리도 더 많이 사용

## 차이점 1

### 값 vs. 식별성

기본 타입은 값

박싱 타입은 객체라서 주소를 가짐

```jsx
new Integer(42) == new Integer(42) -> false
42 == 42 -> true

```

## 차이점 2

기본 타입은 null 불가능, 박싱 타입은 null 가능

```java
Integer i = null;
int j = i;     // NullPointerException

i를 언박싱하는 과정에서 null 이므로 예외 발생
```

## 차이점 3

기본타입은 더 빠르고 메모리 적게씀

기본 타입은 가져와서 쓰는 거고, 박싱 타입은 직접 객체를 생성해서 씀

## 박싱 타입의 위험 사레

### Comparator

박싱 타입에 == 를 쓰면 안 되는 이유

```java
Comparator<Integer> naturalOrder =
    (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
    
naturalOrder.compare(new Integer(42), new Integer(42))

i < j 는 두 Integer가 자동 언박싱이 되어서 비교하므로 정상적으로 숫자적으로 비교됨
i == j 는 두 객체의 주소를 비교하므로 같다고 안 나옴
```

### null 때문에 발생하는 NPE

```java
public class Unbelievable {
    static Integer i;

    public static void main(String[] args) {
        if (i == 42)
            System.out.println("믿을 수 없군!");
    }
}

i == 42할 때 i(null)을 int로 언박싱 시도하다가 NullPointerException 발생
```

### 성능 저하 코드

```java
Long sum = 0L;

for (long i = 0; i <= Integer.MAX_VALUE; i++) {
    sum += i;
}

sum += i 할 때 
Long인 sum을 long으로 언박싱을 하고
i랑 더하고
결과를 다시 Long으로 박싱하고
새로운 Long 객체 생성

엄청 느려짐

해결책
long sum = 0L;  // 기본 타입
```

## 그럼 박싱 타입은 언제 사용해야 하는가?

1. 컬렉션(List, Set, Map)에 저장할 때
컬렉션은 **기본 타입을 저장할 수 없다**
2. 제네릭 타입의 타입 파라미터

### 이유

자바의 제네릭은 컴파일할 때 타입 정보를 지움

```java
List<Integer> → 컴파일 후 → List
List<String> → 컴파일 후 → List
```

런타임에는 **모든 제네릭은 똑같은 타입(List)** 으로 동작한다

그런데 기본 타입(int)은 JVM 내부에서 **완전히 다른 저장 구조**를 가짐

- int → 스택에 값 자체 저장 (primitive)
- Integer → 힙에 객체 형태로 저장 (reference)

이 두 구조는 타입 소거 방식에서 섞일 경우 충돌을 일으킨다

→  **그래서 제네릭은 처음부터 참조 타입만 받도록 설계됨**

## 결론

가능한 한 **기본 타입**을 사용하라
박싱 타입은 꼭 필요할 때만 사용해라
기본 타입과 박싱 타입을 혼용하지 말라
박싱 타입끼리 == 비교하지 말라