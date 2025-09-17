# 아이템 10

## **1. equals를 재정의하지 말아야 하는 경우들**

### 1.1 각 인스턴스가 본질적으로 고유한 경우

**Thread 클래스의 예시:**

```java
Thread t1 = new Thread(() -> System.out.println("Task 1"));
Thread t2 = new Thread(() -> System.out.println("Task 1")); // 같은 작업

// 같은 작업을 하지만 서로 다른 스레드 객체
System.out.println(t1.equals(t2)); // false - 이게 올바름!
```

![제목 없음.png](%EC%A0%9C%EB%AA%A9_%EC%97%86%EC%9D%8C.png)

Thread는 값이 아닌 **동작하는 개체**를 표현하므로, 같은 작업을 수행하더라도 각각은 고유한 실체

### 1.2 논리적 동치성 검사가 필요 없는 경우

**Random 클래스의 예시:**

```java
    public static void main(String[] args) {
        Random r1 = new Random(12345); // 같은 시드
        Random r2 = new Random(12345); // 같은 시드

        System.out.println(r1.nextInt());
        System.out.println(r2.nextInt());
        System.out.println(r1.equals(r2)); // false
    }
    
    1553932502
1553932502
false
```

### 1.3 상위 클래스의 equals가 적합한 경우

**Set 구현체들의 예시:**

```java
Set<String> hashSet = new HashSet<>();
Set<String> linkedHashSet = new LinkedHashSet<>();
hashSet.add("a"); hashSet.add("b");
linkedHashSet.add("a"); linkedHashSet.add("b");

// AbstractSet의 equals를 상속받아 내용물 기준으로 비교
System.out.println(hashSet.equals(linkedHashSet)); // true
```

## **2. equals를 재정의해야 하는 경우**

**2.1 값 클래스 - Integer 예시**

```java
Integer a = new Integer(100);  
Integer b = new Integer(100);

System.out.println(a == b);        // false (다른 객체) hashCode 비교
System.out.println(a.equals(b));   // true (같은 값)

public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

**2.2 커스텀 값 클래스 예시**

```java
    public static void main(String[] args) {
        Money m1 = new Money(1000, "USD");
        Money m2 = new Money(1000, "USD");

        System.out.println(m1.equals(m2)); // false - 문제!

        // Map에서 사용할 때 문제 발생
        Map<Money, String> accounts = new HashMap<>();
        accounts.put(m1, "John's account");
        System.out.println(accounts.get(m2)); // null - 찾을 수 없음!

    }
```

![제목 없음.png](%EC%A0%9C%EB%AA%A9_%EC%97%86%EC%9D%8C%201.png)

## **3. equals 메서드를 재정의하기 위한 일반 규약**

**3.1 대칭성 위배의 실제 예시**

```java
        
  public class A {

    private String name;

    public A(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj instanceof A) {
            return this.name.equals(((A) obj).name);
        }

        if (obj instanceof String) {
            return this.name.equals(obj);
        }

        return false;
    }
}

        A a = new A("a");
        String a1 = "a";

        System.out.println(a.equals(a1)); // true
        System.out.println(a1.equals(a)); // false
        
        
   해결책
    @Override
    public boolean equals(Object obj) {
        if(!(obj instanceof A))
            return false;

        if (obj instanceof A) {
            return this.name.equals(((A) obj).name);
        }

        return false;
    }
```

**3.2 추이성 위배 코드**

```java
class ColorPoint extends Point {
    private final String color;

    public ColorPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }

    // 추이성 위배하는 equals 구현
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;

        // 일반 Point라면 색상을 무시하고 비교 (대칭성을 위한 시도)
        if (!(o instanceof ColorPoint)) {
            return o.equals(this);
        }

        // ColorPoint라면 색상까지 비교
        return super.equals(o) && ((ColorPoint) o).color.equals(color);
    }

    @Override
    public int hashCode() {
        return 31 * super.hashCode() + color.hashCode();
    }

    @Override
    public String toString() {
        return String.format("ColorPoint(%d, %d, %s)", x, y, color);
    }
}

class Point {
    protected final int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }

    @Override
    public int hashCode() {
        return 31 * x + y;
    }

    @Override
    public String toString() {
        return String.format("Point(%d, %d)", x, y);
    }
}

ColorPoint colorPointA = new ColorPoint(1, 1, "a");
Point point = new Point(1, 1);
ColorPoint colorPointB = new ColorPoint(1, 1, "B");

System.out.println(colorPointA.equals(point));
System.out.println(colorPointB.equals(point));
System.out.println(colorPointA.equals(colorPointB));
        
true
true
false

해결

class ColorPoint {
    private final String color;
    private final Point point;

    public ColorPoint(int x, int y, String color) {
        point = new Point(x,y);
        this.color = color;
    }

    // 추이성 위배하는 equals 구현
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;

        // 일반 Point라면 색상을 무시하고 비교 (대칭성을 위한 시도)
        if (!(o instanceof ColorPoint)) {
            return o.equals(this);
        }

        ColorPoint o1 = (ColorPoint) o;
        // ColorPoint라면 색상까지 비교
        return o1.point.equals(this.point) && o1.color.equals(this.color);
    }

    @Override
    public int hashCode() {
        return 31 * super.hashCode() + color.hashCode();
    }

}

false
false
false
```

**일관성 (consistency)**

**Not-Null**

## 주의사항

- **equals를 재정의하면 hashCode도 반드시 재정의**해야 합니다
- **너무 복잡하게 구현하지 말고** 필드의 동치성만 검사하세요
- **Object 외의 타입을 매개변수로 받는 equals는 선언하지 마세요** (다중정의가 됨)
- **@Override 애너테이션을 일관되게 사용**하여 실수를 방지하세요