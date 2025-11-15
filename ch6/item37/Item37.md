# Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라

- ordinal()은 Enum 타입 상수의 인덱스를 반환한다. 그러나 상수의 순서가 바뀌면 인덱스도 바뀔 수 있기에 ordinal()을 사용하지 않는 것을 권장한다.

- 아래의 클래스를 예시로, ordinal() 사용과 EnumMap 사용을 비교해보자.
    
    ```jsx
    Class Plant {
    	enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL }
    	
    	final String name;
    	final LifeCycle lifeCycle;
    	
    	Plant(String name, LifeCycle lifeCycle) {
    		this.name = name;
    		this.lifeCycle = lifeCycle;
    	}
    	
    	@Override public String toString() {
    		return name;
    	}
    	
    	// 책에는 안나와있음
    	Plant[] garden = {
        new Plant("Basil", Plant.LifeCycle.ANNUAL),
        new Plant("Rose", Plant.LifeCycle.PERENNIAL),
        new Plant("Carrot", Plant.LifeCycle.BIENNIAL)
    	};
    }
    ```
    

- ordinal()을 배열 인덱스로 사용
    
    ```jsx
    // 제네릭 배열을 만들기 위해 로타입 Set 배열을 생성
    // 생성된 배열에 Set<Plant>[]로 형변환
    Set<Plant>[] plantsByLifeCycle =
    	(Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
    
    // 각 배열의 요소마다 HashSet 생성
    for (int i = 0; i < plantsByLifeCycle.length; i++)
    	plantsByLifeCycle[i] = new HashSet<>();
    
    // garden의 식물들을 생애주기에 따라 분류
    for (Plant p : garden)
    	plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
    
    //values()는 Enum값 배열로 리턴하는 함수 
    for (int i = 0; i < plantsByLifeCycle.length; i++) {
    	System.out.printf("%s: %s%n",
    		Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
    }
    ```
    
    - 비검사 형변환 수행, 컴파일 깔끔하게 X
    - `Plant.LifeCycle.values()[i]` 같이 수동으로 레이블을 달아야 함
    - 정확한 정숫값을 사용한다는 것을 사용자가 보증해야 함?
        - 배열의 인덱스를 넘어서는 정숫값을 사용하는 경우 ArrayIndexOutOfBoundsException 발생
            
            → 배열 생성 길이를 하드코딩한 경우
            
            → enum과 생성된 배열이 서로 분리된 모듈에 존재할 때 enum은 수정해서 재컴파일했는데,  생성된 배열을 사용하는 클라이언트 측은 재컴파일하지 않고 그대로 사용하고 있는 경우 등
            
        - 또는 Enum값의 순서가 바뀌어서 기존의 배열의 인덱스와 달라진다면 런타임 오류 발생 가능성 높음
    
- ordinal() 사용하지 않고 EnumMap으로 개선한 버전
    
    ```jsx
    Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
            new EnumMap<>(Plant.LifeCycle.class);
    
    // map인 plantsByLifeCycle에 생애주기마다 HashSet 삽입
    for (Plant.LifeCycle lc : Plant.LifeCycle.values())
        plantsByLifeCycle.put(lc, new HashSet<>());
    
    // 생애주기에 맞게 식물을 Set에 삽입
    for (Plant p : garden)
        plantsByLifeCycle.get(p.lifeCycle).add(p);
    
    System.out.println(plantsByLifeCycle);
    
    ```
    
    - 컴파일 깔끔하게 됨
    - 키가 Enum타입 그 자체이므로, 출력할 때 레이블을 직접 달아줄 필요 X
    - 배열 인덱스를 계산하는 과정에서 오류가 날 가능성 X
    
    🤷‍♂️ Map의 타입 매개변수로 LifeCycle을 받는데, EnumMap의 인자로 LifeCycle.Class를 받는 이유?
    
    → 제네릭은 런타임에는 타입이 사라짐, LifeCycle은 Enum.Class로 보임.
    
    → 때문에 런타임에 모든 Enum값을 얻으려면 클래스 정보를 직접 전달해줘야 함. 
    
    🤷‍♂️ EnumMap의 내부 동작?
    
    ```jsx
    // EnumMap 코드 일부
    public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V>
        implements java.io.Serializable, Cloneable
    {
    		// Enum Class 타입
        private final Class<K> keyType;
        // Enum 상수값 배열
        private transient K[] keyUniverse;
        // ordinal()을 통해 얻은 Enum 상수값의 인덱스
        private transient Object[] vals;
        // Map의 크기
        private transient int size = 0;
        ...
    }
    ```
    
    → KeyUniverse[0](Enum 상수값)의 ordinal() 결과값은 vals[0]에 저장됨.
    
    → 내부적으로 ordinal()을 사용하지만, 상수값들의 순서가 바뀌어도 같은 인덱스에서의 KeyUniverse, vals는 서로 키와 값을 의미함. 사용자는 순서를 신경쓸 필요 X
    
    ```jsx
        // EnumMap 코드 일부
        public V put(K key, V value) {
            typeCheck(key);
    
            int index = key.ordinal();
            Object oldValue = vals[index];
            vals[index] = maskNull(value);
            if (oldValue == null)
                size++;
            return unmaskNull(oldValue);
        }
    ```
    
    → 내부적으로 ordinal() 사용하는 것을 확인할 수 있음.
    
    ```jsx
        // EnumMap 생성자
        public EnumMap(Class<K> keyType) {
            this.keyType = keyType;
            keyUniverse = getKeyUniverse(keyType);
            vals = new Object[keyUniverse.length];
        }
    ```
    
    ```jsx
      /**
         * Returns all of the values comprising K.
         * The result is uncloned, cached, and shared by all callers.
         */
        private static <K extends Enum<K>> K[] getKeyUniverse(Class<K> keyType) {
            return SharedSecrets.getJavaLangAccess()
                                            .getEnumConstantsShared(keyType);
        }
    ```
    
    ```jsx
         /**
         * Returns the elements of an enum class or null if the
         * Class object does not represent an enum type;
         * the result is uncloned, cached, and shared by all callers.
         */
        <E extends Enum<E>> E[] getEnumConstantsShared(Class<E> klass);
    ```
    
    → 쉽게 말하자면 getEnumConstantsShared는 인자로 받은 Enum 클래스에서 모든 상수값을 배열로 반환하는 메서드임. EnumMap의 생성자 인자로 Enum클래스를 받으면 이렇게 사용됨.
    

