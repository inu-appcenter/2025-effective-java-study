# Item23

> ***태그 달린 클래스보다는 클래스 계층구조를 활용하라***

<br>

## 주관식과 객관식

 현재 프로젝트에서는 주관식과 객관식을 하나의 **Problem** 도메인으로 통합하고, **ProblemType**이라는 **enum**을 통해 유형을 구분하고 있다. 공통 필드가 많고 각 문제 유형의 로직 차이가 크지 않아 당시에는 적합한 선택이라고 생각하였지만, 코드를 다시 볼 때마다 ‘이게 정말 최선이었을까?’라는 생각이 든다.

 사실 추상 클래스를 활용해 구조적으로 분리하는 방법도 고려했지만, **JPA 상속 매핑**의 복잡성과 예측 불가능성을 우려해 단순하게 enum으로 구분하는 방식을 사용하기로 했다.

<br>

## 분리가 필요한

 하지만, 아이템에서의 예시와 같이 공통되는 필드가 적고, 특정 구분을 위해 존재하는 프로퍼티 때문에 불필요한 코드가 많아진다면 반드시 분리를 고려해봐야 한다. 아래 예시를 보자

<br>

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE }
    
    final Shape shape;
    
    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;
    
    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;
    
    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }
    
    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    
    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

<br>

 데이터베이스 관점에서는 정규화를 잘못한 코드로 보여진다. 또한, 특정 사용에 의미없는 필드를 초기화하는 오버헤드가 존재하고 메모리적으로도 성능적으로도 좋지 않은 코드이다. 

 추상 클래스를 통해 공통 관심사를 분리하고 각 도메인의 특성에 맞게 재정의하자. 바로 아래처럼 말이다.

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;
    
    Circle(double radius) { 
        this.radius = radius; 
    }
    
    @Override 
    double area() { 
        return Math.PI * (radius * radius); 
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;
    
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override 
    double area() { 
        return length * width; 
    }
}
```
