# 아이템 35

# 아이템 35: ordinal() 메서드 대신 인스턴스 필드를 사용하라

## ordinal() : Enum의 상수가 선언된 순서를 0부터 증가시키면서 반환

## 나쁜 예시

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() {
        return ordinal() + 1;
    }
}
```

위 코드를 순서대로 numberofMusicions가 SOLO는 1, DUET은 2 이렇게 순서대로 올라가면서 리턴됨

### 문제점

1. Enum의 상수 순서 변경 시
    
    만약 Ensemble에서 SOLO, DUET을 실수로 순서를 바꿔서 넣으면 SOLO는 2, DUET은 1로 리턴됨
    
2. 중복된 값 불가
    
    예를 들어 8명이서 연주하는 DOUBLE_QUARTET를 저장할려고 할 때 이미 OCTET이 있어서 8을 리턴하는게 불가능
    
3. 값 건너뛰기 불가
    
    아무것도 Ensemble의 상수가 없고 12명을 의미하는 TRIPLE_QUARTET를 추가할려면 의미 없는 상수 11개를 억지로 넣어야함
    
4. 코드 유지보수성에 별로
    
    새로운 상수 추가, 순서를 바꾸면 기존 상수 코드들에 영향
    

## 좋은 예시

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Ensemble(int size) {
        this.numberOfMusicians = size;
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```

1. 명확, 유지보수 용이
    
    순서 변경, 중복 값(위의 12가 예시) 등 유연함
    
2. 의미가 코드에 보임
    
    OCTET은 8을 의미하는게 바로 보임
    

## 결론

```java
대부분의 프로그래머는 이 메서드를 쓸 일이 없다.
이 메서드는 EnumSet, EnumMap 같은 열거 타입 기반의 범용 자료구조에서
내부적으로 사용할 목적으로 설계되었다.
```

ordinal은 라이브러리 내부에서 index를 관리할 때만 사용

비즈니스 로직에서는 절대 사용x