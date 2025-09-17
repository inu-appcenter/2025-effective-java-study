# Item09

> ***try-finally보다는 try-with-resources를 사용하라***
<br> 

## 자원 닫기

 개발자가 직접 할당받은 자원은 사용이 끝나면, 혹은 쓰임이 더이상 없으면 반납하여 메모리 누수나 성능 문제를 방지하여야 한다. **Java**에서는 **InputStream**, **OutputStream**, **Connection** 등과 같은 라이브러리들의 다수가 **finalizer**를 사용하고 있지만, 이전 아이탬에서 알아본 바에 따르면 이는 믿을만하지 못하다.

<br>

 자원이 제대로 닫힘을 보장하는 수단으로 **try-finally**를 사용하였지만, 사용하는 자원이 둘 이상이라면 각 자원에서 발생한 예외가 집어삼켜지거나 코드의 가독성을 해칠 수 있다.

<br>

**예외가 집어 삼켜지는 경우**

```java
public class ExceptionSuppression {
    public static String readFirstLine(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine(); // IOException 발생 가능
        } finally {
            br.close(); // IOException 발생 가능
        }
    }
}
```

<br>

 만약 **try**, **finally** 각각에서 예외가 발생하면 **try**에서 발생한 예외는 이후 **finally**에서 발생한 예외에 의해 집어삼켜지게 되며, 예외를 디버깅하는데 어려움을 겪을 수 있다.

 <br>

**코드의 가독성을 해치는 경우**

```java
public class ReadabilityProblem {
    public static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[1024];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
}
```

<br>

## try-with-resources

 이 구조를 사용하기 위해서는 자원이 **AutoCloseable** 인터페이스를 구현해야 하는데, 다행히도 자주 사용하는 자바의 라이브러리들은 이를 구현하거나 확장해놓았다. 
 
 <br>

```java
public class TryWithResources {
    public static String readFirstLine(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }
    
    public static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[1024];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
}

<br>
```

가독성이 이전 방법에 비해 훨씬 좋을 뿐만 아니라 **readLine** 부분과 **close** 두 메소드에서 예외가 발생해도, ***숨겨진(suppressed)*** 예외까지 모두 확인할 수 있다.
