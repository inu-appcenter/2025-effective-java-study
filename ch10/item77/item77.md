# item 77

## 👻 예외를 무시하지 말라

---

API 설계자가 예외를 명시하는 까닭은, 그 메서드를 사용할때 적절한 조치를 취해달라고 말하는 것이다.

→ 🚫 API 설계자의 목소리를 흘려버리지 말자.

```java
// catch 블록을 비워두면 예외가 무시된다. 아주 의심스러운 코드다!
try {
		...
} catch (SomeException e) {
}
```

예외는 문제 상황에 잘 대처하기 위해 존재하는데 **catch** 블록을 비워두면 예외가 존재할 이유가 없어진다.

📚 책에서는 이런 말을 한다.

> 빈 catch 블록을 목격한다면 여러분 머릿속에 사이렌을 울려야 한다.
> 

✅ 물론 예외를 무시해야 할 때도 있다.

ex) Fileinputstream을 닫을 때

→ (입력 전용 스트림이므로) 파일의 상태를 변경하지 않았으니 복구할 것이 없으며

→ (스트림을 닫는다는 건) 필요한 정보는 이미 다 읽었다는 뜻이니 남은 작업을 중단할 이유도 없다.

> 예외를 무시하기로 했다면 **catch** 블록 안에 그렇게 결정한 이유를 주석으로 남기고,
예외 변수의 이름도 **ignored**로 바꿔놓도록 하자.
> 

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // 기본값. 어떤 지도라도 이 값이면 충분하다.
try {
		numColors = f.get(lL, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
		// 기본값을 사용한다(색상 수를 최소화하면 좋지만, 필수는 아니다).
}
```

해당 내용은 검사와 비검사 예외에 똑같이 적용된다.

즉, 예외를 무시하거나 놓치지 말자.