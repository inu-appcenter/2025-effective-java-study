# Item 8. finalizer와 cleaner 사용을 피하라

- gc는 실행되는 동안 다른 작업들을 멈추기 때문에(stop-the-world) 성능이 좋지 않다는 단점이 있다. 때문에 자주 실행되지 않는다. System.gc()를 명시적으로 실행해도 gc 실행 시점을 정확히 알 수 없는 데다가, 네이티브 자원 연결은 굉장히 제한적이어서 gc에 의존하면 금방 한계에 도달할 수 있다.

- finalizer는 소켓, DB connection, FileInputStream 등 네이티브 자원 연결을 자동으로 끊어주는 역할을 한다. finalize 스레드는 다른 어플리케이션 스레드보다 우선순위가 낮아 실행될 기회를 얻지 못하는 경우가 생긴다. cleaner는 finalizer 팬텀 참조를 이용해 보안 문제를 개선한 것이지만 여전히 성능 문제 등 단점이 많다.

- finalizer와 cleaner 대신 AutoCloseable과 try-with-resource, close()를 사용해야 한다. 자바 라이브러리에서는 사용자(개발자)가 수동으로 실행하지 않는 경우를 대비해 finalizer나 cleaner를 사용하는 경우도 있다.

- 추가적으로, 네이티브 자원은 JVM 힙에 올라가지 않으므로, 가비지 컬렉션의 대상이 아니다. 가비지 컬렉션의 대상은 네이티브 자원과의 연결을 가지고 있는 객체다.

- finalizer 공격 ?

https://inpa.tistory.com/entry/JAVA-%E2%98%95-JVM-%EB%82%B4%EB%B6%80-%EA%B5%AC%EC%A1%B0-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%98%81%EC%97%AD-%EC%8B%AC%ED%99%94%ED%8E%B8?category=976278#%EB%9F%B0%ED%83%80%EC%9E%84_%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%98%81%EC%97%AD_runtime_data_area

https://devfunny.tistory.com/921#google_vignette