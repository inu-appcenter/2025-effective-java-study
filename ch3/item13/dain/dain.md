<aside>

clone 재정의는 주의해서 진행하라

</aside>

Cloneable 인터페이스는 복제 가능한 클래스임을 명시하는 믹스인 인터페이스지만, 설계상 치명적 결함이 있다고 한다. clone 메서드가 Object에 있고 protected라서, Cloneable을 구현해도 외부에서 호출할 수 없다.

- mixins이란, (객체지향언어에서) 다른 클래스에서 사용할 목적으로 만들어진 클래스이다. 다중 상속의 제한이 없는 인터페이스로 구현하기 용이하다고 한다. 대상 타입의 주된 기능에 선택적 기능을 혼합(mixed in)한다고 해서 mixins라고 불린다.
  기

### 기본적인 clone 구현

기본 타입 or 불변 객체만 참조한다면 super.clone()으로도 충분하다.

```java
public class Point implements Cloneable {
    private final int x, y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public Point clone() {
        try {
            return (Point) super.clone(); 
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // Cloneable 구현했으므로 발생 불가
        }
    }
    
    // 사용 예시
    public static void main(String[] args) {
        Point original = new Point(10, 20);
        Point copied = original.clone();
        
        System.out.println(original != copied); // true - 다른 객체
        System.out.println(original.equals(copied)); // true - 같은 값
    }
}
```

### 그럼 가변 객체는?

가변 객체를 참조하면 원본과 복사본이 상태를 공유하게 된다는 문제가 있다.

```java
public class Person implements Cloneable {
    private String name;
    private List<String> hobbies;
    
    public Person(String name) {
        this.name = name;
        this.hobbies = new ArrayList<>();
    }
    
    public void addHobby(String hobby) {
        hobbies.add(hobby);
    }
    
    // 주의
    @Override
    public Person clone() {
        try {
            return (Person) super.clone(); // hobbies 리스트가 공유됨
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
    
    public static void main(String[] args) {
        Person original = new Person("김철수");
        original.addHobby("독서");
        
        Person copied = original.clone();
        copied.addHobby("영화감상"); // 원본에도 영향
        
        System.out.println("원본 취미: " + original.hobbies); // [독서, 영화감상]
        System.out.println("복사본 취미: " + copied.hobbies); // [독서, 영화감상]
        // 둘 다 같은 리스트를 참조하므로 문제 발생
    }
}
```

```java
public class Person implements Cloneable {
    private String name;
    private List<String> hobbies;
    
    @Override
    public Person clone() {
        try {
            Person result = (Person) super.clone();
            result.hobbies = new ArrayList<>(hobbies); // 리스트도 새로 생성
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
    
    public static void main(String[] args) {
        Person original = new Person("김철수");
        original.addHobby("독서");
        
        Person copied = original.clone();
        copied.addHobby("영화감상"); // 이제 복사본에만 추가됨
        
        System.out.println("원본 취미: " + original.hobbies); // [독서]
        System.out.println("복사본 취미: " + copied.hobbies); // [독서, 영화감상]
    }
}
```

### 복잡한 가변 상태(연결 리스트)의 경우

```java
public class LinkedList implements Cloneable {
    private Node head;
    
    private static class Node {
        String data;
        Node next;
        
        Node(String data, Node next) {
            this.data = data;
            this.next = next;
        }
        
        // 반복적인 deepCopy
        Node deepCopy() {
            Node result = new Node(data, next);
            for (Node p = result; p.next != null; p = p.next) {
                p.next = new Node(p.next.data, p.next.next);
            }
            return result;
        }
    }
    
    public void add(String data) {
        head = new Node(data, head);
    }
    
    @Override
    public LinkedList clone() {
        try {
            LinkedList result = (LinkedList) super.clone();
            if (head != null) {
                result.head = head.deepCopy(); // 연결 리스트 전체 복사
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
    
    public void printAll() {
        Node current = head;
        while (current != null) {
            System.out.print(current.data + " -> ");
            current = current.next;
        }
        System.out.println("null");
    }
    
    public static void main(String[] args) {
        LinkedList original = new LinkedList();
        original.add("C");
        original.add("B");
        original.add("A");
        
        LinkedList copied = original.clone();
        copied.add("D"); // 복사본에만 추가
        
        System.out.print("원본: ");
        original.printAll(); // A -> B -> C -> null
        
        System.out.print("복사본: ");
        copied.printAll(); // D -> A -> B -> C -> null
    }
}
```

### 복사 생성자/팩터리 방식(권장)

clone 보다 복사 생성자, 복사 팩터리가 훨씬 안전하고 권장된다.

**Why?**

- 생성자를 사용하지 않는 위험한 객체 생성 방식 회피
- final 필드와 충돌하지 않음
- 불필요한 검사 예외 없음
- 형변환 불필요
- 인터페이스 타입으로 변환 가능

```java
public class Student {
    private String name;
    private List<String> subjects;
    
    // 일반 생성자
    public Student(String name) {
        this.name = name;
        this.subjects = new ArrayList<>();
    }
    
    // 복사 생성자 -
    public Student(Student other) {
        this.name = other.name;
        this.subjects = new ArrayList<>(other.subjects); // 자동으로 깊은 복사
    }
    
    // 복사 팩터리 + 타입 변환도 가능
    public static Student from(Student other) {
        return new Student(other);
    }
    
    // 다른 컬렉션 타입으로부터도 생성 가능
    public static Student fromSubjects(String name, Collection<String> subjects) {
        Student student = new Student(name);
        student.subjects.addAll(subjects);
        return student;
    }
    
    public void addSubject(String subject) {
        subjects.add(subject);
    }
    
    public static void main(String[] args) {
        Student original = new Student("홍길동");
        original.addSubject("수학");
        original.addSubject("과학");
        
        // 복사 생성자 사용
        Student copied1 = new Student(original);
        copied1.addSubject("영어");
        
        // 복사 팩터리 사용
        Student copied2 = Student.from(original);
        copied2.addSubject("국어");
        
        System.out.println("원본 과목 수: " + original.subjects.size()); // 2
        System.out.println("복사본1 과목 수: " + copied1.subjects.size()); // 3
        System.out.println("복사본2 과목 수: " + copied2.subjects.size()); // 3
        
        // HashSet을 ArrayList로 변환하는 예시
        Set<String> subjectSet = Set.of("물리", "화학", "생물");
        Student converted = Student.fromSubjects("이순신", subjectSet);
        System.out.println("변환된 학생 과목 수: " + converted.subjects.size()); // 3
    }
}
```

### 정리

- **새로운 클래스는 `Cloneable` 구현 금지**
- **복제가 필요하면 복사 생성자나 복사 팩터리 사용**
- **예외로, 배열은 `clone` 방식이 가장 깔끔함**