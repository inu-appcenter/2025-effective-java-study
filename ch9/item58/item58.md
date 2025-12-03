# item 58

## ⭐️ 전통적인 for 문보다는 for-each 문을 사용하라

---

```java
코드 58-1 컬렉션 순회하기 - 더 나은 방법이 있다.
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
		Element e = i.next();
		... // e로 무언가를 한다.
}
```

```java
코드 58-2 배열 순회하기 - 더 나은 방법이 있다.
		for (int i = 0; i < a.length; i++) {
		... // a[i]로 무언가를 한다.
}
```

모두 while 보다는 낫지만 가장 좋은 방법은 아니다.
→ 반복자, 인덱스 변수는 코드를 지저분하게 하며, 오류 가능성이 높아진다.

### 👍🏻 for-each을 사용하자

정식 이름은 “향상된 for 문(enhanced for statement)”이다.

```java
코드 58-3 컬렉션과 배열을 순회하는 올바른 관용구
		for (Element e : elements) {
		... // e로 무언가를 한다.
}
```

코드가 깔끔하고 오류가 날 일도 없다.

또한, 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어서
어떤 컨테이너를 다루는지는 신경 쓰지 않아도 된다.

→ 콜론(：)은 “안의(in)”라는 뜻이다.

→ 반복 대상이 컬렉션이든 배열이든, for-each 문을 사용해도 속도는 그대로다.

### 🗄️ 컬렉션 중첩

컬렉션을 중첩해 순회해야 한다면 for-each 문의 이점이 더욱 커진다.

```java
코드 58-4 버그를 찾아보자.
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
						NINE, TEN, JACK, QUEEN, KING }
...

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
		for (Iterator<Rank> j = ranks. iterator(); j .hasNext(); )
				deck.add(new Card(i.next(), j.next()));
```

⚠️ 문제는 바깥 컬렉션(suits)의 반복자에서 next 메서드가 너무 많이 불린다는 것이다.

마지막 줄의 i.next()는 ‘숫자(Suit) 하나당’ 한 번씩만 불려야 하는데,
안쪽 반복문에서 호출되는 바람에 ‘카드(Rank) 하나당’ 한 번씩 불리고 있다.

→ 숫자가 바닥나면 반복문에서 `NoSuchElementException`을 던진다.

```java
코드 58-5 같은 버그, 다른 증상!
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
... 
Collection<Face> faces = EnumSet.allOf(Face.class);

for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
		for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
				System.out.p rintIn(i.next() + " n + j.next());
```

→ 이 프로그램은 예외를 던지진 않지만, 가능한 조합을 (“ONE ONE”부터 “SIX SIX”까지)
단 여섯 쌍만 출력하고 끝나버린다. (36개 조합이 나와야 한다)

```java
코드 58-6 문제는 고쳤지만 보기 좋진 않다. 더 나은 방법이있다.
// 바깥 원소를 저장하는 변수를 하나 추가해야 한다.
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
		Suit suit = i.next();
		for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
				deck.add(new Card(suit, j.next()));
}
```

→ 이는 for-each로 간단히 해결된다.

```java
코드 58-7 컬렉션이나 배열의 중첩 반복을 위한 권장 관용구
for (Suit suit : suits)
		for (Rank rank : ranks)
				deck.add(new Card(suit, rank));
```

### ❌ for-each를 사용하지 못하는 상황

1. `파괴적인 필터링`(destructive filtering):
   컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다.
2. `변형(`transforming):
   리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면
   리스트의 반복자나 배열의 인덱스를 사용해야 한다.
3. `병렬 반복`(parallel iteration)
   여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해
   엄격하고 명시적으로 제어해야 한다.

→ 세 가지 상황 중 하나에 속할 때는 일반적인 for 문을 사용하되, 위에서 언급한 문제들을 경계해야 한다.

### 💬 Iterable

for-each 문은 컬렉션과 배열은 물론 `Iterable 인터페이스`를 구현한 객체라면 무엇이든 순회할 수 있다.

원소들의 묶음을 표현하는 타입을 작성해야 한다면 Iterable을 구현하는 쪽으로 고민해보기 바란다.

📁 정리

전통적인 for 문과 비교했을 때,
for-each 문은 명료하고, 유연하고, 버그를 예방해준다. 또한 성능 저하도 없다.

✅ 가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자.