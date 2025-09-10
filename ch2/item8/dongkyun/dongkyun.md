# item8

## 🧻 finalizer와 cleaner 사용을 피하라

---

- 자바는 두 가지 객체 소멸자를 제공합니다.
    - **finalizer**는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요합니다.
        - 오동작, 낮은 성능, 이식성 문제의 원인이 되기도 합니다.
        - 기본적으로 쓰지 말아야합니다.
    - **cleaner**는 finalizer보다는 덜 위험하지만,
      여전히 예측할 수 없고, 느리고, 일반적으로 불필요합니다.

    - 이 둘은 즉시 수행된다는 보장이 없습니다.
        - 즉, finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.
        - 이를 얼마나 실행할 지는, 가비지 컬렉터 알고리즘에 달렸으며,
        가비지 컬렉터 구현마다 천차만별입니다.
    
    - 또한 이 둘은 수행 여부조차 보장하지 않습니다.
        - 따라서 프로그램 생애주기와 상관없는, 상태를 영구적으로 수정하는 작업 에서는 절대 finalizer나 cleaner에 의존해서는 안 됩니다.
    
    - finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료됩니다.
    - 그나마 cleaner는 자신의 스레드를 통제해 더 양호합니다.

- finalizer와 cleaner는 **심각한 성능 문제**도 동반합니다
    - 안전망을 설치하는 대가로 성능이 약 5배 정도 느려집니다.

- finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있습니다.
    - 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 됩니다.
    - 허용되지 않은 작업을 수행하는 것은 매우 간단합니다.

  → 객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만,
  finalizer가 있다면 그렇지도 않습니다.

  → final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 **finalize** 메서드를 만들고 final로 선언해야 합니다.


- 이를 대신할 묘안은 무엇인가요?
    - **AutoCloseable를 구현하고 클라이언트에서 인스턴스를 다 쓰고 난 후 close 메서드를 호출하면 됩니다.**
        - 각 인스턴스는 자신이 닫혔는지를 추적할 필요가 있습니다.

- 그럼 도대체 cleaner와 finalizer는 어디에 쓰는 물건인가요..?
    1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망입니다.
        - 자바 라이브러리의 일부 클래스는 안전망 역할의 finalizer를 제공합니다.
          Fileinputstream, FileOutputStream, ThreadPoolExecutor가 대표적입니다.
    2. 네이티브 피어와 연결된 객체에서입니다.
        - 네이티브 피어는 자바 객체가 아니라 가비지 컬렉터가 존재 파악을 할 수 없습니다.
        - 즉 cleaner나 finalizer가 필요합니다. → (성능 저하가 감당 될 경우에만)

- 네이티브 피어란?
    - 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체입니다.

```java
// 잘 짜여진 클라이언트 코드
// 안녕 출력 후 방청소 출력
public class Adult {
		public static void main(String[] args) {
				try (Room myRoom = new Room(7)) {
						System. out. p rintln ("안녕〜,,);
				}
		}
}
```

```java
// 방 청소는 출력되지 않음
// System.exit을 호출할 때의 cleaner 동작은 구현하기 나름이다. 청소가 이뤄질지는 보장
// 하지 않는다
public class Teenager {
		public static void main(String[] args) {
				new Room(99);
				System, out • print In (" 아무렴,,);
		}
}
```

- cleaner(자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수
  용으로만 사용해야합니다.
  물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.