# item15

## 🪪 클래스와 멤버의 접근 권한을 최소화하라

---

**📦 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.**

⚠️ **배열 / 컬렉션 필드 주의
→ public 배열 필드 → 외부에서 변경 가능 → 보안/캡슐화 깨짐**
해결 방법: **불변 리스트** 또는 **방어적 복사**

```java
// 문제 있는 코드
public static final Thing[] VALUES = { ... };

// 해결 방법 1: 불변 리스트
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES =
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

// 해결 방법 2: 방어적 복사
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
```

⛔️ **상속과 접근 제한**

- **오버라이드 시 접근 수준을 좁힐 수 없음**
    - 부모 메서드보다 좁으면 컴파일 오류
      → **리스코프 치환 원칙**
- 인터페이스 구현 시 모든 메서드는 **public**