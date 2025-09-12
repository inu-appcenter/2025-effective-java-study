# Item 7. 다 쓴 객체 참조를 해제하라

- 객체 참조의 종류
    1. 강한 참조
        - 기본적으로 `new`를 통해 생성되는 참조, null처리를 해주지 않으면 지워지지 않는다. `OutOfMemory`가 발생할 수 있다.
        
        ```jsx
        Object o = new Object();
        ```
        
    2. 소프트 참조
        - `SoftReference<T>` 객체로 감싼 참조, 메모리가 충분할 때는 안 지우고, 메모리가 부족해지면 수거 대상으로 삼는다. 캐시를 구현하는 데 쓰인다.
        
        ```jsx
        SoftReference<byte[]> ref = new SoftReference<>(new byte[1024*1024]);
        ```
        
    
    1. 약한 참조
        - `WeakReference<T>` 객체로 감싼 참조, GC가 돌면 무조건 수거 대상으로 삼는다.
        - `WeakHashMap` → 키를 약한 참조로 들고 있어서, 키 객체가 더 이상 강하게 참조되지 않으면 자동으로 엔트리 삭제.
        
        ```jsx
        WeakReference<Object> ref = new WeakReference<>(new Object());
        ```
        
    
    1. 팬텀 참조
        - `cleaner`를 구현하는 기반이 된다.
        - 객체가 GC 대상이 될 때 ReferenceQueue에 enqueue된다. ReferenceQueue를 통해 GC 대상이 될 때 객체가 사라진다고 일종의 알림을 남기는 것이다.
            - GC와 cleaner 동작 순서: stw → 참조 끊긴 객체 탐색 → 참조 끊긴 객체들 ReferenceQueue에 Enqueue → stw 해제 → 네이티브 자원 연결 close → 객체 삭제
        - get으로 팬텀 객체를 참조하면 늘 null이 된다. 객체가 부활할 위험도 없다.
        
        ```jsx
        Connection conn = ds.getConnection(); // 강한참조
        ReferenceQueue<Object> queue = new ReferenceQueue<>();
        PhantomReference<Object> ref = new PhantomReference<>(conn, queue); // 팬텀 참조 등록
        ```
        

https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC