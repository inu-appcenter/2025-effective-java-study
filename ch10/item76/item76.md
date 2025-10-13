# Item76

> ***가능한 한 실패 원자적으로 만들라***

<br>

## 실패 원자적

아이탬에서 언급하는 **실패 원자적**이란 단어의 의미는, 로직 수행 중 예외가 발생했어도 대상 객체는 메서드 호출 전의 상태를 유지해야 한다라는 뜻을 담고 있다.

트랜젝션 내에서 예외가 발생하면 이전 작업들이 롤백되듯이 말이다. 

<br>

### 방법 1: 불변 객체로 설계하라

불변 객체는 초기화되는 동시에 절대 값을 수정할 수 없다. 따라서, 예외가 발생하든 안하든 객체의 상태는 변경될 수 없다.

```java
public final class ImmutablePoint {
    private final int x;
    private final int y;
    
    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    // 해당 메서드가 실패하더라도, 기존 객체가 불안정한 상태에 빠질 수는 없다.
    public ImmutablePoint move(int dx, int dy) {
        return new ImmutablePoint(x + dx, y + dy);
    }
}
```

<br>

### 방법 2: 작업을 수행하기 전 매개변수의 유효성을 검사하라

즉, 예외를 반환할 가능성이 있는 것들을 미리 검사하여 잠재적 예외 가능성을 차단하는 방법이다.

```java
public Object pop() {
		// 객체의 상태를 변경하기 전에 매개변수의 유효성을 미리 검사
    if (size == 0)
        throw new EmptyStackException();
    
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```

<br>

### 방법 3: 임시 복사본에서 작업을 수행하라

객체의 임시 복사본에서 작업을 수행한 다음, 작업이 마무리되면 원래 객체와 교체하는 방법이다.

```java
public void sort(List<String> list) {
		// 복사본 생성
    String[] array = list.toArray(new String[0]);
    
    // 복사본에서 작업 수행
    Arrays.sort(array);
    
    // 성공 시에만 원본에 반영
    for (int i = 0; i < array.length; i++) {
        list.set(i, array[i]);
    }
}
```

<br>

### 방법 4: 복구 코드를 작성하라

작업 도중 발생하는 실패를 가로채 예외가 발생하면 미리 백업해둔 데이터를 덮어쓰는 방법이다. 
~~자주 쓰이는 방식은 아니라고 한다.~~

```java
public void updateDatabase(Data data) {
    Data backup = currentData.clone();  // 백업
    
    try {
        currentData = data;
        persistToDisk(data);  
    } catch (IOException e) {
        currentData = backup;  // 실패 시 복구
        throw new DatabaseException("Update failed", e);
    }
}
```

<br>

### 다만

실패 원자성을 항상 유지해야 하는 것은 아니다. 이 실패 원자성을 달성하기에 비용이나 복잡도가 큰 연산도 있고 수지타산을 따져봤을 때, 실이 더 않은 경우도 존재하기 때문이다.
