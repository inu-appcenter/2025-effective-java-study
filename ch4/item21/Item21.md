# Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

- 자바 8 이전에 기존 구현체를 깨뜨리지 않고는 메서드를 추가할 방법이 없었음.
    
    → 인터페이스를 구현하는 구현체는 추상메서드를 전부 구현해야 하므로, 인터페이스에 메서드 추가하면 구현체는 새로 추가된 메서드를 필수로 구현해야 했음.
    
    → 자바 8 이후 디폴트 메서드가 생겼지만, 디폴트 메서드가 기존 구현체와 매끄럽게 연동되리라는 보장은 없음.
    

```jsx
@Override
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter); 
    boolean result = false;

    for (Iterator<E> it = iterator(); it.hasNext(); ) {
        E element = it.next();              
        if (filter.test(element)) {         
            it.remove();                    
            result = true;                 
            }
    }
    return result;
}

```

위 코드는 자바 8에 추가된 컬렉션 인터페이스에 추가된 removeIf 메서드임. 

→ Predicate가 true를 반환하면 remove 메서드를 호출해 원소를 제거하는 메서드

- 아파치의 SynchronizedCollection 클래스는 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스임.

- 이 책을 쓰는 시점에 removeIf 메서드를 재정의하지 않고 있음.
    
    → 아파치의 SynchronizedCollection에서 사용할 수 있는 removeIf는 동기화를 사용하지 못함.
    
    →  removeIf를 사용했을 때 의도치 않은 결과가 발생할 수 있음.
    

- 디폴트 메서드는 기존 구현체에 런타임 오류를 발생시킬 수 있으므로, 설계할 때 세심한 주의 필요
    
    → 인터페이스를 새로 만들었으면 최소한 세개는 구현체를 만들어볼 것