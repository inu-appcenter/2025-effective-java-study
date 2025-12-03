# Item 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

- Serializable을 사용하면 버그와 보안 문제가 일어날 가능성이 커짐. 때문에 직렬화 프록시 패턴을 사용해야 함.
    
    ```jsx
    public final class Period implements Serializable {
        private static final long serialVersionUID = 234098243823485285L;
    
        private final Date start;
        private final Date end;
    
        public Period(Date start, Date end) {
    		    // readObject를 그대로 사용했다면 아래의 조건에 맞지 않는 객체라도 생성 가능
            if (start.after(end)) {
                throw new IllegalArgumentException(start + " after " + end);
            }
            this.start = new Date(start.getTime());
            this.end = new Date(end.getTime());
        }
    
        public Date start() { return new Date(start.getTime()); }
        public Date end()   { return new Date(end.getTime()); }
    
        private Object writeReplace() {
            return new SerializationProxy(this);
        }
    
        private void readObject(ObjectInputStream stream)
                throws InvalidObjectException {
            throw new InvalidObjectException("Serialization proxy required");
        }
    
        private static class SerializationProxy implements Serializable {
            private static final long serialVersionUID = 234098243823485285L;
    
            private final Date start;
            private final Date end;
    
            SerializationProxy(Period p) {
                this.start = p.start;
                this.end = p.end;
            }
    
            private Object readResolve() {
                return new Period(start, end);
            }
        }
    }
    
    ```
    
    - writeReplace(), readObject(), readResolve()는 구현해두면 JVM이 자동으로 직렬화/역직렬화를 거칠 때 호출하는 메서드들임.
        - writeReplace()는 직렬화될 객체를 다른 객체로 바꿔주는 메서드, 직렬화 직전에 호출됨.
        - readObject()는 역직렬화를 수행하는 메서드임.
        - readResolve()는 역직렬화된 객체 대신 다른 객체를 반환할 수 있는 메서드임.
    
    - period라는 Serializable를 구현한 클래스가 있을 때, readObject를 그대로 사용한다면 직렬화 프록시가 있을 이유가 없으므로, 명시적으로 readObject를 사용하지 못하게 예외를 던지도록 구현함.
        
        → period의 private 메서드
        
        → 위 객체에서는 JVM이 readObject를 호출할 일이 없음. 역직렬화되어 들어온 객체는 proxy이고, proxy 내부에서 default readObject → readResolve를 호출하므로 생성자를 통해 period가 생성됨.
        
    
    - writeReplace()는 객체를 직렬화할 때 period 객체 자신이 아니라 proxy 객체를 직렬화하도록 함.
        
        → period의 private 메서드
        
    
    - readResolve()는 객체를 역직렬화할 때 proxy 객체가 아니라 period 객체를 생성자를 통해 생성함.
        
        → proxy의 private 메서드
        
        → 직렬화를 proxy로 하므로, 역직렬화 될 객체로 proxy, readResolve()는 proxy의 메서드여야 함.
        
    
    - Proxy는 일종의 DTO라고 생각하면 됨. User 객체가 있다고 할 때, 프론트단으로 User 객체를 던져주지는 않음.
        - period를 직렬화한다면 값 뿐만 아니라 메서드 등 내부 구조도 드러남.
        - period를 직렬화한다면 API로 공개되는 순간부터 period를 수정하는게 곤란해짐.
        
    - 가짜 바이트스트림 공격, 내부 필드 탈취 공격 등 차단 가능,  불변성(Immutability X invariant → 객체가 만족해야 하는 조건/규칙) 보장 가능
    
    - 다만 바이트 스트림을 조작해 값을 변경하는 건 막을 수 없음 .. 근데 이건 JSON이던 바이트스트림이던 데이터 전달 과정에서 값 조작하는 건 다른 차원의 문제

- 직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 작동
    - EnumSet 내부 Proxy 구조
        
        ```jsx
        private static class SerializationProxy<E extends Enum<E>>
                implements Serializable {
        
            private static final long serialVersionUID = 362491234563181265L;
        
            private final Class<E> elementType;
            private final Enum<?>[] elements;
        
            SerializationProxy(EnumSet<E> set) {
                this.elementType = set.elementType;
                this.elements = set.toArray(new Enum<?>[0]);
            }
        
            private Object readResolve() {
        		    // noneOf를 통해 element 개수를 확인하고, Jumbo / regular EnumSet 결정
                EnumSet<E> result = EnumSet.noneOf(elementType);
                for (Enum<?> e : elements) {
                    result.add((E) e);
                }
                return result;
            }
        }
        
        // openjdk-23 EnumSet 내부 코드
        public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
                Enum<?>[] universe = getUniverse(elementType);
                if (universe == null)
                    throw new ClassCastException(elementType + " not an enum");
        
                if (universe.length <= 64)
                    return new RegularEnumSet<>(elementType, universe);
                else
                    return new JumboEnumSet<>(elementType, universe);
            }
        ```
        
    - 직렬화된 인스턴스의 클래스가 regularEnumSet → Proxy라고 해도, 역직렬화할 때 element의 개수가 다르면 JumboEnumSet 반환함.