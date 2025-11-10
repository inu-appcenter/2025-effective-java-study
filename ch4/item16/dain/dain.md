<aside>

public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

</aside>

public 클래스에서 필드를 직접 노출하면 캡슐화의 이점을 잃는다.

1. API 변경 없이 내부 표현을 바꿀 수 없고
2. 불변식을 보장하기 어렵고
3. 필드 접근 시 부가 작업을 수행할 수 없다

### 잘못된 예시(퇴보한… 클래스)

```java
// 나쁜 예시 - 캡슐화를 위반한 클래스
class BadPoint {
    public double x;
    public double y;
    
    public static void main(String[] args) {
        BadPoint point = new BadPoint();
        point.x = 10;  // 직접 접근
        point.y = 20;  // 직접 접근
        
        // 문제점들:
        point.x = -1000; // 음수 좌표도 설정 가능 (검증 불가)
        point.y = Double.NaN; // 비정상 값도 설정 가능
        
        System.out.println("x: " + point.x + ", y: " + point.y);
        
        // 내부 구조를 바꾸고 싶어도 클라이언트 코드가 깨짐
        // 예를 들어 극좌표로 바꾸려면 모든 클라이언트를 수정해야 함
    }
}
```

### 올바른 예시

```java
public class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    // 접근자 (getter)
    public double getX() { return x; }
    public double getY() { return y; }
    
    // 변경자 (setter) - 검증 로직 포함 가능
    public void setX(double x) {
        if (Double.isNaN(x)) {
            throw new IllegalArgumentException("x는 NaN일 수 없습니다");
        }
        this.x = x;
    }
    
    public void setY(double y) {
        if (Double.isNaN(y)) {
            throw new IllegalArgumentException("y는 NaN일 수 없습니다");
        }
        this.y = y;
    }
    
    // 부가 기능도 제공 가능
    public double distanceFromOrigin() {
        return Math.sqrt(x * x + y * y);
    }
    
    // 필드 접근 시 로깅이나 캐싱 등의 부수 작업 가능
    private int accessCount = 0;
    public double getXWithLogging() {
        accessCount++;
        System.out.println("x 접근 횟수: " + accessCount);
        return x;
    }
    
    public static void main(String[] args) {
        Point point = new Point(3, 4);
        
        System.out.println("x: " + point.getX()); // 안전한 접근
        System.out.println("y: " + point.getY());
        System.out.println("원점으로부터 거리: " + point.distanceFromOrigin());
        
        point.setX(5); // 검증된 변경
        // point.setX(Double.NaN); // 예외 발생으로 보호됨
        
        point.getXWithLogging(); // 부수 작업 수행
    }
}
```

### 내부 표현 변경의 유연성

접근자를 사용하면 내부 구현을 자유롭게 변경할 수 있다

```java
public class FlexiblePoint {
    // 내부적으로 극좌표로 저장 (클라이언트는 모름)
    private double radius;
    private double angle;
    
    public FlexiblePoint(double x, double y) {
        this.radius = Math.sqrt(x * x + y * y);
        this.angle = Math.atan2(y, x);
    }
    
    // 클라이언트는 여전히 직교좌표로 사용
    public double getX() {
        return radius * Math.cos(angle);
    }
    
    public double getY() {
        return radius * Math.sin(angle);
    }
    
    public void setX(double x) {
        double currentY = getY();
        this.radius = Math.sqrt(x * x + currentY * currentY);
        this.angle = Math.atan2(currentY, x);
    }
    
    public void setY(double y) {
        double currentX = getX();
        this.radius = Math.sqrt(currentX * currentX + y * y);
        this.angle = Math.atan2(y, currentX);
    }
    
    // 극좌표의 장점도 활용 가능
    public double getRadius() { return radius; }
    public double getAngle() { return angle; }
    
    public void rotate(double deltaAngle) {
        this.angle += deltaAngle;
    }
    
    public static void main(String[] args) {
        FlexiblePoint point = new FlexiblePoint(3, 4);
        
        System.out.printf("직교좌표: (%.1f, %.1f)%n", point.getX(), point.getY());
        System.out.printf("극좌표: 반지름=%.1f, 각도=%.2f라디안%n", 
                         point.getRadius(), point.getAngle());
        
        point.rotate(Math.PI / 4); // 45도 회전
        System.out.printf("회전 후: (%.1f, %.1f)%n", point.getX(), point.getY());
        
        // 클라이언트 코드는 내부 구현(극좌표)을 전혀 몰라도 됨
    }
}
```