- Map 관리를 Stream으로?
    
    ```jsx
    System.out.println(Arrays.stream(garden)
    	.collect(groupingBy(p -> p.lifeCycle)));
    ```
    
    - EnumMap 사용 X, 기본적인 HashMap 사용 → 해시 계산 등 불필요한 연산 & 공간 낭비 발생
    - Value도 List임
    
    ```jsx
    System.out.println(Arrays.stream(garden)
    	.collect(groupingBy(p -> p.lifeCycle,
    		() -> new EnumMap<>(LifeCycle.class), toset())));
    ```
    
    - EnumMap 사용, Value도 Set임
    - EnumMap만 사용했을 때와는 달리, groupingBy를 통해 실제 있는 lifeCycle에 대한 엔트리만 생성
        
        → 한해살이, 두해살이, 여러해살이가 Enum 상수인데 두해살이인 객체가 없다면 엔트리는 2개만 생성
        

- 고체 - 액체 - 기체 상태 전이 매핑 프로그램 ordinal() 사용
    
    ```jsx
    public enum Phase {
        SOLID, LIQUID, GAS;
        
        public enum Transition {
            MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
            
            private static final Transition[][] TRANSITIONS = {
                { null,    MELT,     SUBLIME  },
                { FREEZE,  null,     BOIL     },
                { DEPOSIT, CONDENSE, null     }
            };
            
            public static Transition from(Phase from, Phase to) {
                return TRANSITIONS[from.ordinal()][to.ordinal()];
            }
        }
    }
    ```
    
    - EnumMap을 사용하지 않고 ordinal()을 사용한 첫 예시와 똑같은 문제점이 있음.
        
        → 새로운 상태의 물질이 발견되고, 전이 상태도 추가된다면 런타임 오류 가능성 높음
        

- 고체 - 액체 - 기체 상태 전이 매핑 프로그램 EnumMap 사용
    
    ```jsx
    public enum Phase {
        SOLID, LIQUID, GAS;
        
        public enum Transition {
            MELT(SOLID, LIQUID), 
            FREEZE(LIQUID, SOLID),
            BOIL(LIQUID, GAS), 
            CONDENSE(GAS, LIQUID),
            SUBLIME(SOLID, GAS), 
            DEPOSIT(GAS, SOLID);
            
            private final Phase from;
            private final Phase to;
            
            Transition(Phase from, Phase to) {
                this.from = from;
                this.to = to;
            }
            
            private static final Map<Phase, Map<Phase, Transition>> m =
                Stream.of(values())
                    .collect(groupingBy(
                        t -> t.from,
                        () -> new EnumMap<>(Phase.class),
                        toMap(
                            t -> t.to,
                            t -> t,
                            (x, y) -> y, 
                            () -> new EnumMap<>(Phase.class)
                        )
                    ));
            
            public static Transition from(Phase from, Phase to) {
                return m.get(from).get(to);
            }
        }
    }
    ```
    
    - 새로운 상수가 추가돼도 사용자는 순서를 고려할 필요가 없음.
    - toMap의 (x, y) → y는 중복된 key를 가질 때 어떤 value를 선택할 지 처리하는 부분이지만, 여기에서는 중복된 key를 가질 경우가 없음. 같은 key를 가져도 내부 map에 엔트리가 추가되는 것임.
        
        → 단지 EnumMap을 사용하기 위해 점층적 팩토리 방식을 사용한 것
        

결론 - ordinal 잘 쓴 EnumMap 있으니까 웬만하면 ordinal 직접 쓸 생각 하지 말아라