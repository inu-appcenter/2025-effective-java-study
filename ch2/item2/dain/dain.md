# Item2

<aside>

생성자에 매개변수가 많다면 빌더를 고려하라

</aside>

매개변수가 많은 생성자와 정적 팩터리 메서드의 한계에 대한 해결책으로 **빌더 패턴**을 권장한다.

1. 여러 개의 매개변수를 받는 생성자
2. 여러 개 오버로딩
3. 선택적 매개변수 多

**→ 코드가 점점 길어지고 읽기 어렵다.**

```java
public class NutritionFacts {
    private final int servingSize;   // 필수
    private final int servings;      // 필수
    private final int calories;      // 선택
    private final int fat;           // 선택
    private final int sodium;        // 선택
    private final int carbohydrate;  // 선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }
    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat,
                         int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat,
                         int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}

```

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

위처럼… 가독성이 떨어지고 어떤 매개변수가 무엇인지 알 수 없다.

그렇다면 자바빈즈 패턴으로 변경해보자. 순서가 자유롭고, 가독성도 개선될 것이다.

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

이번엔 또다른 문제가 발생한다. 메서드를 여러 개 호출해야 하고, 객체가 완성되기 전의 중간 상태는 일관성이 무너져 있다. → 불변으로 만들 수도 없다

<aside>

DB접근이 없다면 문제 없지 않나

static일 때만… 그럼 static final 하고 set 막으면

스레드/메서드 영역?

</aside>

위 두 방식의 장점만을 취한 것이 빌더 패턴이다.

```java
public class NutritionFacts {
    private final int servingSize;   // 필수
    private final int servings;      // 필수
    private final int calories;      // 선택
    private final int fat;           // 선택
    private final int sodium;        // 선택
    private final int carbohydrate;  // 선택

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수(기본값 설정)
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) { calories = val; return this; }
        public Builder fat(int val) { fat = val; return this; }
        public Builder sodium(int val) { sodium = val; return this; }
        public Builder carbohydrate(int val) { carbohydrate = val; return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)    .calories(100).sodium(35).carbohydrate(27).build();
```

이제 매개변수의 의미가 메서드 이름에 드러나고, 불변 객체를 생성할 수 있으며, 선택 매개변수가 많아져도 필수만 빌더 생성자에 두니 훨씬 깔끔하다.

⇒ 매개변수의 개수가 많아질수록 빌더 패턴으로의 전환을 권장한다. 하지만 초기에 많은 매개변수를 갖지 않더라도 추후 늘어날 가능성을 고려하면 처음부터 빌더 패턴을 사용하는 것이 좋겠다.