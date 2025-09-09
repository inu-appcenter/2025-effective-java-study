# item2

## 💬 생성자에 매개변수가 많다면 빌더를 고려하라

---

- 정적 팩터리와 생성자에는 똑같은 제약이 하나 있습니다.
  선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 점입니다.


- 이럴때, **점층적 생성자 패턴**을 사용해왔습니다.
    - 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개까지 받는 생성자, … 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식

    ```java
    public class NutritionFacts {
    		private final int servingSize;
    		private final int servings; 
    		private final int calories; 
    		private final int fat; 
    		private final int sodium; 
    		private final int carbohydrate; 
    		
    		public NutritionFacts（int servingSize, int servings） {
    				this（servingSize, servings, 0）;
    		}
    		public NutritionFacts（int servingSize, int servings, int calories） {
    				this（servingSize, servings, calories, 0）;
    		}
    		public NutritionFacts(int servingSize, int servings,int calories,
    													int fat) {
    				this(servingSize, servings, calories, fat, 0);
    		}
    		public NutritionFacts(int servingSize, int servings, int calories,
    													int fat, int sodium) {
    				this(servingSize, servings, calories； fat, sodium, 0);
    		}
    		public NutritionFactsdnt servingSize, int servings, int calories,
    															int fat, int sodium, int carbohydrate) {
    				this.servingSize = servingSize;
    				this.servings = servings;
    				this.calories = calories;
    				this.fat = fat;
    				this.sodium = sodium;
    				this.carbohydrate = carbohydrate;
    				}
    		}
    ```

    - 점충적 생성자 패턴도 쓸 수는 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵습니다.

- 다른 대안으로는 자바빈즈 패턴이 있습니다.
    - 매개변수가 없는 생성자로 객체를 만든 후, 세터(setter) 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식입니다.

    ```java
    public class NutritionFacts {
    		// 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    		private int servingSize = -1;
    		private int servings = -1;
    		private int calories = 0;
    		private int fat = 0;
    		private int sodium = 0;
    		private int carbohydrate = 0;
    		
    		public NutritionFacts() { }
    		// 세터 메서드들
    		public void setServingSize(int val) { servingSize = val; }
    		public void setServings(int val) { servings = val; }
    		public void setCaloriesdnt(int val) { calories = val; }
    		public void setFatdnt(int val) { fat = val; }
    		public void setSodiumdnt(int val) { sodium = val; }
    		public void setCarbohydrate(int val) { carbohydrate = val; }
    }
    ```

    - 더 읽기 쉬운 코드가 되었습니다.
    - 그러나… 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고 객체가 완성되기 전까지는 **일관성**이 무너진 상태에 놓이게 됩니다.
        - 일관성이 무너진 문제로 자바빈즈 패턴에서는 클래스를 **불변**으로 만들 수 없습니다.

- **빌더 패턴**
    - 안전성과 가독성을 겸비
    - 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻습니다.
      그 후, 빌더 객체가 제공하는 일종의 세터 메서드로 매개변수들을 설정합니다.
      마지막으로 build()를 호출해 객체를 얻습니다.

    ```java
    public class NutritionFacts {
    		private final int servingSize;
    		private final int servings;
    		private final int calories;
    		private final int fat;
    		private final int sodium;
    		private final int carbohydrate;
    		
    		public static class Builder {
    		// 필수 매개변수
    		private final int servingSize;
    		private final int servings;
    		
    		// 선택 매개변수 - 기본값으로 초기화한다.
    		private int calories = 0;
    		private int fat = 0;
    		private int sodium = 0;
    		private int carbohydrate = 0;
    		
    		public Builder(int servingSize, int servings) {
    				this.servingSize = servingSize;
    				this.servings = servings;
    		}
    		public Builder calories(int val)
    				{ calories = val; return this; }
    		public Builder fat(int val)
    				{ fat = val; return this; }
    		public Builder sodium(int val)
    				{ sodium = val; return this; }
    		public Builder carbohydratednt val)
    				{ carbohydrate = val; return this; }
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

    - 빌더의 세터 메서드들은 빌더 자신을 반환합니다.
        - 즉, 연쇄적으로 호출이 가능합니다.
        - 이런 방식을 플루언트 API / 메서드 연쇄라고 합니다.

    ```java
    NutritionFacts cocaCola = new NutritionFacts.Builder(240f 8)
    		.calories(100).sodium(35).carbohydrate(27).build();
    ```

    - 가독성이 좋으며, 빌더 패턴은 명명된 선택적 매개변수를 흉내 낸 것입니다.
    
    - 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋습니다.
    - 각 계층의 클래스에 관련 빌더를 멤버로 정의하여 사용합니다.
    
    - 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능을 공변 반환 타이핑이라고 합니다.
    - 이 기능을 이용하면 클라이언트가 형 변환에 신경 쓰지 않고도 빌더를 사용할 수 있습니다.
    → 체이닝 메서드 이용 시
        
        ```java
        // Animal.copy()는 Animal을 반환하지만 Cat.copy()는 더 구체적인 Cat을 반환
        class Animal {
            Animal copy() { return new Animal(); }
        }
        
        class Cat extends Animal {
            @Override
            Cat copy() { return new Cat(); }  // 공변 반환 타입
        }
        ```


- 빌더 패턴은 상당히 유연합니다.
    - 하지만 빌더부터 만들어야한다는 생성 비용이 있습니다.
      매개변수가 4개 이상은 되어야 값어치를 합니다.
    - 그러나 API는 시간이 지날 수록 매개변수가 많아지므로 시작부터 빌더로 잡는 것이 나을 때가 많습니다.