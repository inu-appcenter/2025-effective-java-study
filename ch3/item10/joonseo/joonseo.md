# Item10

> ***equals는 일반 규약을 지켜 재정의하라***

 <br>

## equals 재정의

 equals 메서드는 재정의가 쉽지만, 규약에 맞춰 재정의하지 않으면 의도대로 동작하지 않을 수 있다. 아이탬 10에서는 equals를 재정의하면 안되는 조건을 다음과 같이 명시하고 있다.

 <br>

- 각 인스턴스가 본질적으로 고유할 때
- 인스턴스의 논리적 동치성을 검사할 일이 없을 때
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 적용될 때
- 클래스가 private이거나, package-private이고 equals를 호출할 일이 없을 때

<br>

그럼 언제 사용해야 할까?

 두 인스턴스가 **물리적**(메모리 상 위치하는 공간)으로 다르지만, **논리적**(색연필 2세트를 샀는데, 논리적으로 각각 안에 들어있는 빨간색 색연필은 같다)으로 같을 때, 근데 **equals 메서드가 물리적 동치성을 비교하는 경우**이다. 

 <br>

 그럼, 논리적 동치성을 비교할 수 있도록 equals 메서드를 재정의해줘야 하는데 의도대로 동작하게 하려면 **아래 5개의 성질을 반드시 만족**해야 한다.

 <br>

## equals 일반 규약

### 반사성

> **null이 아닌 모든 참조값 x에 대해 x.equals(x)는 true다.**

 <br>

**반사성**은 쉽다. 이 조건은 아이탬에서도 말하듯 못 지키기가 더 어렵다.

<br>

### 대칭성

> **null이 아닌 모든 참조값 x, y에 대해 x.equals(y)가 true면, y.equals(x)도 true다.**

 <br>

 **대칭성**은 **두 객체 서로의 동치 여부가 동일**해야 한다는 성질이다. 이것도 반사성과 같이 못지키기가 더 어려워 보였지만, 아래 예시를 보면 생각보다 주의해야할 점이 있다. 

 <br>

```java
public final class CaseInsensitiveString {
    private final String s;
    
    public CaseInsensitiveString(String s) {
        if (s == null) 
            throw new NullPointerException();
        this.s = s;
    }
    
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String) // 한 방향으로만 작동함
            return s.equalsIgnoreCase((String) o);
        return false;
    }
}
```

<br>

 이렇게 되면, 같은 문자열을 갖고 있는 String 과 CaseInsensitiveString의 equals 메소드가 서로 다른 결과를 반환한다.

 <br>

```java
    System.out.println(cis.equals(s));  // true
    System.out.println(s.equals(cis));  // false
```

<br>

### 추이성

> **null이 아닌 모든 참조값 x, y, z에 대해 x.equals(y), y.equals(z)가 true면, x.equals(z)도 true다.**

 <br>

 추이성은 A = B이고 B = C이면, A = C를 만족해야 한다는 규칙인데, 아이탬에서 제공하는 추이성을 위배하는 경우를 살펴보자.

 <br>

**Point**

```java
public class Point {
    private final int x, y;
    
    public Point(int x, int y) {
        this.x = x; this.y = y;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

<br>

**ColorPoint**

```java
public class ColorPoint extends Point {
    private final Color color;
    
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        
        if (!(o instanceof ColorPoint))
            return o.equals(this);
           
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```

<br>

 **ColorPoint** 클래스의 **equals** 메서드는 비교 인스턴스가 **Point**라면 색상은 무시하고 비교하며, **ColorPoint**에 대해서만 위치, 색상까지 비교한다.

 <br>

이렇게 메소드를 구성하면 아래와 같은 테스트 케이스에 대해 통과할 수 없다.

<br>

```java
    ColorPoint redPoint = new ColorPoint(1, 2, Color.RED);
    Point point = new Point(1, 2);
    ColorPoint bluePoint = new ColorPoint(1, 2, Color.BLUE);
    
    System.out.println(redPoint.equals(point));    // true
    System.out.println(point.equals(bluePoint));   // true  
    System.out.println(redPoint.equals(bluePoint)); // false
```

<br>

 이러한 추이성 위배 문제는 객체지향 언어의 동치관계에서 나타나는 근본적인 문제로, 구체 클래스를 확장해 새로운 값을 추가하게 되면, equals 일반 규약을 만족할 방법이 없다.

 <br>

 아이탬에서는 타입비교를 **instanceOf** 대신 **getClass**를 사용하는 방법을 제시하고 있지만, 이는 리스코프 치환 원칙(하위타입 → 상위타입)을 위배하며 다형성에 제한을 둔다.

 <br>

 아이탬의 설명을 인용하면, ***Point의 하위 클래스는 정의상 여전히 Point이므로, 어디서든 Point로 활용될 수 있어야 한다*** 지만, getClass로 타입에 제한을 둔다면 이는 지켜질 수 없다.

 <br>

```java
public class ColorPoint {
    private final Point point;
    private final Color color;
    
