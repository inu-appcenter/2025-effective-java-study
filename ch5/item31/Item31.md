# Item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

- 공변(Covariant)
    - SubType이 SuperType의 하위 타입일 때 SubType은 SuperType이 될 수 있음.
    
    ```jsx
    class Animal {
        ...
    }
    
    class Cat extends Animal {
        ...
    }
    
    Animal animal = new Cat(); 
    ```
    

- 불공변(Invariant)
    - SubType이 SuperType의 하위 타입일 때 SubType은 SuperType이 될 수 없고, SuperType은 SubType이 될 수 없음.
    - String은 Object의 하위 타입이지만 List<String>는 List<Object>의 하위 타입이 아님.
    - 제네릭은 불공변하게 제공됨.
        
        ```jsx
        // 제네릭이 공변인 경우
        List<String> strings = new ArrayList<>();
        strings.add("hello");
        
        List<Object> objects = strings;
        objects.add(123);
        objects.add(new Date());
        ```
        
        → 런타임 오류 발생할 위험이 있음.
        
    

- 제네릭의 불공변적인 특성때문에 아래와 같은 상황 발생
    1. 
        
        ```jsx
        public class Stack<E> {
        	public Stack();
        	public void push(E e);
        	public E pop();
        	public boolean isEmpty();
        }
        
        public void pushAll(Iterable<E> src) {
        	for (E e: src)
        		push(e);
        }
        
        //Number는 Integer의 상위 타입이지만, 제네릭은 불공변
        //Integer를 Stack<Number>에 넣을 수 없음
        Stack<Number> numberStack = new Stack<>();
        Iterable<Integer> integers = ...;
        numberStack.pushAll(integers);
        ```
        
        - 이러한 상황을 위해 한정적 와일드 카드 타입 `<? extends E>` 제공
            
            ```jsx
            public void pushAll(Iterable<? extends E> src) {
            	for (E e : src)
            		push(e);
            }
            ```
            
            `<? extends E>`  → E의 하위타입인 매개변수 사용
            
        
    2. 
        
        ```jsx
        public void popAll(Collection<E> dst) {
        	while (!isEmpty())
        		dst.add(pop());
        }
        
        // 스택에 있는 요소들 Object 컬렉션으로 옮기는 경우
        // Number를 Collection<Object>에 넣을 수 없음
        Stack<Number> numberStak = new Stack<>();
        Colection<Object> objects = ...;
        numberStack.popAll(objects);
        ```
        
        - 이러한 상황을 위해 한정적 와일드 타입 `<? super E>` 제공
            
            ```jsx
            public void popAll(Collection<? super E> dst) {
            	while (!isEmpty())
            		dst.add(pop());
            }
            ```
            
            `<? super E>`  → E의 상위타입인 매개변수 사용 
            

- PECS: Producer-Extends, Consumer-Super
    - 타입 매개변수가 생산자라면 Extends, 소비자라면 super를 사용하자.
    - 단, 매개변수가 생산자와 소비자의 역할을 동시에 한다면 와일드카드 타입을 쓰지 말아야 한다.
    - 클래스 사용자가 와일드카드 타입을 신경써야 한다면 API 자체에 문제가 있을 가능성이 높다.

- Item 30의 max 코드 수정
    
    ```jsx
    // 기존 max 코드
    public static <E extends Comparable<E>> E max(List<E> list)
    
    // 수정된 max 코드
    public static <E extends Comparable<? super E>> E max(
    	List<? extends E> list)
    ```
    
    → 기존의 max 코드는 `List<ScheduledFuture<?>> scheduledFutueres = …;` 를 처리할 수 없음.
    
    - ScheduledFuture가 Comparable 구현 X
        
        → 기존 max 코드의  `E extends Comparable<E>` 에서 걸러짐
        
    - ScheduledFuture는 Delayed의 하위 인터페이스, Delayed는 Comparable 확장 O
        
        → 수정된 max 코드의  `E extends Comparable<? super E>` 에서 E가 ScheduledFuture라면 ?는 E의 상위 타입, 즉 Delayed와 같은 상위 타입 중 Comparable을 구현한 타입이 들어올 수 있음
        

```jsx
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

→ 저자는 타입 매개변수가 하나인 경우에는 아래의 코드를 사용하고, 헬퍼 메서드(<?>에는 쓰기가 불가능하므로)를 작성하라고 했는데, 개인적으로는 그렇게 해서 좋을 점이 뭔지 모르겠음. 개인적인 스타일 차이가 아닌가.. 라고 생각합니다.