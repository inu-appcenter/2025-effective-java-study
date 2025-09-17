# Item12

<aside>

toString을 항상 재정의하라

</aside>

## 1. Why?

- 사람이 보기 편하고 유의미한 정보를 제공할 수 있음

    ```java
    public class Student {
        private String name;
        private int age;
    
        // toString 재정의 안함
    }
    
    Student student = new Student("김철수", 20);
    System.out.println(student); // Student@1a2b3c4d (의미없음)
    ```

  Object의 기본 toString은 별로 도움이 되지 않는다... "클래스명@해시코드" 형태로만 나와서 디버깅할 때나 로그를 볼 때 객체가 어떤 값을 가지고 있는지 알 수가 없다. 따라서 모든 구체 클래스에서 toString을 재정의해야 한다.

    ```java
    public class Student {
        private String name;
        private int age;
    
        @Override
        public String toString() {
            return String.format("Student{name='%s', age=%d}", name, age);
        }
    }
    
    Student student = new Student("김철수", 20);
    System.out.println(student); // Student{name='김철수', age=20} (유용함~)
    
    ```

  객체의 상태를 한눈에 파악할 수 있으니 로그 분석, 디버깅이 수월해진.


## 2. toString이 자동 호출되는 상황

toString은 개발자가 직접 호출하지 않아도 자동으로 호출되는 경우가 많다.

특히, 로그에서 자동 호출되는 경우는 (toString이 제대로 구현되어 있지 않으면)오류 상황을 파악하기 어려우니 실무적으로 가장 중요해 보인다.

```java
Student student = new Student("김철수", 20);

// 1. print에서 자동 호출
System.out.println(student);

// 2. 문자열 연결에서 자동 호출
String message = "학생 정보: " + student;

// 3. 컬렉션에서 자동 호출
List<Student> students = Arrays.asList(student);
System.out.println(students); // [Student{name='김철수', age=20}]

// 4. 로그에서 자동 호출
log.error("처리 실패: " + student);

```

## 3. 포맷 명시 여부 결정

toString을 구현할 때 중요한 것은 반환값의 포맷을 명시할지의 여부라고 한다.

### 포맷 명시 (값 클래스에 권장)

- toString이 반환할 정확한 형태를 문서에 명시
- 표준화된 사용 가능(그대로 쓰면 되니까)
- 파싱 메서드 제공 가능
- 대신, 기존 설계를 변경하기 어려움

```java
public class PhoneNumber {
    private int areaCode, prefix, lineNum;

    /**
     * "XXX-YYY-ZZZZ" 형태로 반환
     */
    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }

    // 파싱 메서드도 함께 제공
    public static PhoneNumber fromString(String str) {
        String[] parts = str.split("-");
        return new PhoneNumber(
            Integer.parseInt(parts[0]),
            Integer.parseInt(parts[1]),
            Integer.parseInt(parts[2])
        );
    }
}

```

### 포맷 미명시 (유연성 필요시)

- 형태를 구체적으로 정하지 않고 대략적인 설명만 제공
- 파싱이 복잡하고 어려우니 사실상 안 함
- 표준화 어려움 (일관성X)

```java
public class Book {
    private String title;
    private String author;
    private int pages;

    /**
     * 책 정보를 반환한다.
     * 형식은 향후 변경될 수 있다.
     */
    @Override
    public String toString() {
        return String.format("Book{title='%s', author='%s', pages=%d}",
                           title, author, pages);
    }
}

```

## 4. 접근자 제공 필수

toString의 정보에 접근할 수 있는 getter 메서드를 반드시 제공해야 한다. 그렇지 않으면 사용자가 문자열을 파싱해야 하는 번거로운 상황이 발생한다.

- 제공하지 않으면…

    1. 번거로움 - 매번 파싱 코드 작성

    2. 성능 저하 - 문자열 생성 + 파싱 과정

    3. 오류 가능성 - 파싱 실패 위험

    4. 취약성 - toString 형태 바뀌면 코드 깨짐


```java
public class PhoneNumber {
    private int areaCode, prefix, lineNum;

    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }

    // toString 정보에 접근할 수 있는 getter 제공
    public int getAreaCode() { return areaCode; }
    public int getPrefix() { return prefix; }
    public int getLineNum() { return lineNum; }
}

// 사용자가 문자열 파싱 대신 직접 접근 가능
PhoneNumber phone = new PhoneNumber(707, 867, 5309);
int areaCode = phone.getAreaCode(); // 파싱 불필요!

```

## 5. 구현

### 기본 패턴

```java
public class Pizza {
    private String name;
    private int size;
    private List<String> toppings;

    @Override
    public String toString() {
        return String.format("Pizza{name='%s', size=%d, toppings=%s}",
                           name, size, toppings);
    }
}

```

### 큰 객체는 요약 정보만

- 간결하고 유익한 정보를 제공해야 함… 큰 컬렉션이나 복잡한 객체의 경우 모든 정보를 다 보여주면 오히려 가독성이 떨어지니 주의하자.

```java
public class StudentList {
    private List<Student> students;

    @Override
    public String toString() {
        return String.format("StudentList{count=%d}", students.size());
        // 모든 학생 정보를 다 출력하지 않음!!
    }
}

```

## 6. toString 재정의 하지 않는 경우

- 정적 유틸리티 클래스
    - 인스턴스를 만들 수 없으니 toString이 의미 없다
- 열거 타입
    - Java가 이미 완벽한 toString 제공한다.
- 추상 클래스
    - 하위 클래스에서 공통으로 사용할 구현을 제공하는 경우만


## 그래서…

1. 모든 구체 클래스에서 toString 재정의
2. 간결하고 유익한 정보 반환
3. 포맷 명시 여부 신중히 결정
4. toString 정보에 대한 접근자 제공
5. 디버깅에 도움되는 형태로 구현