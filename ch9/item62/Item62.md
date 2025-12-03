# Item 62. 다른 타입이 적절하다면 문자열 사용을 피하라

- 데이터를 저장할 때 boolean, int, float 등 더 적절한 타입이 있는데 String으로 저장하면 안됨.

- 열거타입으로 저장해야 할 데이터를 문자열로 저장하면 타입 안정성만 떨어짐.

- 여러 자료형이 섞여있는 혼합 타입을 문자열로 대신하는 것은 적합하지 않음.
    
    ```jsx
    String compoundKey = className + "#" + i.next();
    
    className = User, i.next = 12
    -> compoundKey = "User#12"
    
    ```
    
    - compoundKey 내부 구분자는 #인데, 만약 className이나 i.next에 #이 껴있다면 명확히 구분이 안됨.
    - 여기서 className이나 i.next를 다시 얻고싶다면 파싱을 거쳐야 함.
    - compoundKey라는 문자열을 구성하는 내부 요소들은 타입 정보를 잃어버렸고, 만약 compoundKey끼리 비교한다고 하면 className이나 [i.next](http://i.next) 등을 이용해 compareTo나 equals 사용이 불가능함.
    
    → 여러 자료형이 섞여있는 혼합타입을 사용할 때는 private 정적 멤버 클래스, 즉 전용클래스를 새로 만들어야 함.
    
    ```jsx
    private static class CompoundKey {
        private final String className;
        private final int index;
    
        public CompoundKey(String className, int index) {
            this.className = className;
            this.index = index;
        }
    
        @Override
        public boolean equals(Object o) { ... }
    
        @Override
        public int hashCode() { ... }
    
        @Override
        public String toString() {
            return className + ":" + index;
        }
    }
    
    ```
    

- 문자열은 권한을 표현하기 적합하지 않음.
    
    ```jsx
    public class ThreadLocal {
    	private ThreadLocal() {} // 객체 생성 불가
    	
    	public static void set (String key, Object value);
    	public static Object get (String key);
    }
    	
    ```
    
    - 🤷‍♂️ ThreadLocal이란?
        - ThreadLocal은 멀티스레드에서 공유할 필요 없는 값들에 대해, 같은 코드라도 스레드마다 독립된 값을 유지할 수 있도록 해주는 클래스임.
        - 각 스레드들은 내부적으로 ThreadLocalMap을 가지고 있고, ThreadLocal.set(key, value)를 호출하면 Map 안에 key와 value를 저장함.
    
    - 위 코드는 ThreadLocal의 key를 문자열로 사용한 예시임.
        - ThreadLocal의 key는 고유해야 하지만, 만약 문자열을 key로 사용한다면 다른 클라이언트가 같은 키를 사용할 수도 있음.
            
            → 기능 오류, 보안 취약
            
        
        - JDK ThreadLocal의 key는 문자열이 아니라 ThreadLocal 객체의 참조값임. 이렇게 된다면 사용자가 위조할 수도 없고 고유한 값을 가짐.
            
            ```jsx
            public class ThreadLocal {
            	
            ```
            
    
    - ThreadLocal 리팩토링 과정
        
        ```jsx
        public class ThreadLocal {
            private ThreadLocal() { }
        
        		// 책의 코드에서는 public이지만 private로 하면 더 Good
            public static class Key {
                Key() { }
            }
        		
        		// 위조 불가능한 고유 키 생성
            public static Key getKey() {
                return new Key();
            }
        
            public static void set(Key key, Object value);
            public static Object get(Key key);
        }
        
        ```
        
        - key를 ThreadLocal의 참조값으로 바꿔버리면 없앨 수 있는 코드가 많아짐.
        
        ```jsx
        public final class ThreadLocal {
        	public ThreadLocal();
        	public void set(Object value);
        	public Object get();
        }
        ```
        
        - ThreadLocalMap의 key를 ThreadLocal 객체 그 자체로 사용하면 key 전용 클래스가 필요가 없어짐.
        - 위의 코드(책 코드)에서는 ThreadLocal은 단순히 ThreadLocalMap에 값을 저장해줄 메서드(set, get)을 구현하기 위한 헬퍼 클래스였음.
            
            → private 생성자로 ThreadLocal 생성 불가능하게 막아놓고, set과 get은 static으로 선언했음.
            
        - ThreadLocalMap의 key를 ThreadLocal 객체 자체로 사용하므로, ThreadLocal의 생성자를 public으로 바꾸고 set과 get도 인스턴스 메서드로 바꿈.
        
        ```jsx
        public final class ThreadLocal<T> {
            public ThreadLocal() { }
            public void set(T value) { }
            public T get() {}
        }
        ```
        
        - 위의 코드에 제네릭을 사용해 타입 안전하게 리팩토링한 코드