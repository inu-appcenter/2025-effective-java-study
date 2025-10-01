# item13

## ©️ clone 재정의는 주의해서 진행하라

---

⚠️ Cloneable과 clone()은 **설계상 여러 문제**가 많다.

**1️⃣ 불변 객체 / 기본 타입만 있는 경우**

```java
class PhoneNumber implements Cloneable {
    private final int areaCode;
    private final int number;

    public PhoneNumber(int areaCode, int number) {
        this.areaCode = areaCode;
        this.number = number;
    }

    @Override
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // 일어나지 않음
        }
    }
}

public class Main {
    public static void main(String[] args) {
        PhoneNumber p1 = new PhoneNumber(123, 4567);
        PhoneNumber p2 = p1.clone();

        System.out.println(p1 == p2); // false, 다른 객체
        System.out.println(p1.equals(p2)); // false, equals 미정의
    }
}
```

- 기본 타입/불변 객체만 있어서 super.clone()으로 충분
- 복제본과 원본은 **서로 다른 객체**.

**2️⃣ 가변 객체를 참조하는 경우 (배열)**

```java
class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;

    public Stack() {
        elements = new Object[16];
    }

    public void push(Object e) { elements[size++] = e; }
    public Object pop() { return elements[--size]; }

    @Override
    public Stack clone() {
        try {
            Stack copy = (Stack) super.clone();
            copy.elements = elements.clone(); // 배열 복제
            return copy;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Stack s1 = new Stack();
        s1.push("A");

        Stack s2 = s1.clone();
        s2.push("B");

        System.out.println(s1.pop()); // A
        System.out.println(s2.pop()); // B
    }
}
```

- 단순히 super.clone()만 쓰면 elements 배열을 공유 → 복제본/원본이 서로 영향을 줌
  → **배열 복제 필요**

**3️⃣ 깊은 복사 예시 (연결 리스트)**

```java
class Node {
    int value;
    Node next;

    Node(int value, Node next) { this.value = value; this.next = next; }

    Node deepCopy() {
        return new Node(value, next == null ? null : next.deepCopy());
    }
}

class LinkedList implements Cloneable {
    Node head;

    LinkedList(Node head) { this.head = head; }

    @Override
    public LinkedList clone() {
        try {
            LinkedList copy = (LinkedList) super.clone();
            copy.head = head == null ? null : head.deepCopy(); // 깊은 복사
            return copy;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        LinkedList list1 = new LinkedList(new Node(1, new Node(2, null)));
        LinkedList list2 = list1.clone();

        list2.head.value = 100;
        System.out.println(list1.head.value); // 1, 원본 영향 없음
    }
}
```

- 단순 배열보다 **가변 상태가 더 복잡** → deepCopy() 필요
- 깊은 복사 후 원본과 독립적으로 동작

- 📝 정리
    1. **불변 객체** → super.clone()만으로 충분
    2. **가변 객체(배열)** → elements.clone()
    3. **복잡한 가변 객체(연결 리스트, 해시테이블)** → deep copy 필요
    4. Cloneable + clone() → 항상 **super.clone() → 필요시 수정 → 자기 타입 반환(공변 반환)**