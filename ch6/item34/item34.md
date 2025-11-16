> ***int 상수 대신 열거 타입을 사용하라***

### 정수 열거 패턴은 취약하다

 자바에서 **열거 타입**(Enum)이 도입되기 전에는 정수와 상수를 조합해 사용하는 **정수 열거 패턴**이 일반적이었다. 하지만 이 방식은 타입 안정성을 보장하지 못하고, 비즈니스 로직의 정확성을 개발자에게 전적으로 의존해야 한다는 한계가 있었다.

<br>

 이를 보완하기 위해 **문자열 열거 패턴**을 사용하기도 했지만, 이 경우 클라이언트 코드에서 문자열 값을 직접 하드코딩해야 했다. 또한 문자열 오타가 있어도 컴파일 시점에 오류로 감지되지 않아 잠재적인 위험을 안고 가야 했다.

```java
public static final int APPLE_FUJI     = 0;
public static final int APLLE_PIPPIN   = 1;

public static final String ORANGE_NAVEL = "O_N"
public static final String ORANGE_TEMPLE = "O_T"
```

<br>


### 열거 타입(Enum)

 **Java 5**부터 **열거 타입**(Enum)이 도입되면서, 이전 정수 열거 패턴의 단점을 해결하고 이를 완벽히 대체할 수 있게 되었다.

```java
public enum Role{
    ADMIN,
    NORMAL
}
```

<br>


 **Enum**의 각 상수는 **public static final** 필드를 선언한 것처럼 동작한다. 또한 열거 타입은 외부에서 접근 가능한 생성자를 제공하지 않으므로 사실상 **final**이며, 클라이언트가 새로운 인스턴스를 생성하거나 상속할 수도 없다. 이러한 특성 덕분에 **Enum**은 통제가 가능한 전형적인 **싱글턴의 일반화된 형태**로 볼 수 있다.

<br>


 열거 타입은 **메서드**와 **필드**를 추가할 수 있으며, **인터페이스를 구현**할 수도 있다. 이는 **Enum** 역시 클래스의 한 형태이기 때문이다. 각 상수는 자체적인 값을 갖고 고유한 동작을 수행할 수 있는 구조로 되어 있다. 

<br>


 보통 클래스는 객체를 생성해 사용하는데, 열거 타입은 이러한 고정된 객체들을 상수로 정의해두는 방식이라 볼 수 있다. 아래 예시를 살펴보자.

```java
@Getter
@AllArgsConstructor
public enum MissionType {

    // 레슨 x개 완료하기
    COMPLETE_LESSON_ONE(1, "레슨 1개 완료하기", 10, 20),
    COMPLETE_LESSONS_TWO(2, "레슨 2개 완료하기", 15, 10),
    COMPLETE_LESSONS_THREE(3, "레슨 3개 완료하기", 20, 5),

    // 레슨 정답율 100%로 x개 완료하기
    PERFECT_LESSON_ONE(4, "정답율 100%로 레슨 1개 완료하기", 30, 16),
    PERFECT_LESSONS_TWO(5, "정답율 100%로 레슨 2개 완료하기", 35, 8),
    PERFECT_LESSONS_THREE(6, "정답율 100%로 레슨 3개 완료하기", 40, 6),

    // 학습 x분 완료하기
    LEARNING_MINUTES_TEN(7, "학습 10분 완료하기", 25, 15),
    LEARNING_MINUTES_FIFTEEN(8, "학습 15분 완료하기", 35, 10),
    LEARNING_MINUTES_TWENTY(9, "학습 20분 완료하기", 40, 5),

    // 새로운 친구 팔로우하기
    FOLLOW_NEW_FRIEND(10, "새로운 친구 팔로우하기", 40, 5);

    private final int missionId;
    private final String description;
    private final int awardXp;
    private final int ticket;

}
```

<br>


 각 미션은 고유한 아이디 값과 이름, 보상 경험치와 티켓을 갖는다. 현재는 클래스를 분리해 관리하고 있지만, 티켓 값에 따라 랜덤하게 미션을 생성해 클라이언트에게 반환하는 메서드 포함시킬 수도 있을 것이다.

<br>


열거 타입은 근본적으로 **불변**이라 모든 필드는 **final**이어야 하고, **private**로 각 필드를 선언한 후 **접근자 메서드**를 두는 방법을 아이템에서 추천하고 있다.

<br>


### 활용

열거 타입은 **values**라는 메서드를 제공한다. 열거 타입의 상수(문자열 아님)들을 리스트로 반환한다.

```java
public enum Type {
    DEPOSIT, WITHDRAW, TRANSFER;
}

Type[] types = Type.values()

types -> [DEPOSIT, WITHDRAW, TRANSFER]
```
<br>


> 열거 타입의 접근 수준을 **private**, **package-private**, **public** 중 어느 것으로 정의할지와 관련해서는 이전 아이템에서 이미 충분히 다뤘으므로 여기서는 생략한다.

<br>


상수마다 동작을 달리 해야하는 경우가 있다. 아래는 **switch-case**를 활용한 예시이다.

