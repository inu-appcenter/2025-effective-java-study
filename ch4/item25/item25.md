<aside>

톱레벨 클래스는 한 파일에 하나만 담으라

</aside>

소스 파일 하나에 톱레벨 클래스를 여러 개 선언해도 자바 컴파일러는 에러를 내지 않는다. 하지만 이건 득은 없고 위험만 감수하는 행위다. **어느 소스 파일을 먼저 컴파일하느냐에 따라 결과가 달라지기 때문이다.**

(**톱레벨 클래스**: 파일에 정의되어 있는 가장 바깥에 있는 클래스)

```java
// Main.java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

```java
// Utensil.java 
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

여기까지는 문제가 없다. 다만 이후에 같은 클래스를 다시 한 번 정의한다면?

```java
// Dessert.java 
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

이제 같은 클래스가 두 파일에 중복 정의된 상태다. 컴파일 명령어에 따라 결과가 달라진다.

### 해결 방법1: 파일 분리

가장 간단하고 명확한 해결책은 **톱레벨 클래스를 각각 별도 파일로 분리**하는 것이다.

```java
// Utensil.java
class Utensil {
    static final String NAME = "pan";
}

// Dessert.java
class Dessert {
    static final String NAME = "cake";
}

// Main.java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

### 해결 방법2: 정적 멤버 클래스

여러 클래스를 꼭 한 파일에 담고 싶은 경우… 다른 클래스에 종속적인 부차적인 클래스라면 **정적 멤버 클래스**로 만들자.

```java
// Main.java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    
    private static class Utensil {
        static final String NAME = "pan";
    }
    
    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

**장점**:

1. **가독성**: 관련 클래스들이 명확히 그룹화됨
2. **캡슐화**: `private`으로 선언하면 접근 범위를 최소화할 수 있음
3. **안정성**: 컴파일 순서에 영향받지 않음

단, Utensil과 Dessert가 Main에 **종속적인 보조 클래스일 때** 적합하다. 만약 독립적으로 사용될 클래스라면, 그냥 별도 파일로 분리하는 게 맞다.

### **정리**

- **소스 파일 하나에는 톱레벨 클래스를 하나만 담자**
- 정말 여러 클래스를 한 파일에 담아야 한다면, 정적 멤버 클래스를 고려하되 가능하면 분리하자