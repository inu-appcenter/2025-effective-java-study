# 아이템 11

```java
- hashCode 오버라이딩 

class Point {
    protected final int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}

public static void main(String[] args) {
    Point point1 = new Point(1, 1);
    Point point2 = new Point(1, 1);

    System.out.println(point1.equals(point2));

    HashMap<Point, String> map = new HashMap<>();
    map.put(point1, "1");

    String s = map.get(point2);
    System.out.println(s);

    System.out.println(point1.hashCode());
    System.out.println(point2.hashCode());

}

true
null
990368553
1096979270

----

- hashCode 오버라이딩 후

class Point {
    protected final int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }

    @Override
    public int hashCode() {
        return 31 * x + y;
    }
}

Point point1 = new Point(1, 1);
Point point2 = new Point(1, 1);

System.out.println(point1.equals(point2));

HashMap<Point, String> map = new HashMap<>();
map.put(point1, "1");

String s = map.get(point2);
System.out.println(s);

System.out.println(point1.hashCode());
System.out.println(point2.hashCode());

// Set의 중복 방지 기능은 해당 클래스의 hashCode를 기준으로 한다
HashSet<Point> set = new HashSet<>();
set.add(point1);
set.add(point2);

System.out.println(set.size());

true
1
32
32
1
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

System.out.println(user1);
System.out.println(user2);

Assertions.assertThat(user1).isEqualTo(user2);
}

com.example.trip_nest.domain.user.entity.User@4ed7fb78
com.example.trip_nest.domain.user.entity.User@393b99d

Expected :com.example.trip_nest.domain.user.entity.User@393b99d
Actual   :com.example.trip_nest.domain.user.entity.User@4ed7fb78

@Override
public boolean equals(Object o) {
  if (o == null || getClass() != o.getClass()) return false;
  User user = (User) o;
  return Objects.equals(nickName, user.nickName) && Objects.equals(name, user.name) && Objects.equals(password, user.password);
}

@Override
public int hashCode() {
  return Objects.hash(nickName, name, password);
}

성공
```

## hashCode ≠ 메모리 주소

### 1. **hashCode의 실제 정체**

- hashCode는 객체를 식별하기 위한 **정수값**일 뿐
- 실제 메모리 주소와는 **완전히 별개**