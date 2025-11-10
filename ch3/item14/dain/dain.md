<aside>

Comparable을 구현할지 고려하라

</aside>

Comparable의 역할은 객체 스스로의 기본 정렬 순서를 정의할 수 있도록 하는 것이다. 즉, Comparable을 구현하면 클래스의 인스턴스에 순서가 있다.

- Comparable 인터페이스를 구현한 클래스의 객체들은 Arrays.sort(), Collections.sort() 메소드를 이용하거나 compareTo()를 직접 구현하여 사용할 수 있다.

```java
public class StudentGrades {
    public static void main(String[] args) {
        String[] names = {"김철수", "박영희", "이민수", "최지영"};
        Arrays.sort(names); 
        
        Set<String> sortedNames = new TreeSet<>(Arrays.asList(names));
        System.out.println(sortedNames); // 자동으로 정렬
        
        // 결과: [김철수, 박영희, 이민수, 최지영]
    }
}
```

### 단일 필드 비교

```java
public class Student implements Comparable<Student> {
    private final String name;
    private final int grade;
    
    public Student(String name, int grade) {
        this.name = name;
        this.grade = grade;
    }
    
    @Override
    public int compareTo(Student other) {
        // Integer.compare 사용 
        return Integer.compare(this.grade, other.grade);
    }
    
    public static void main(String[] args) {
        List<Student> students = Arrays.asList(
            new Student("김철수", 85),
            new Student("박영희", 92),
            new Student("이민수", 78)
        );
        
        Collections.sort(students);
        
        for (Student s : students) {
            System.out.println(s.name + ": " + s.grade);
        }
        // 결과: 이민수: 78, 김철수: 85, 박영희: 92
    }
}
```

### 여러 필드 비교

가장 중요한 필드부터 비교하고, 결과가 0이 아니면 즉시 반환한다.

```java
public class PhoneNumber implements Comparable<PhoneNumber> {
    private final short areaCode, prefix, lineNum;
    
    public PhoneNumber(short areaCode, short prefix, short lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }
    
    @Override
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode); // 가장 중요
        if (result != 0) return result;
        
        result = Short.compare(prefix, pn.prefix); // 두 번째로 중요
        if (result != 0) return result;
        
        return Short.compare(lineNum, pn.lineNum); // 마지막
    }
    
    public static void main(String[] args) {
        List<PhoneNumber> numbers = Arrays.asList(
            new PhoneNumber((short)02, (short)123, (short)4567),
            new PhoneNumber((short)02, (short)123, (short)1234),
            new PhoneNumber((short)031, (short)456, (short)7890)
        );
        
        Collections.sort(numbers);
        
        for (PhoneNumber pn : numbers) {
            System.out.printf("%03d-%03d-%04d%n", 
                pn.areaCode, pn.prefix, pn.lineNum);
        }
        // 결과: 02-123-1234, 02-123-4567, 031-456-7890
    }
}
```

### 비교자 생성 메서드 활용

```java
public class Employee implements Comparable<Employee> {
    private final String name;
    private final String department;
    private final int salary;
    
    public Employee(String name, String department, int salary) {
        this.name = name;
        this.department = department;
        this.salary = salary;
    }
    
    // 정적 비교자를 미리 생성 (성능상 유리함)
    private static final Comparator<Employee> COMPARATOR =
            comparing((Employee e) -> e.department)          // 부서별로
            .thenComparing(e -> e.salary, reverseOrder())    // 연봉 내림차순
            .thenComparing(e -> e.name);                     // 이름순
    
    @Override
    public int compareTo(Employee other) {
        return COMPARATOR.compare(this, other);
    }
    
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("김철수", "개발", 5000),
            new Employee("박영희", "개발", 6000),
            new Employee("이민수", "개발", 5000),
            new Employee("최지영", "마케팅", 4500),
            new Employee("정수현", "마케팅", 4800)
        );
        
        Collections.sort(employees);
        
        employees.forEach(e -> 
            System.out.printf("%-6s | %-4s | %,7d원%n", 
                e.name, e.department, e.salary));
        
        // 결과:
        // 박영희 | 개발 | 6,000원  (개발팀, 연봉 높은 순)
        // 김철수 | 개발 | 5,000원  (개발팀, 같은 연봉이면 이름순)
        // 이민수 | 개발 | 5,000원
        // 정수현 | 마케팅 | 4,800원 (마케팅팀, 연봉 높은 순)
        // 최지영 | 마케팅 | 4,500원
    }
}
```

### equals와 compareTo는 일관되어야 한다

Why? → 정렬된 컬렉션에서 예상과 다른 동작이 일어나지 않도록.

```java
public class InconsistentExample {
    public static void main(String[] args) {
        BigDecimal bd1 = new BigDecimal("1.0");
        BigDecimal bd2 = new BigDecimal("1.00");
        
        // equals는 false, compareTo는 0
        System.out.println("equals: " + bd1.equals(bd2));        // false
        System.out.println("compareTo: " + bd1.compareTo(bd2));  // 0
        
        // HashSet은 equals 사용 - 2개 원소
        Set<BigDecimal> hashSet = new HashSet<>();
        hashSet.add(bd1);
        hashSet.add(bd2);
        System.out.println("HashSet 크기: " + hashSet.size()); // 2
        
        // TreeSet은 compareTo 사용 - 1개 원소
        Set<BigDecimal> treeSet = new TreeSet<>();
        treeSet.add(bd1);
        treeSet.add(bd2);
        System.out.println("TreeSet 크기: " + treeSet.size()); // 1
    }
}
```

### 잘못된 비교

값의 차이를 이용한 비교를 주의하

```java
// 잘못된 방식 - 오버플로 위험
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode(); // 위험
    }
};

// 올바른 방식 1 - 정적 compare 메서드
static Comparator<Object> hashCodeOrder = 
    (o1, o2) -> Integer.compare(o1.hashCode(), o2.hashCode());

// 올바른 방식 2 - 비교자 생성 메서드
static Comparator<Object> hashCodeOrder = 
    Comparator.comparingInt(o -> o.hashCode());
```

### 정리

- 순서가 명확한 값 클래스는 Comparable 구현 필수
- compareTo에서 <, > 연산자 사용 금지 - 박싱된 기본 타입의 compare 메서드 사용
- 비교자 생성 메서드로 간결하고 유연한 정렬 로직 구현 가능
- equals와 compareTo 일관성 유지 권장