### package-private와 private 중첩 클래스는 예외

package-private이나 private 중첩 클래스는 필드를 직접 노출해도 문제없다.

```java
public class GeometryCalculator {
    
    // private 중첩 클래스 - 필드 노출 OK
    private static class Vector {
        double x, y; // public 접근자 불필요
        
        Vector(double x, double y) {
            this.x = x;
            this.y = y;
        }
        
        double magnitude() {
            return Math.sqrt(x * x + y * y);
        }
    }
    
    public double calculateDistance(double x1, double y1, double x2, double y2) {
        Vector v = new Vector(x2 - x1, y2 - y1);
        return v.magnitude(); // v.x, v.y 직접 접근도 가능
    }
    
    public static void main(String[] args) {
        GeometryCalculator calc = new GeometryCalculator();
        double distance = calc.calculateDistance(0, 0, 3, 4);
        System.out.println("거리: " + distance); // 5.0
    }
}

// package-private 클래스도 마찬가지
class InternalCoordinate {
    double x, y; // 같은 패키지에서만 사용되므로 직접 노출 OK
    
    InternalCoordinate(double x, double y) {
        this.x = x;
        this.y = y;
    }
}
```

### 불변 필드라도 노출하지 말 것

```java
// 불변 필드를 노출한 예시
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    
    public final int hour;   // 불변이지만 노출
    public final int minute; // 불변이지만 노출
    
    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) {
            throw new IllegalArgumentException("시간: " + hour);
        }
        if (minute < 0 || minute >= MINUTES_PER_HOUR) {
            throw new IllegalArgumentException("분: " + minute);
        }
        this.hour = hour;
        this.minute = minute;
    }
    
    // 문제점: API 변경 없이 표현 방식을 바꿀 수 없음
    // 예를 들어, 내부적으로 분 단위로만 저장하고 싶어도 불가능
    
    public static void main(String[] args) {
        Time time = new Time(14, 30);
        System.out.println(time.hour + "시 " + time.minute + "분");
        
        // 클라이언트가 필드에 직접 의존하므로 내부 구현 변경 어려움
    }
}

// 더 나은 방식 - 접근자 사용
public final class BetterTime {
    private final int totalMinutes; // 내부적으로는 분 단위로 저장
    
    public BetterTime(int hour, int minute) {
        if (hour < 0 || hour >= 24) {
            throw new IllegalArgumentException("시간: " + hour);
        }
        if (minute < 0 || minute >= 60) {
            throw new IllegalArgumentException("분: " + minute);
        }
        this.totalMinutes = hour * 60 + minute;
    }
    
    public int getHour() {
        return totalMinutes / 60;
    }
    
    public int getMinute() {
        return totalMinutes % 60;
    }
    
    // 추가 기능도 쉽게 제공
    public int getTotalMinutes() {
        return totalMinutes;
    }
    
    public BetterTime addMinutes(int minutes) {
        int newTotal = (totalMinutes + minutes) % (24 * 60);
        return new BetterTime(newTotal / 60, newTotal % 60);
    }
    
    public static void main(String[] args) {
        BetterTime time = new BetterTime(14, 30);
        System.out.println(time.getHour() + "시 " + time.getMinute() + "분");
        
        BetterTime later = time.addMinutes(45);
        System.out.println("45분 후: " + later.getHour() + "시 " + later.getMinute() + "분");
        
        System.out.println("총 분: " + time.getTotalMinutes() + "분");
    }
}
```

### 정리

- **public 클래스**: 절대 가변 필드 직접 노출 금지, 불변 필드도 가급적 노출 안 함
- **접근자 메서드**: 캡슐화 제공, 내부 구현 변경 유연성, 부가 기능 수행 가능
- **예외 상황**: package-private 클래스나 private 중첩 클래스는 필드 노출 허용
- **검증과 로깅**: setter에서 데이터 검증, getter에서 부수 작업 수행 가능