    public ColorPoint(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }
    
    /**
     * ColorPoint의 Point 뷰 반환
     */
    public Point asPoint() {
        return point;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
		}
}
```

<br>

 대안으로, 위 코드와 같이 상속 대신 **컴포지션** 방식으로 구현하여 **View** 메서드를 구현해 **Point**로의 활용 가능성을 열어두고, 각 필드 타입 클래스가 정의한 **equals** 메서드를 통해 대칭성, 추이성, 일관성을 만족하게 구현할 수 있다.

 <br>

### 일관성

> **null이 아닌 모든 참조값 x, y에 대해 x.equals(y)의 결과는 항상 같은 값을 반환한다.**

<br>

 일관성은 두 객체를 비교하는 행위에서 **true**, **false**를 반환했다면, **앞으로도 같은 값을 반환**해야 한다는 규칙이다. 클래스가 불변이면 이 일관성을 반드시 만족해야 한다. 

 <br>

 또한, 클래스가 가변/불변에 상관없이 **equals** 구현에 **신뢰할 수 없는 자원이 끼어들게 해서는 안된다.** 아래 예시를 보자

 <br>

```java
public class Stock {
    private final String symbol;
    private final String exchange;
    
    public Stock(String symbol, String exchange) {
        this.symbol = symbol;
        this.exchange = exchange;
    }
   
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Stock)) return false;
        
        Stock other = (Stock) o;
      

        double thisPrice = StockPriceAPI.getCurrentPrice(this.symbol);
        double otherPrice = StockPriceAPI.getCurrentPrice(other.symbol);
        
        return this.symbol.equals(other.symbol) && 
               this.exchange.equals(other.exchange) &&
               Math.abs(thisPrice - otherPrice) < 0.01;
    
    }
}
```

<br>

 **Stock** 클래스는 **equals** 메서드에서 실시간으로 변할 가능성이 있는 주가를 가져와 비교를 진행한다. 이렇게 동적으로 변할 가능성이 있는 필드를 비교하는 행위는 의도한대로 결과를 반환하지 못할 가능성이 크다.

 <br>

### null-아님

> **null이 아닌 모든 참조값 x에 대해 x.equals(null)은 항상 false다.**

 <br>

**null-아님** 규약은 **모든 객체는 null과 같지 않아야 한다는 뜻**이다. 많은 코드가 아래처럼 **null** 타입인지를 체크하지만, 두번째 코드처럼 조금 더 포괄적인 맥락에서 검사하는 것이 더 좋다.

<br>

```java
// 명시적 null 검사 - 필요 없다!
@Override 
public boolean equals(Object o) {
    if (o == null)
        return false;
    // ... 나머지 코드
}

// 묵시적 null 검사 - 이쪽이 낫다.
@Override 
public boolean equals(Object o) {
    if (!(o instanceof MyType))
        return false;
    MyType mt = (MyType) o;
    // ... 필드 비교
}
```

<br>

<aside>

1️⃣ **equals 구현 팁(1) - == 연산자를 통해 자기 자신의 참조인지 확인하라**

---

입력으로 들어온 인스턴스가 **자기 자신의 참조인지 확인**하여, 불필요한 필드 검사를 하지 않도록 설계하자.

</aside>

<br>

<aside>

2️⃣ **equals 구현 팁(2) - 먼저 비교할 필드를 선택하라**

---

‘**아닐 가능성이 가장 큰**’ 필드를 먼저 검사하여 불필요한 필드 검사를 제거하자. 예시로 **그 인스턴스의 고유의 값**(PK, 주민등록번호 등)을 먼저 비교하면 결과를 더 빠르게 반환할 수 있을 것이다.

</aside>

<br>

## equals 구현 주의 사항

### equals를 재정의 할 때는 hashCode도 반드시 재정의하자

이건 아이탬 11에서 다룰 예정이다.

<br>

### 너무 복잡하게 해결하려 들지 말자

필드의 동치성을 검사하는 것만으로도 equals 규약을 어렵지 않게 지킬 수 있다. 예컨데 별칭이라든지 논리적 동치성을 판단하는데 너무 딥한 요소는 제거하고 구현하도록 하자.

<br>

### 매개변수의 타입은 Object이다

아래 예시 코드는 equals를 오버로딩한 것이지, 오버라이딩한 것이 아니다.

<br>

```java
public boolean equals(MyClass o){
}
```

<br>

이렇게 구체적으로 타입을 명시한 **equals** 메소드는 **@Override** 어노테이션이 긍정 오류를 내게 하고 보안적인 측면에서도 잘못된 정보를 주기 때문이다.
