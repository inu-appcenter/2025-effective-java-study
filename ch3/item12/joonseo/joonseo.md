# Item12

> ***toString을 항상 재정의하라.***

 <br>

## toString

 toString 메서드는 재정의하지 않으면, 클래스 이름과 16진수 해시코드를 반환한다. toString 일반 규약에 따르면 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다고 한다. 그리고 모든 하위 클래스에서 이 메서드를 재정의하라고 명시하고 있다.

  <br>

toString 메서드는 클래스 인스턴스를 출력할 때, 자동으로 호출된다. 아래처럼 말이다.

 <br>

**Test**

```java
public class Test {
    public static void main(String[] args) {
        InnerClass class1 = new InnerClass("Joonseo", 22);

        System.out.println(class1);
        System.out.println(class1.toString());

    }

    public static class InnerClass {

        public String name;
        public Integer age;

        public InnerClass(String name, Integer age) {
            this.name = name;
            this.age = age;
        }
    }
}
```

 <br>

**출력**

```java
Test$InnerClass@5acf9800
Test$InnerClass@5acf9800
```

 <br>

 실전에서 **toString**은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다고 한다. 나의 생각은 살짝 달랐던 게, 어디서든 호출될 수 있는 **public** 메서드에 객체가 갖고 있는 모든 정보를 반환하는게 안전할까? 라는 생각도 들었다.

그치만, 지금까지 문제 없이 **deprecated**되지 않는 걸 보면 안전한 것 같다.

 <br>
