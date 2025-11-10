<aside>

적시에 방어적 복사본을 만들라

</aside>

자바는 메모리 충돌 오류에 대해 안전한 언어이다.

**왜?** JVM이 메모리 접근을 엄격히 관리하기 때문이다. 배열 범위를 벗어나거나, 해제된 메모리에 접근하거나, 타입이 맞지 않는 메모리 영역을 건드리면 즉시 예외가 발생한다.

**하지만…** 모든 공격으로부터 안전하다는 뜻은 아니다. 클라이언트가 클래스 내부 상태를 마음대로 바꿀 수 있다면? 메모리는 안전해도 비즈니스 로직의 불변식(**Invariant)**은 깨진다.

### 문제가 되는 상황

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException("시작이 종료보다 늦음");
        this.start = start;
        this.end = end;
    }

    public Date start() { return start; }
    public Date end() { return end; }
}
```

final 키워드도 붙였고, 생성자에서 유효성도 검사했으니 안전해 보인다. 하지만…

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78);  // 💥 Period 내부가 바뀜!
****
```

→ Period 생성자가 받은 Date 객체의 참조를 그대로 저장했기 때문이다.

외부에서 `end.setYear(78)`을 호출하면 Date 객체 내부의 상태가 변경된다. Date는 가변 클래스라서 내부 필드들이 final이 아니다. setYear 같은 메서드로 내부 상태를 마음대로 바꿀 수 있다.

### 해결1: 생성자에서 방어적 복사

```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    
    if (this.start.compareTo(this.end) > 0)
        throw new IllegalArgumentException("시작이 종료보다 늦음");
}
```

**핵심 1: 유효성 검사보다 복사를 먼저 한다**

```java
// ❌ 
if (start.compareTo(end) > 0)  // 1. 검사 (OK)
    throw new IllegalArgumentException(...);
// 2. 이 찰나에 다른 스레드가 start를 변경할 수 있음!
this.start = new Date(start.getTime());  // 3. 복사 (이미 늦음)
```

멀티스레드 환경에서 검사와 복사 사이에 다른 스레드가 원본을 수정할 수 있다(TOCTOU 공격). 복사를 먼저 하면 복사본은 외부와 단절되니 안전하다.

**핵심 2: clone을 쓰지 않는다**

Date는 final이 아니라서 악의적인 하위 클래스가 clone을 오버라이드할 수 있다. **확장 가능한 타입의 방어적 복사에는 clone을 쓰지 말자.**

```java
class MaliciousDate extends Date {
    private static List<Date> stolenDates = new ArrayList<>();
    
    @Override
    public Object clone() {
        MaliciousDate copy = new MaliciousDate();
        stolenDates.add(copy);  // 모든 복사본을 몰래 저장
        return copy;
    }
}
```

### **해결 2: 접근자에서도 방어적 복사**

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78);  // 💥 또 다른 공격!
```

접근자가 내부 객체의 참조를 그대로 반환하면, 외부에서 그걸로 내부를 수정할 수 있다.

```java
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```

이제 외부가 받는 건 복사본이다. 아무리 수정해도 Period 내부는 안전하다.

접근자에서는 clone을 써도 되지만, **그냥 생성자 쓰는 게 더 명확하고 안전하겠다…**

### **방어적 복사가 필요한 다른 상황들**

**컬렉션에 저장할 때**

```java
public class Zoo {
    private final Set<Animal> animals = new HashSet<>();
    
    // ❌ 외부에서 animal을 수정하면 Set의 해시값이 달라져서 오작동
    public void add(Animal animal) {
        animals.add(animal);
    }
    
    // ✅ 복사본 저장
    public void add(Animal animal) {
        animals.add(new Animal(animal));
    }
}
```

HashSet은 객체를 저장할 때 hashCode()를 계산해서 위치를 정한다. Animal 객체가 변경되면 hashCode()가 달라지는데, HashSet은 이미 예전 위치에 저장해뒀다. 그래서 contains로 찾고자 해도 불가능하다.

**배열 반환 시**

```java
private String[] animals = {"cat", "dog"};

// ❌ 내부 배열 직접 노출
public String[] getAnimals() {
    return animals;
}

// ✅ 복사본 반환
public String[] getAnimals() {
    return animals.clone();
}

// ✅✅ 더 나은 방법: 불변 뷰
public List<String> getAnimals() {
    return Collections.unmodifiableList(Arrays.asList(animals));
}
```

## 방어적 복사를 생략할 수 있는 경우

1. 같은 패키지 내부라서 호출자를 신뢰할 때
2. 통제권 이전이 명확할 때 (더 이상 수정하지 않기로 약속이 되어 있어야 함)
3. 불변식이 깨져도 피해가 호출자에게만 국한될 때 (다른 클라이언트는 영향X)

## 근본 해결책: 불변 객체 사용

사실 Date 자체가 문제다. Date 대신 Instant, LocalDateTime 같은 불변 타입을 쓰면 방어적 복사가 필요 없다.

```java
public final class Period {
    private final Instant start;
    private final Instant end;
    
    public Period(Instant start, Instant end) {
        if (start.isAfter(end))
            throw new IllegalArgumentException("시작이 종료보다 늦음");
        this.start = start;  // 불변이니 그대로 저장
        this.end = end;
    }
    
    public Instant start() { return start; }  // 그대로 반환
    public Instant end() { return end; }
}
```

불변 객체를 조합해서 클래스를 만들면, 방어적 복사를 고민할 일이 줄어든다. 이게 가장 좋은 방어다.