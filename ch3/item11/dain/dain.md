# Item11

<aside>

equals를 재정의하려거든 hashCode도 재정의하

</aside>

### 1. Why?

equals를 재정의한 클래스에서 hashCode를 재정의하지 않으면 해시 기반 컬렉션에서 문제가 발생한다. HashMap, HashSet 같은 컬렉션들이 제대로 동작하지 않아서 찾을 수 있는 객체를 못 찾거나 중복이 허용되는 상황이 벌어진다.

```java
public class PhoneNumber {
    private int areaCode, prefix, lineNum;
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof PhoneNumber)) return false;
        PhoneNumber pn = (PhoneNumber) obj;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }
    
    // hashCode는 재정의하지 않음
}

// 사용 예시
Map<PhoneNumber, String> phoneBook = new HashMap<>();
phoneBook.put(new PhoneNumber(707, 867, 5309), "케로로");

// 같은 값으로 조회하면 "케로로"가 나와야 할 것 같지만...
String name = phoneBook.get(new PhoneNumber(707, 867, 5309));
System.out.println(name); // null 출력 (문제 발생)
```

**왜 null이 나올까… → HashMap의 내부 동작 방식 때문**

- HashMap은 먼저 키의 hashCode()로 해시 버킷을 찾는다
- 그 다음에 equals()로 정확한 키를 찾는다
- 하지만 두 PhoneNumber 객체의 hashCode가 다르면 다른 버킷에서 찾는다
- equals가 true여도 애초에 다른 버킷에서 찾으니까 못 찾는다

(HashMap은 최적화를 위해 해시코드가 다른 엔트리끼리는 equals 비교조차 하지 않음. 따라서 재정의가 필수적.)

### 2. hashCode 일반 규약

1. **일관성**

   객체의 equals 정보가 변경되지 않았다면 몇 번을 호출해도 항상 같은 값을 반환해야 한다.

    ```java
    PhoneNumber phone = new PhoneNumber(707, 867, 5309);
    int hash1 = phone.hashCode();
    int hash2 = phone.hashCode();
    // hash1 == hash2 여야 함 (같은 실행 중에는)
    ```

2. **equals가 같으면 hashCode도 같아야 함**

   이 규약을 어기면 HashMap에서 논리적으로 같은 키를 못 찾는 문제가 발생한다.

    ```java
    PhoneNumber phone1 = new PhoneNumber(707, 867, 5309);
    PhoneNumber phone2 = new PhoneNumber(707, 867, 5309);
    
    if (phone1.equals(phone2)) {
        // 이 경우 반드시 phone1.hashCode() == phone2.hashCode() 여야 함
    }
    ```

3. **equals가 다르면 hashCode가 달라야 성능이 좋음 (권장사항)**

   다른 객체들이 다른 해시코드를 가져야 해시테이블 성능이 좋아진다.

    ```java
    PhoneNumber phone1 = new PhoneNumber(707, 867, 5309);
    PhoneNumber phone2 = new PhoneNumber(707, 867, 5310);
    
    // phone1.equals(phone2)가 false라면
    // phone1.hashCode() != phone2.hashCode() 인 것이 좋음 (필수는 아님)
    ```


### 3. 올바른 hashCode 구현

```java
@Override
public int hashCode() {
    // 첫 번째 필드의 해시코드로 result 초기화
    int result = Integer.hashCode(areaCode);
    
    // 나머지 필드들 처리
    result = 31 * result + Integer.hashCode(prefix);
    result = 31 * result + Integer.hashCode(lineNum);
    //31은 홀쉬면서 소수(정통적으로 소수를 곱함) 
    
    // result 반환
    return result;
}
```

```java
@Override
public int hashCode() {
    return Objects.hash(areaCode, prefix, lineNum);
} // 한 줄로도 가능~ 대신 성능이 떨어짐(배열 생성+박싱/언박싱)
```

### 4. 주의할 점

핵심 필드만 사용하기 → equals에서 사용하지 않는 필드를 hashCode에 포함하면 두 번째 규약을 어길 위험이 있음.

핵심 필드 생략하지 말기 → 아래 예시에서, 같은 지역의 전화번호들이 모두 같은 해시코드를 가져서 해시테이블 성능이 크게 떨어진다. 결국 성능 최적화는 커녕 오히려 성능 저하를 가져온다.

```java
// areaCode만 사용 (성능상 유리할 것을 기대하고...)
@Override
public int hashCode() {
    return Integer.hashCode(areaCode); // prefix, lineNum 생략
}
```

hashCode 생성 규칙을 공개하면 클라이언트가 이에 의존하게 되어 추후 최적화에 어려움이 있다느데..