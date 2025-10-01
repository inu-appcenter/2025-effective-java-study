# Item16

> ***public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라***

<br>

## 캡슐화의 이점

<br>

 **캡슐화**의 이점은 데이터의 무결성을 보장하고 스레드 안전성을 보장함에 큰 의미가 있다. 이번 아이탬에서는 필드를 **private**로 제한하는 대신, 특정 필드에 대한 작업을 수행하는 메서드를 정의하여 사용하는 방식을 제안하고 있다. 

<br>

```java
// 기존 Point
public class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { 
        return x; 
    }
    
    public double getY() { 
        return y; 
    }
    
    public void setX(double x) { 
        this.x = x; 
    }
    
    public void setY(double y) { 
        this.y = y; 
    }
}
```

<br>

 관례적으로 `setOOO`, `getOOO` 네이밍으로 필드에 대한 업데이트, 조회를 수행하는 메서드를 작성한다. 이러한 메서드를 통해 클래스 내부 표현 방식을 언제든지 바꿀 수 있는 유연성을 얻을 수 있다.

<br>

*내부 표현 방식을 언제든지 바꿀 수 있는 유연성* 에 대해 처음에는 이해하지 못했는데, 위 Point 코드와 바뀐 Point 코드를 보며 이해해보았다. 

<br>

```java
// 바뀐 Point
public class Point {
    private double r;      // 극좌표계로 변경
    private double theta;  // 각도
    
    public Point(double x, double y) {
        this.r = Math.sqrt(x * x + y * y);
        this.theta = Math.atan2(y, x);
    }
    
    public double getX() { 
        return r * Math.cos(theta); 
    }
    
    public double getY() { 
        return r * Math.sin(theta);
    }
    
    public double getDistance() {
        return r;
    }
    
    public double getAngle() {
        return theta
    }
}
```

<br>

 만약, 인스턴스 필드가 **public**이고 클라이언트가 이를 직접 접근하여 사용했다면, 기존 클라이언트 코드는 컴파일 조차되지 않아 수정이 필요했을 것이다. 하지만, 필드가 **private**이고 접근 제어자를 통해 필드값을 활용했다면, 클라이언트 코드 수정이 필요 없었을 것이다. 

<br>

```java
// 초기 구현
public class Point {
    public double x;  // public 필드
    public double y;
}

// 클라이언트에서 직접 사용
Point p = new Point();
p.x = 3;  // 직접 접근
p.y = 4;
double distance = Math.sqrt(p.x * p.x + p.y * p.y);

// 극좌표계로 변경하려면?
public class Point {
    public double r;      
    public double theta; 
}

// 모든 클라이언트 코드가 망가짐!
// p.x = 3;  // 컴파일 에러! x 필드가 없음
// p.y = 4;  // 컴파일 에러! y 필드가 없음
```

<br>

하지만, package-private 클래스나 private 중첨 클래스라면 데이터 필드를 노출해도 영향 범위가 제한적이기 때문에 크게 영향을 주지 않는다.
