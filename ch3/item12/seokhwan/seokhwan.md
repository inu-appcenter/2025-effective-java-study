# Item 12

추가 일시: 2025년 9월 13일 오전 3:00
강의: Effective JAVA

## 🍀 Item 12: toString을 항상 재정의하라

---

Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다.

때문에 toString의 일반 규약에 따라 ‘간결하면서 사람이 읽기 쉬운 형태의 유익한 정보’를 반환하기 위해 toString 메서드 재정의를 고려해야한다.

toString 메서드 재정의를 통해 메서드 사용의 편의성과, 디버깅에 이점을 줄 수 있다.

그러기 위해서는 toString은 해당 객체가 가진 주요 정보를 반환하는 것이 좋다.

### toString 메서드와 문서화

---

toString을 구현할 때 반환값의 포맷을 문서화할지 정해야 한다.

 문서화 여부에 따른 장단점이 존재한다.

**포맷을 명시하는 경우**

```java
/**
 * 이 전화번호의 문자열 표현을 반환한다.
 * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
 * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
 */
@Override
public String toString() {
    return String.format("%03d-%03d-%04d", 
                        areaCode, prefix, lineNum);
}
```

장점 - 표준적, 명확함, 파싱 가능

단점 - 포맷에 얽매임

**포맷을 명시하지 않는 경우**

```java
/**
 * 이 약물에 관한 대략적인 설명을 반환한다.
 * 다음은 이 설명의 일반적인 형태이나,
 * 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
 * 
 * "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
 */
@Override
public String toString() { ... }
```

장점 - 향후 개선에 유연

단점 - 파싱하기 어려움

toString 재정의시 접근 메서드를 제공해야한다.

```java
// toString 정보를 얻기 위한 별도 접근자 제공
public int getAreaCode() { return areaCode; }
public int getPrefix() { return prefix; }
public int getLineNum() { return lineNum; }
```

유지 보수성을 올려주고 향후 포맷을 바꾸더라도 시스템이 망가지지 않는다.

### toString을 재정의하지 않아도 되는 경우

---

**정적 유틸리티 클래스**

인스턴스가 없으니 출력할 일도 없다.

**열거 타입**

Java에서 이미 완벽한 toString을 제공한다.

상위 클래스에서 이미 적절히 재정의한 경우에도 필요 없다.

→ 물론 특별한 요구사항에 따라 재정의 할 수 있다.