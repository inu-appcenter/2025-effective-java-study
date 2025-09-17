# Item9

<aside>

try-finally 보다는 try-with-resources를 사용하라

</aside>

InputStream처럼, 자원을 사용하고 나서 반드시 닫아줘야 하는 상황이 있다. 이런 자원들은 제대로 해제하지 않으면 메모리 누수나 성능 문제로 이어질 수 있으니 신경을 써줘야 한다.

전통적인 방식으로는 try-finally가 있는데, 몇 가지 문제가 있다.

### 1. Why?

```java
// try-finally 
static String readFirstLine(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close(); // 자원 해제 필요
    }
}
```

자원을 하나 사용한다면 아직은 괜찮다.

```java
// try-finally 자원 두 개 이상 사용
static void copyFile(String src, String dst) throws IOException {
    FileInputStream in = new FileInputStream(src);
    try {
        FileOutputStream out = new FileOutputStream(dst);
        try { // 더러워요... 
            byte[] buf = new byte[1024];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

자원이 두 개로 늘어나니 확 더러워진다. 자원이 늘어날수록 중첩도 더 깊어지고, 코드 가독성도 더 나빠질 것이다.

```java
// 예외 손실
static String readFirstLine(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine(); // 예외 발생 가능 (첫 번째 예외)
    } finally {
        br.close(); // 이것도 예외 발생 가능 (두 번째 예외)
    }
}

// 두 번째 예외가 첫 번째 예외를 먹음 
// 진짜 원인(첫 번째 예외)을 알 수 없어 디버깅 어려움...
```

만약 readLine()에서 예외가 발생하고, 같은 이유로 close()에서도 예외가 발생한다면… 두 번째 예외가 첫 번째 예외를 집어삼켜 버린다. 스택 추적에는 close() 예외만 남고, 진짜 문제의 원인인 readLine() 예외는 사라져 디버깅이 어렵다.

### 2. try-with-resouces 사용

try-with-resources는 이런 문제들을 깔끔하게 해결한다. 사용 조건은 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다는 것.

```java
// try-with-resources 
static String readFirstLine(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
    // br.close() 자동 호출
}
```

```java
// try-with-resources 자원 여러 개 사용
static void copyFile(String src, String dst) throws IOException {
    try (FileInputStream in = new FileInputStream(src);
         FileOutputStream out = new FileOutputStream(dst)) {
        
        byte[] buf = new byte[1024];
        int n;
        while ((n = in.read(buf)) >= 0) {
            out.write(buf, 0, n);
        }
    }
    // in.close()와 out.close() 자동 호출~ 
}
```

### 3. 장점

- 간결해진 코드
    - 중첩된 try-finally 블록이 사라져서 코드가 훨씬 읽기 쉬워진다. 자원이 많아져도 코드 복잡도가 크게 증가하지 않는다.
- 예외 정보의 보존
    - 첫 번째 예외를 메인 예외로 표시하고, close()에서 발생한 예외들은 'suppressed' 예외로 기록한다. 디버깅할 때 진짜 원인을 놓치지 않을 수 있다.
- catch절과 함께 사용 가능
    - try-finally처럼 catch절도 함께 쓸 수 있어서 예외 처리가 더 유연하다.

자원을 다루는 코드를 작성할 때는 try-finally 대신 try-with-resources를 사용하자~