# Item6

<aside>

불필요한 객체 생성을 피하라

</aside>

불필요한 객체 생성은 불필요한 자원 소비를 유발한다.

자주 호출되는 메서드나 반복문 내에서 매번 새 객체를 생성한다면, 불필요한 부하와 가비지 컬렉션 비용이 발생한다.

```java
String s = new String("cat");
```

예를 들어 보자면, 위의 new String(”cat”)은 매번 새 String 객체를 만든다.

기능적으로는 문자열 리터럴 “cat”과 동일하지만, 메모리 낭비가 심하다. 이런 코드가 반복문 내에 포함되어 있다고 생각해보자… 🤮

굳이 이럴 필요가 없으니 문자열 리터럴을 사용하자.

```java
String s = "cat";
```

생성 비용이 비싼 객체는 어떻게 사용해야 할까?

```java
boolean isAnimalName(String s) {
    return s.matches("cat|dog|bird");  // 매번 Pattern 객체를 새로 생성함
}
```

위에서 사용한 String.matches는 쉽고 비싼 방식이다. 매번 Pattern 객체를 새로 생성하고, 그대로 버려져서 가비지 컬렉션 대상이 되니 반복적인 사용은 성능 면에서 부적합하다.

그럼 이런 경우엔 어떻게 해야하나…

```java
public class AnimalNameChecker {
    private static final Pattern ANIMAL_PATTERN = Pattern.compile("cat|dog|bird");

    public static boolean isAnimalName(String s) {
        return ANIMAL_PATTERN.matcher(s).matches();
    }
}
```

Pattern 객체를 **`static final`** 필드로 미리 생성해 캐싱해두고, 메서드 호출 시마다 재사용하자. 해당 메서드가 자주 호출된다면 성능이 상당히 향상될 것이다. 뿐만 아니라 Pattern 인스턴스를 끄집어내어 네이밍하니 그 의미가 명확히 드러난다.

재사용이 의도를 벗어나는 경우도 존재한다.

**‘어댑터(Adapter)**’는 뒷단 객체에 작업을 위임하는 일종의 뷰(view) 역할을 한다.

이 뷰 객체는 자체 상태 없이, 항상 하나의 뒷단 객체 상태를 참조한다. 어댑터 인스턴스를 여러 개 만들어도 모두 실제 뒷단 객체의 상태를 반영하니, 여러 개의 뷰가 똑같은 객체를 나타내는 효과를 낸다. 따라서 재사용해도 아무 문제가 없다.

```java
// 상태 없는 어댑터: 재사용 안전
public class AnimalViewAdapter {
    private final Animal animal;

    public AnimalViewAdapter(Animal animal) {
        this.animal = animal;
    }

    public String getSound() {
        return animal.makeSound();
    }
}

```

그러나 만약 어댑터가 독자적인 상태를 내부에 갖거나 수정 가능하게 갖는다면?

어댑터가 자신의 고유 데이터를 저장하거나, 내부 가변 상태를 가진다면 여러 곳에서 같은 어댑터 인스턴스를 공유해선 안 된다. 어댑터를 재사용할 때 그 내부 상태가 변경되면, 어댑터를 참조하는 모든 곳에 영향을 준다.

→ 한 클라이언트가 어댑터 객체의 상태를 바꾸면, 동시에 다른 클라이언트도 그 변경된 상태를 보게 되면서 예기치 않은 버그와 동시성 문제가 발생할 수 있다.

```java
// 상태 있는 어댑터: 공유 시 문제 발생 가능
public class StatefulAnimalAdapter {
    private Animal animal;
    private String currentMood;  // 어댑터 독립 상태

    public StatefulAnimalAdapter(Animal animal) {
        this.animal = animal;
        this.currentMood = "Happy";
    }

    public void setMood(String mood) {
        this.currentMood = mood;
    }

    public String describe() {
        return animal.makeSound() + " is " + currentMood;
    }
}

```

**`currentMood`**라는 내부 상태를 갖는 어댑터는 여러 클라이언트가 공유하면, 한 곳에서 **`setMood`** 호출 시 다른 곳에도 영향을 미쳐 버그를 유발할 수 있다. 이런 경우에는 어댑터를 새로 만들거나 클라이언트 별로 별도 인스턴스를 유지해야 한다.