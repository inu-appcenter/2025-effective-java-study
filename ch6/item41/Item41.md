# Item 41. 정의하는 것이 타입이라면 마커 인터페이스를 사용하라

- 아무런 메서드 없이 자신을 구현하는 클래스가 특정 속성을 가진다는 것을 표시해주는 인터페이스를 마커 인터페이스라고 함. 예시로 Serializable이 있음.
    
    ```jsx
    package java.io;
    public interface Serializable {
    }
    ```
    
    → 이 코드가 Serializable의 전부임.
    
- 마커 인터페이스가 마커 애너테이션보다 나은 점이 두 가지 있음.
    1. 마커 인터페이스는 구현한 클래스의 인스턴스를 구분하는 타입으로 사용 가능
        - 마커 인터페이스는 타입이기 때문에, 애너테이션이 런타임에 잡을 오류를 컴파일 타임에 잡을 수 있음.
        - ObjectOutputStream.writeObject 메서드는 마커 인터페이스의 이점을 살리지 못한 설계
            
            ```jsx
                // Object를 받을게 아니라 Serializable을 받았어야 함!
                public final void writeObject(Object obj) throws IOException {
                    if (enableOverride) {
                        writeObjectOverride(obj);
                        return;
                    }
                    try {
                        writeObject0(obj, false);
                    } catch (IOException ex) {
                        if (depth == 0) {
                            writeFatalException(ex);
                        }
                        throw ex;
                    }
                }
            ```
            
        
    2. 적용 대상을 더 정밀하게 지정 가능
        - 어노테이션의 적용 대상(@Target)을 Element.TYPE으로 선언하면 모든 타입에 달 수 있으나, 여기서 더 세밀하게 제한하지는 못함. 특정 인터페이스나 클래스 타입으로 제한 불가능
        - 마커 인터페이스를 이용하면 해당 인터페이스를 implement하기만 하면 됨
        
- 반면 마커 애너테이션은 거대한 애너테이션 시스템의 지원을 받는 점이 마커 인터페이스보다 유리함.

- 클래스와 인터페이스 이외의 요소(모듈, 패키지, 필드, 지역변수 등)에 마킹이 필요하다면 애너테이션을 사용할 수 밖에 없음.
    
    → 근데 마킹이 클래스, 인터페이스에 적용해야하고 마킹된 객체가 매개변수의 타입으로 사용돼야 한다면 마커 인터페이스 써야함
    

결론 - 마커 애너테이션이 활발한 프레임워크에서는 도움이 많이 되지만, 객체의 타입으로 사용해야 하는 경우는 마커 인터페이스를 사용하자