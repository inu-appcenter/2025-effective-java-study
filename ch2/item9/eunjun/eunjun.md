# Item 9. try-finally보다는 try-with-resources를 사용하라

- try-finally를 사용했을 때
    - 자원이 둘 이상일 때 코드가 지저분해지고, 직접 close()를 사용해야 한다는 번거로움이 있음.
        
        ```jsx
        static void copy(String src, String dst) throws IOException {
            InputStream in = new FileInputStream(src);
            try {
                OutputStream out = new FileOutputStream(dst);
                try {
                    byte[] buf = new byte[BUFFER_SIZE];
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
        
        ```
        
    
    - 예외가 try 블록과 finally 블록 모두에서 발생할 수 있는데, 두 블록 모두에서 발생한 경우 try 블록의 예외는 finally 블록의 예외에 묻혀버림.
        
        → 디버깅이 어려워짐.
        

- try-with-resources를 사용했을 때
    - 코드가 간편해지고, close()를 직접 사용할 필요가 없음.
        
        ```jsx
        static void copy(String src, String dst) throws IOException {
            try (InputStream in = new FileInputStream(src);
                 OutputStream out = new FileOutputStream(dst)) {
        
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            }
        }
        
        ```
        

- try-finally에서 발생하던 예외 덮어씌워지는 문제를 해결했음.
    - try 블록, close()에서 예외가 모두 발생하면 close()에서 발생한 예외는 숨겨지고, try 블록에서 발생한 예외가 스택 트레이스에 기록됨.
    - 자바 7에서 throwable에 추가된 getSuppressed 메서드를 이용하면 숨겨진 예외들도 출력할 수 있음.
    - 일반적인 try-catch, try-finally, try-catch-finally 구조와 try-with-resources를 함께 사용할 수 있다.
        
        → try {} catch {}가 일반적인 try-catch 구조,
        
             try-with-resources를 함께 쓴다면 try(AutoCloseable) {} catch{}가 된다.