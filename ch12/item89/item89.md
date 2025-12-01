# Item89

> ***인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라***

<br>

### readResolve와 readObject

열거 타입 활용 방식을 알아보기 전, readResolve와 readObject 메서드가 각각 어떤 역할을 하는지 명확하게 하고 가자.

<br>

**readObject**

```java
private void readObject(ObjectInputStream ois) 
    throws IOException, ClassNotFoundException
```

<br>

**readObject** 메서드는 역직렬화 과정(JSON → 객체) 객체를 복원하는 방법을 커스터마이징할 수 있게 한다. ObjectInputStream이 스트림에서 클래스의 필드에 해당하는 부분을 읽은 후에 호출된다.

<br>

읽은 필드를 실제 객체 필드에 매핑하여 초기화 하고, transient 필드를 재계산하거나 기본값으로 초기화한다. 

<br>

**readResolve**

```java
private Object readResolve() throws ObjectStreamException
```

<br>

readResolve는 역직렬화가 완료된 객체를 다른 객체로 혹은 그대로 반환할 지 커스터마이징할 수 있게 한다.

<br>

### 싱글톤임을 보장

**readResolve**에서 역직렬화된 객체(더이상은 싱글톤이지 않은)를 반환하게 되면 싱글톤은 바로 깨지게 된다. 그럼 반대로 아래 코드처럼 기존 싱글톤 객체를 반환하면 어떻게 될까?

```java
private Object readResolve() {
    return INSTANCE;
}
```

<br>

새로 생성된 객체가 반환되지 않고 기존 싱글톤을 반환하여 싱글톤의 속성을 유지할 수 있으며 새로 생성된 객체는 사용되지 않고 가비지 컬렉터에 의해 정리된다.

<br>

따라서, 싱글톤 경우 한정으로 모든 필드는 **transient**(직렬화/역직렬화 대상에서 제외)로 선언해야 한다. 그렇지 않으면, 실제 참조를 들고 있는 역직렬화 객체는 공격 대상이 된다.

<br>

### 어떻게 공격의 대상이

```java
public class Apple implements Serializable {
    public static final Apple INSTANCE = new Apple();
    private String[] varieties = { "Fuji", "Gala" };
    
    public void printVarieties() {
        System.out.println(Arrays.toString(varieties));
    }
    
    private Object readResolve() {
        return INSTANCE;
    }
    
    private static final long serialVersionUID = 0;
}
```

<br>

이 **Apple** 클래스는 Apple 타입의 인스턴스와 **String 배열** 타입의 필드를 들고 있다. 이 때, 악의적인 사용자가 **String** 배열 타입의 필드가 아닌 아래 **AppleStealer** 타입의 스트림을 보냈다고 가정해보자.

<br>

```java
public class AppleStealer implements Serializable {
    static Apple impersonator;
    private Apple payload;
    
    private Object readResolve() {
        impersonator = payload;
        return new String[] { "Granny Smith" };
    }
    
    private static final long serialVersionUID = 0;
}
```

<br>

조작된 스트림 내의 **payload** 필드는 자기 자신을 순환참조로 가리키고 있다. 그러면 이 **AppleStealer**는 **Apple**의 필드로서 역직렬화되는데, 그 순간 **payload** 필드가 자기 자신을 참조하고 있으므로 원래는 가비지 컬렉터에 의해 정리되어야 할 임시 객체를 참조하게 된다.

<br>

그리고 이 필드가 **static**이라 가비지 컬렉터에 의해 정리되지도 않는다. 그리고 이 클래스는 타입 검증을 통과하기 위해 원래 필드 타입에 맞는 문자열 배열을 반환하게 되고, 결국 싱글턴이 아닌 조작된 객체가 생성되게 되는 것이다.

<br>

~~이 부분은 복잡해서 발표에서 다시 풀어서 설명하겠습니다..~~

### 그럼 어떻게 해결하나요

**varieties** 필드를 **transient**로 선언하여 역직렬화 대상에서 제외하는 방법으로 우회할 수도 있지만, 아이템에서는 **원소 하나짜리 열거 타입으로 바꾸는 것을 권장**하고 있다. 

<br>

열거 타입은 역직렬화 시 **새로운 인스턴스를 생성하지 않고** 이름만으로 기존 상수를 조회해서 반환하기 때문이다.

<br>

일반 클래스는 역직렬화 시 임시 객체를 생성하고 필드를 복원하는 과정에서 공격자가 개입할 여지가 있지만, 열거 타입은 그런 과정 자체가 없다.

<br>

즉, **탈취당할 임시 객체가 애초에 생성되지 않는다.**

```java
public enum Apple {
    INSTANCE;
    
    private String[] varieties = { "Fuji", "Gala" };
    
    public void printVarieties() {
        System.out.println(Arrays.toString(varieties));
    }
}
```

<br>

### readResolve의 접근자

- **final** 클래스 → **private**
- 상속 관계에 있는 경우 → **package**-**private** 이상