```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;
    
    public double apply(double x, double y) {
        switch(this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```

<br>


 이 방식의 단점이라면, 상수가 하나 추가될 때마다 **apply** 메서드의 **case** 문도 함께 수정해야 한다는 점이다. (물론 단점이라고는 하지만, 개인적으로는 이후에 소개할 방식과 비교했을 때 큰 차이는 느껴지지 않았다.) 

<br>


이러한 불편함을 줄이기 위해 열거 타입 내부에 **추상 메서드를 선언**하고, **각 상수에서 이를 구현**하는 방식을 사용할 수도 있다.

```java
public enum Operation {
    PLUS {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE {
        public double apply(double x, double y) {
            return x / y;
        }
    };
    
    public abstract double apply(double x, double y);
}
```

<br>

열거 타입에서는 **상수의 이름을 입력받아 해당 상수를 반환**해주는 **valueOf** 메서드가 제공된다. 그리고 열거 타입의 **toString** 메서드를 재정의하거든, **toString이 반환하는 문자열의 상수 타입을 반환**하는 **fromString** 메서드도 함께 제공하는 것을 고려하자.

<br>


**fromString** 결의 메서드는 실제 개발에서 자주 사용된다. 예를 들어 **enum validation** 과정에서 입력된 문자열이 해당 열거 타입의 상수로 존재하는지 판단할 때나, 단순히 문자열을 특정 열거 타입의 상수로 변환해야 할 때도 빈번하게 사용되는 것 같다.

```java
@JsonCreator
public static ProblemType from(String type) {
    return Arrays.stream(values())
            .filter(t -> t.type.equals(type))
            .findFirst()
            .orElseThrow(() -> new RestApiException(CustomErrorCode.PROBLEM_TYPE_INVALID))
}
```

<br>


아이템에서 소개하는 방식은 **stringToEnum**이라는 불변 **Map**을 두고 **fromString** 메서드에서는 입력된 문자열에 해당하는 상수를 반환하는 구조로 설계되어 있다. 기존 방식에서는 **from** 메서드가 호출될 때마다 배열을 생성해야 해서 비용적으로 비효율적이지만, 이 방법은 상수 Map을 통해 메모리 측면에서 효율적이다. 

```java
private static final Map<String, ProblemType> TYPE_MAP = 
    Arrays.stream(values())
        .collect(Collectors.toMap(t -> t.type, t -> t));

@JsonCreator
public static ProblemType from(String type) {
    ProblemType result = TYPE_MAP.get(type);
    if (result == null) {
        throw new RestApiException(CustomErrorCode.PROBLEM_TYPE_INVALID);
    }
    return result;
}
```

<br>


당연하게도 아래 방식은 안된다.

```java
public enum Operation {
    PLUS("+"), MINUS("-");
    
    private static final Map<String, Operation> MAP = new HashMap<>();
    private final String symbol;
    
    Operation(String symbol) {
        this.symbol = symbol;
        MAP.put(symbol, this);  // ❌ 컴파일 에러: static 필드 접근 불가
    }
}
```

<br>


### 한계 극복

상수별 메서드를 구현에는 열거 타입 상수끼리 코드를 공유, 즉 재사용하기 어렵다는 단점이 있다. 아래는 안티패턴의 예시이다.

```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        int overtimePay;
        
        switch(this) {
            case SATURDAY: 
            case SUNDAY:  // 주말
                overtimePay = basePay / 2;
                break;
            default:  // 주중
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 
                    0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```

<br>


만약, 새로운 상수가 추가되는 경우 이에 따라 **pay** 메서드의 **switch-case**도 수정이 발생해야 한다. 하지만, 개발자가 깜빡하고 놓쳐도 문제를 인지할 방법이 전혀 없다.

<br>


따라서, 아래와 같은 **전략 열거 타입 패턴**을 제시한다.

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);
    
    private final PayType payType;
    
    PayrollDay(PayType payType) { 
        this.payType = payType; 
    }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                    (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };
        
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

<br>


 중첩 열거 타입에서 외부 열거 타입의 특징과 관련 메서드를 정의해두면, 새로운 **PayrollDay** 상수가 추가되었을 때 **PayType**만 선택해서 정의해두면 전혀 문제 없이 더 안전하고 유연하게 활용할 수 있다.

<br>


아이템에서는 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 **switch-case**가 더 적합하다고 한다. 언제는 **전략 열거 타입 패턴**이, 언제는 **switch-case**가 더 유용한지 헷갈려서 아래처럼 정리하였다.

<br>


**상황별 쓰임**

- **외부 라이브러리**(직접 수정이 불가능한), **부가 기능** 로직 → **`switch-case`**
- **열거 타입의 핵심 로직**(직접 수정이 가능한) → **`전략 패턴`** or **`상수별 메서드`**

<br>


### 언제

**컴파일 시간에 모든 가능한 값을 알 수 있는 상수 집합**이라면 열거 타입을 적극활용하자. **본질적으로 열거 타입**(태양계 행성)이거나 **허용되는 값이 제한적이고 미리 알 수 있는 경우**(결제 타입, 연산자)에 좋다
