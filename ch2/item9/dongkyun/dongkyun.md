# item9

## ♻️ try-finally보다는 try-with-resources를 사용하라

---

- try-finally

  → 자원 닫기를 놓칠 확률이 크다

- **try-with-resources**가 이런 문제를 해결했다.
    - 이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.
    - 이는 문제를 진단하기도 훨씬 좋다.
        - 실전에서는 프로그래머에게 보여줄 예외 하나만 보존되고 여러 개의 다른 예외가 숨겨질 수도 있다.

          → 그냥 버려지지 않고, 숨겨졌다(suppressed) 꼬리표를 달고 스택 내역에 출력

    - catch 절 사용 시

      → try문을 중첩하지 않고도 다수의 예외 처리가 가능


**1️⃣ 기존 try-finally 방식**

```java
public class TryFinallyExample {
    public static void main(String[] args) {
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader("file.txt"));
            String line = reader.readLine();
            System.out.println(line);
        } catch (IOException e) {
            System.out.println("읽기 중 오류 발생: " + e.getMessage());
        } finally {
            try {
                if (reader != null) reader.close();
            } catch (IOException e) {
                System.out.println("닫는 중 오류 발생: " + e.getMessage());
            }
        }
    }
}
```

**2️⃣ try-with-resources 방식**

```java
public class TryWithResourcesExample {
    public static void main(String[] args) {
        try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
            String line = reader.readLine();
            System.out.println(line);
        } catch (IOException e) {
            System.out.println("읽기/닫기 중 오류 발생: " + e.getMessage());
            // 숨겨진 예외 확인
            for (Throwable t : e.getSuppressed()) {
                System.out.println("Suppressed: " + t);
            }
        }
    }
}
```

```java
읽기/닫기 중 오류 발생: file.txt (No such file or directory)
Suppressed: java.io.IOException: Stream close failed
```

- 읽기 / 닫기 예외 모두 출력 가능

  → 숨겨진 예외 X

- catch에서 getSuppressed()를 통해 숨겨진 예외도 확인 가능