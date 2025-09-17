# 아이템 9

### try-finally

```java
public class FirstError extends RuntimeException {
}

public class SecondError extends RuntimeException {
}

public class MyResource implements AutoCloseable {

    public void doSomething() {
        System.out.println("doSomething");
        throw new FirstError();
    }

    @Override
    public void close()  {
        System.out.println("close resource");
        throw new SecondError();
    }
}

public class AppRunner {

    public static void main(String[] args) {
        MyResource myResource = new MyResource();
        try {
            myResource.doSomething();
        } finally {
            myResource.close();
        }
    }
}

main 메서드 결과
doSomething
close resource
Exception in thread "main" item9.SecondError
	at item9.MyResource.close(MyResource.java:13)
	at item9.AppRunner.main(AppRunner.java:10)
	
	
SecondError만 찍히는 것을 볼 수 있음
```

### try-reources

```java
public class AppRunner {

    public static void main(String[] args) {

        try(MyResource myResource = new MyResource()) {
            myResource.doSomething();
        }
    }
}

doSomething
close resource
Exception in thread "main" item9.FirstError
	at item9.MyResource.doSomething(MyResource.java:7)
	at item9.AppRunner.main(AppRunner.java:8)
	Suppressed: item9.SecondError
		at item9.MyResource.close(MyResource.java:13)
		at item9.AppRunner.main(AppRunner.java:7)
		
에러코드가 First, Second 모두 찍히는 것을 볼 수 있음
```

**기존 try-finally의 문제점:**

1. **코드 복잡성**: 자원이 여러 개일 때 중첩된 try-finally 블록으로 인해 코드가 지저분해짐
2. **예외 누락**: finally 블록에서 발생한 예외가 try 블록의 예외를 덮어버려 디버깅이 어려워짐
3. **실수 유발**: 개발자가 자원 해제를 놓치기 쉬움

## try-with-resources의 장점

**1. 코드 간결성**

```java
*// Before (try-finally)*
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}

*// After (try-with-resources)*
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

**2. 예외 처리 개선**

- 원본 예외가 보존되고, finally에서 발생한 예외는 'suppressed' 상태로 스택 트레이스에 포함
- `getSuppressed()` 메서드로 프로그래밍적으로 접근 가능

**3. 여러 자원 처리**

```java
`static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}`
```