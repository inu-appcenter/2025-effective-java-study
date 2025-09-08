# 아이템 2

## 생성자에 매개변수가 많다면 빌더를 고려하라

### 점층적 생성자 패턴

```java
public NutritionFacts(int servingSize, int servings) { ... }
    public NutritionFacts(int servingSize, int servings, int calories) { ... }
    public NutritionFacts(int servingSize, int servings, int calories, int fat) { ... }
```

**단점**:

- 매개변수 개수가 많아지면 코드 작성/읽기 어려움
- 각 값의 의미가 헷갈림
- 매개변수 순서를 바꿔도 컴파일러가 감지 못함(어차피 타입이 같으면 컴파일 오류가 아니라 로직적인 오류이기 때문)

### 자바빈즈 패턴

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
```

**단점**:

- **일관성 깨짐**: 객체가 완전히 생성되기 전까지 일관성 없는 상태
- **불변 클래스 불가능**: setter 메서드로 인해 불변 객체 만들 수 없음

### 상속에서의 Builder

```java
@Entity
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Parent {

    @Id @GeneratedValue
    private Long id;

    private String parentName;
    private int parentAge;
}

---------

@Entity
@NoArgsConstructor
public class Child extends Parent {

    private String childName;
    private int childAge;

    @Builder
    public Child(Long id, String parentName, int parentAge, String childName, int childAge) {
        super(id, parentName, parentAge);-> 이거 필요
        this.childName = childName;
        this.childAge = childAge;
    } 
}

---------

public class TestService {

    public void test() {
        Child.builder()
                .parentName("parent")
                .parentAge(10)
                .childName("child")
                .childAge(20);
    }
}

```

- 빌더가 부모 생성자가 필요한 이유
    
    .build()는 생성자가 호출된다는 의미
    
    .parentName은 이미 생성한 생성자에 넣을 뿐임
    
    따라서 Parent의 필드에 값을 넣어야 하므로 Child의 생성자에 super(전부다)를 넣어야 함
    

```java
public Child build() {
    return new Child(this.id, this.parentName, this.parentAge, 
                    this.childName, this.childAge);
    //     ↑ 생성자 호출이 필수!
}
```