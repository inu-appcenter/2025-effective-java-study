# Item54

> ***null이 아닌, 빈 컬렉션이나 배열을 반환하라***

<br>

### 빈 컬렉션이나 빈 배열 반환하기

 아이템에서 먼저 소개한 예시는 **null**을 반환하지 말고, 빈 컬렉션이나 배열을 반환하라고 권고하고 있다. **null**을 반환하게 되면, 이를 호출하는 메서드는 항상 **null-safe**하게 방어 코드를 작성해야 하기 때문에 번거로워지기 때문이다.

<br>

 일부에서는 **null**을 반환하는 것이 메모리 측면에서 더 효율적이라고 주장하기도 하지만, **빈 컬렉션이나 배열도 메모리를 차지하기 때문**이다. 하지만, 실제로 메모리에 미치는 영향은 매우 미미하며, 비어있는 불변 참조를 만들어 반환하면 이런 문제도 충분히 해결할 수 있기 때문이다.

```java
public class A {
	private final List<B> items = new ArrayList<>();
	
	public List<B> getItems(){
		return items.isEmpty() ?
		   Collections.emptyList() : // 싱글톤 반환
		   new ArrayList<>(items); // 방어적 복사 적용
	}
}
```

<br>

배열의 경우에도 절대 **null**을 반환하지 말고, 길이가 0인 배열을 만들어 반환하는 것이 좋다.

<br>

### 그럼 언제 빈 컬렉션을 반환하고, 언제 예외를 던져야 할까요

이건 지극히 나만의 기준이다.

- **빈 컬렉션을 반환해야 하는 경우**
    
    → 반환값이 없어도 되는데 **NPE**를 방지하고 싶은 경우
    
    ```java
    return problems.stream()
            .map(problem -> ProblemResponse.create(
                    problem,
    	              // 주관식 문제는 선지(List<Option>)에 대해 빈 리스트를 반환
                    optionResponseMap.getOrDefault(problem.getId(), List.of())
            ))
            .toList();
    ```
    
<br>    

- **예외를 반환해야 하는 경우**
    
    → 반환값이 없으면 안 되는 경우
    
    ```java
    @Transactional(readOnly = true)
    public MyPageResponse getMyPage(long userId) {
        return userRepository.findMyPageByUserId(userId)
                .orElseThrow(()-> new RestApiException(CustomErrorCode.USER_PAGE_NOT_FOUND));
    }
    ```
