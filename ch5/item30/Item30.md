# Item 30. 이왕이면 제네릭 메서드로 만들라

- 로 타입은 지양
    
    ```jsx
    // 컴파일은 되지만, 타입 안전하지 않음.
    public static Set union(Set s1, Set s2){
    	Set result = new HashSet(s1);
    	result.addAll(s2);
    	return result;
    }
    	
    // 위 코드를 개선한 버전
    public static <E> Set<E> union(Set<E> s1, Set<E> s2){
    	Set<E> result = new HashSet<>(s1);
    	result.addAll(s2);
    	return result;
    }
    ```
    
    → Set 내에 어떤 타입이던 들어갈 수 있으므로, 런타임 오류 발생 가능성 높음
    
- 제네릭 싱글턴 팩토리
    
    ```jsx
    // JDK 내부 코드 일부
    public class Collections {
        private static final Set EMPTY_SET = new EmptySet();  // 불변 빈 Set
        
        @SuppressWarnings("unchecked")
        public static <T> Set<T> emptySet() {
            return (Set<T>) EMPTY_SET;  // 같은 객체를 다른 타입으로 반환
        }
        
        static final ReverseComparator REVERSE_ORDER
                = new ReverseComparator();
                
        @SuppressWarnings("unchecked")
        public static <T> Comparator<T> reverseOrder() {
            return (Comparator<T>) ReverseComparator.REVERSE_ORDER;
        }
    }
    
    private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;
    
    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction(){
    	return (UnaryOperator<T>) IDENTITY_FN;
    }
    
    ```
    
    - 불변 객체로 로타입을 선언하고, 반환할 때는 불변 객체를 캐스팅해 매번 새로운 객체를 생성하지 않도록 함.
    - 비슷하게, <Object>를 활용해 제네릭 싱글턴 팩토리 적용 가능
    
- 재귀적 타입 한정
    - 제네릭 없는 버전의 comparable
        
        ```jsx
        class Point implements Comparable {
            int x, y;
            
            Point(int x, int y) {
                this.x = x;
                this.y = y;
            }
            
            @Override
            public int compareTo(Object o) {
                Point other = (Point) o;  // 캐스팅 필수!
                int result = Integer.compare(this.x, other.x);
                if (result == 0) {
                    return Integer.compare(this.y, other.y);
                }
                return result;
            }
        }
        
        // 사용
        Point p1 = new Point(1, 2);
        Point p2 = new Point(3, 4);
        
        p1.compareTo(p2);        // OK
        p1.compareTo("String");  // 컴파일 OK, 런타임 💥 ClassCastException
        p1.compareTo(123);       // 컴파일 OK, 런타임 💥 ClassCastException
        
        // 컬렉션에서 사용
        List<Point> points = new ArrayList<>();
        points.add(new Point(5, 5));
        points.add(new Point(1, 1));
        Collections.sort(points);  // ⚠️ 경고 발생 (unchecked)
        ```
        
    
    - 제네릭 사용한 버전의 comparable - 런타임 오류 발생 X, 컴파일 오류
        
        ```jsx
        class Point implements Comparable<Point> {
            int x, y;
            
            Point(int x, int y) {
                this.x = x;
                this.y = y;
            }
            
            @Override
            public int compareTo(Point other) {  // Point 타입 직접 받음, 캐스팅 불필요!
                int result = Integer.compare(this.x, other.x);
                if (result == 0) {
                    return Integer.compare(this.y, other.y);
                }
                return result;
            }
        }
        
        // 사용
        Point p1 = new Point(1, 2);
        Point p2 = new Point(3, 4);
        
        p1.compareTo(p2);        // OK
        p1.compareTo("String");  // ❌ 컴파일 에러 - 미리 막힘!
        p1.compareTo(123);       // ❌ 컴파일 에러 - 미리 막힘!
        
        // 컬렉션에서 사용
        List<Point> points = new ArrayList<>();
        points.add(new Point(5, 5));
        points.add(new Point(1, 1));
        Collections.sort(points);  // 경고 없음, 안전
        ```
        
    
    - 제네릭 없는 버전의 max 메서드 구현
        
        ```jsx
        public static Comparable max(Collection c) { 
            Comparable result = null;
            for (Object o : c) {
                Comparable e = (Comparable) o;  // 캐스팅!
                if (result == null || e.compareTo(result) > 0) {
                    result = e;
                }
            }
            return result;
        }
        
        List points = new ArrayList();  // 로타입
        points.add(new Point(1, 1));
        points.add("String");  // 💥 섞여도 컴파일 OK
        Point max = (Point) max(points);  // 런타임 에러!
        ```
        
    
    - 제네릭 사용 버전의 max 메서드 구현
        
        ```jsx
        public static <E extends Comparable<E>> E max(Collection<E> c) {
            E result = null;
            for (E e : c) {
                if (result == null || e.compareTo(result) > 0) {  // 캐스팅 불필요
                    result = e;
                }
            }
            return result;
        }
        
        List<Point> points = new ArrayList<>();
        points.add(new Point(1, 1));
        points.add(new Point(5, 5));
        points.add("String");  // ❌ 컴파일 에러 - 미리 막힘!
        
        Point max = max(points);  // 캐스팅 불필요, 타입 안전
        System.out.println(max.x + ", " + max.y);  // 5, 5
        ```
        
        → 여기서 재귀적 타입 한정이란 `<E extends Comparable<E>>` 를 말한다. 의미는 Comparable 인터페이스를 구현한 객체들을 타입 매개변수로 받겠다는 의미다. 즉, 같은 타입끼리 비교가 가능한 객체들만 매개변수로 받고 그렇지 않다면 컴파일 오류를 발생시키겠다는 의미다.