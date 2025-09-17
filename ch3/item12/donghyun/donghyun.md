# 아이템 12

## toString 재정의의 중요성

**기본 toString의 문제점**

- Object의 기본 toString은 `클래스명@해시코드` 형태로 출력 (예: `PhoneNumber@adbbd`)
- 이는 사람이 읽기엔 유용하지 않음
- `707-867-5309` 같은 실제 정보가 훨씬 유용함

## toString 재정의의 장점

**자동 호출 상황들**

- `System.out.println()`, `printf` 사용 시
- 문자열 연결 연산자(`+`) 사용 시
- `assert` 구문에서
- 디버거가 객체 출력 시
- 오류 로깅 시

**포함할 정보**

- 객체의 주요 정보를 모두 포함
- 객체가 너무 크면 요약 정보 제공
- 스스로를 완벽히 설명하는 문자열이 이상적

```java
- toString 오버라이딩 전

item10.Point@20
item10.Point@40
에러 메시지 :  예외 발생item10.Point@20

------

- toString 오버라이딩 후

@Override
public String toString() {
  return String.format("Point(x=%d, y=%d)", x, y);
}

public static void main(String[] args) {
  Point point1 = new Point(1, 1);
  Point point2 = new Point(2, 2);

  System.out.println(point1);
  System.out.println(point2);

  try {
      throw new RuntimeException("예외 발생" +  point1);
  } catch (Exception e) {
      System.out.print("에러 메시지 :  " + e.getMessage());
  }
}

Point(x=1, y=1)
Point(x=2, y=2)
에러 메시지 :  예외 발생Point(x=1, y=1)
```

```java
@Test
void contextLoads() {
	User user1 = User.builder()
			.name("a")
			.password("a")
			.nickName("a")
			.build();

	User user2 = User.builder()
			.name("a")
			.password("a")
			.nickName("a")
			.build();

	Assertions.assertThat(user1).isEqualTo(user2);
}
  @Override
  public String toString() {
      return "User{" +
              "nickName='" + nickName + '\'' +
              ", name='" + name + '\'' +
              ", password='" + password + '\'' +
              '}';
  }

User{nickName='a', name='a', password='a'}
User{nickName='a', name='a', password='a'}

Expected :User{nickName='a', name='a', password='a'}
Actual   :User{nickName='a', name='a', password='a'}
<Click to see difference>

org.opentest4j.AssertionFailedError: 
expected: "User{nickName='a', name='a', password='a'} (User@5453020f)"
but was: "User{nickName='a', name='a', password='a'} (User@27a57c66)"
```