# Item 9

추가 일시: 2025년 9월 13일 오전 2:59
강의: Effective JAVA

## 🍀 Item 9: try-finally보다는 try-with-resources를 사용하라

---

자바에서 자원 관리 시 try-finally 대신 try-with-resources를 사용하면 더 간결한 코드와 안정성을 확보할 수 있다.

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였는데 이는 자원이 둘 이상이면 중첩 try문으로 코드가 지저분해지는 것을 볼 수 있다.

**나쁜 예시: try-finally로 2개 자원 처리**

```java
static void copyFile(String src, String dst) throws IOException {
    FileInputStream in = new FileInputStream(src);
    try {
        FileOutputStream out = new FileOutputStream(dst);
        try {
            // 파일 복사 작업
            byte[] buffer = new byte[1024];
            int bytesRead;
            while ((bytesRead = in.read(buffer)) != -1) {
                out.write(buffer, 0, bytesRead);
            }
        } finally {
            out.close(); // 2번째 자원 닫기
        }
    } finally {
        in.close(); // 1번째 자원 닫기
    }
}
```

뿐만 아니라 마지막에 발생한 예외만 메서드 밖으로 던지는 Java 특성상 디버그에 어려움을 겪을 수 있다.

```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine(); // ← 1번 예외 발생! (디스크 오류)
    } finally {
        br.close(); // ← 2번 예외 발생! (같은 디스크 오류로)
    }
}
```

다음과 같이 진짜 원인인 readLine()의 예외는 무시된다.

**좋은 예시: try-with-resources 사용**

```java
static void copyFile(String src, String dst) throws IOException {
    try (FileInputStream in = new FileInputStream(src);
         FileOutputStream out = new FileOutputStream(dst)) {
        
        byte[] buffer = new byte[1024];
        int bytesRead;
        while ((bytesRead = in.read(buffer)) != -1) {
            out.write(buffer, 0, bytesRead);
        }
    } // 자동으로 역순(out → in)으로 닫힘
}
```

때문에 try-with-resources를 사용하면 중첩도 사라지고, 예외 처리도 안전하게 할 수 있다.

```java
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

추가적으로 catch절과 함께 사용이 가능하다.