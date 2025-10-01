# item14

## ↔️ Comparable을 구현할지 고려하라

---

- compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.

‼ **compareTo 작성 요령**

- 기본 타입 필드 비교: Integer.compare, Double.compare 등 **정적 compare 메서드** 사용
- 객체 참조 필드 비교: 재귀적으로 compareTo 호출
- 여러 필드가 있는 경우: 중요도 순서대로 비교, 먼저 순서가 결정되면 바로 반환

**1️⃣ Comparable 기본 구조**

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // compareTo 구현: 나이 기준으로 오름차순 정렬
    @Override
    public int compareTo(Person other) {
        return Integer.compare(this.age, other.age);
    }

    @Override
    public String toString() {
        return name + "(" + age + ")";
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Person[] people = {
            new Person("Alice", 30),
            new Person("Bob", 25),
            new Person("Charlie", 35)
        };

        Arrays.sort(people); // Comparable 구현 덕분에 정렬 가능

        System.out.println(Arrays.toString(people));
        // 출력: [Bob(25), Alice(30), Charlie(35)]
    }
}
```

✅ Comparable 구현 → Arrays.sort나 TreeSet에서 **자동 정렬** 가능

**2️⃣ 여러 필드 비교**

```java
public class PhoneNumber implements Comparable<PhoneNumber> {
    private short areaCode;
    private short prefix;
    private short lineNum;

    public PhoneNumber(short areaCode, short prefix, short lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }

    @Override
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(this.areaCode, pn.areaCode);
        if (result == 0) result = Short.compare(this.prefix, pn.prefix);
        if (result == 0) result = Short.compare(this.lineNum, pn.lineNum);
        return result;
    }

    @Override
    public String toString() {
        return areaCode + "-" + prefix + "-" + lineNum;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        PhoneNumber[] numbers = {
            new PhoneNumber((short) 123, (short) 45, (short) 6789),
            new PhoneNumber((short) 123, (short) 44, (short) 6789),
            new PhoneNumber((short) 122, (short) 99, (short) 1111)
        };

        Arrays.sort(numbers);

        System.out.println(Arrays.toString(numbers));
        // 출력: [122-99-1111, 123-44-6789, 123-45-6789]
    }
}
```

✅ 중요한 필드 → 덜 중요한 필드 순으로 비교

**3️⃣ Comparator 활용**

```java
public class PhoneNumber {
    private short areaCode;
    private short prefix;
    private short lineNum;

		// 첫 번째 비교자 : areaCode
		// 두 번째 ~~ : prefix
		// 세 번째 ~~ : lineNum 
    private static final Comparator<PhoneNumber> COMPARATOR =
        Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
                  .thenComparingInt(pn -> pn.prefix)
                  .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
}
```

✅ **비교자 생성 메서드로 깔끔하게 비교 순서 정의**

⚠️ 주의 사항
→ compareTo와 equals가 일치하지 않으면 **TreeSet/TreeMap에서 예상과 다른 동작**