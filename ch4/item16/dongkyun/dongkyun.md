# item16

## 🦮 public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

---

⚠️ **문제: public 필드 직접 노출**

```java
class Point {
    public double x;
    public double y;
}
```

✅ 해결 : **접근자 메서드 활용**

```java
public class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

❌ **예외 : package-private 또는 private 중첩 클래스**

```java
class InternalPoint {
    double x; // 직접 노출해도 괜찮음
    double y;
}
```

- 패키지 외부에서는 사용 불가 → 캡슐화 영향 없음

✚ **불변 필드(public final)도 주의**

```java
public final class Time {
    public final int hour;
    public final int minute;

    public Time(int hour, int minute) { ... }
}
```

- 불변 필드라면 직접 노출해도 안전성 일부 확보 가능
- 하지만 **표현 변경 유연성 없음** → 접근자 메서드가 더 안전