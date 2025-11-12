# 아이템 28

# 아이템 28. 배열보다는 리스트를 사용하라

## 배열 vs 제네릭

1. 공변 vs 불공변
    1. 배열은 공변임
        
        공변 : 하위 타입 관게가 배열에서도 그대로 유지(예 : Super(부모) - Sub(자식이면 
        
        Super[] - Sub[] 도 부모-자식 관계)
        
        ```java
        Object[] objectArray = new Long[1]; // Long은 Object의 하위 타입이므로 배열도 허용
        objectArray[0] = "문자열"; // ArrayStoreException 배열은 공변이라서 컴파일은 되지만 런타임에서는 문법 검사를 함
        ```
        
    2. 제네릭은 불공변
        
        불공변 : 공변이 안 통함
        
        List<Super타입>, List<Sub타입>은 아무 관계가 없음, 두 관계는 부모-자식 관계가 x
        
        ```java
        List<Object> ol = new ArrayList<Long>(); // 컴파일 오류
        ```
        
    3. 결국 배열, 제네릭 모두 Long 저장소에 String을 넣을 수 없는 건 동일함
        
        하지만 배열은 런타임에 잡고, 리스트는 컴파일에 잡음
        
        따라서 컴파일 오류를 내는 리스트가 더 안전
        

1. 실체화
    1. 배열은 실체화됨
        
        실체화 : 런타임에도 자신이 담기로한 원소의 타입을 인지/검사 할 수 있음
        
        즉 위의 new Long[1]은 런타임에도 Long 원소의 배열이라는 정보를 가지고 있음으로
        
        String을 넣으면 런타임 오류(ArrayStoreException)
        
    2. 제네릭은 타입 정보가 소거됨
        
        컴파일 시점에만 원소 타입 검사, 런타임에는 알 수 없음
        
        따라서 제네릭은 불공변해야함
        
    

## 자바에서 제네릭 배열 생성이 허용되지 않는 이유

```java
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42);              // (2)
Object[] objects = stringLists;                   // (3)
objects[0] = intList;                             // (4)
String s = stringLists[0].get(0);                 // (5)
```

(1) : 제네릭 배열은 생성할 수 없지만 이유를 알기 위해 일단 허용한다고 가정

(2) : 제네릭 Integer 생성

(3) : stringLists는 배열이므로 Object[]로 참조 가능

(4) : stringLists, intList모두 런타임시에는 그냥 List로만 보이기 때문에 성공

(5) : 여기서 Integer를 String으로 형변환할려고 하기 때문에 ClassCastException 발생

→ 따라서 (1)번을 애초에 자바에서 금지

## 배열 대신 리스트 사용 권장

E[] 대신 List<E> 사용

코드 약간 복잡, 성능 약간 하락

타입 안전성, 상호 운용성 상승

## 예시

```java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

경고 없음

컴파일, 런타임 모두 안전

## 핵심정리

배열은 런타임에는 타입체크를 하고 컴파일에는 타입체크 안함

제네릭은 반대로 컴파일에 타입 체크함

따라서 둘을 함께 쓰는 것은 위험

배열 대신 리스트(제네릭)에 쓰는 것이 안전