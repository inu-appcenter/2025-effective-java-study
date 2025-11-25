# 아이템 60

# 아이템 60 - 정확한 계산에는 float, double을 피하라

## 피해야 하는 이유

float, double은 정확한 십진 계산을 위한 타입이 아님

이진 부동 소수점 방식으로 숫자 저장함

## 이진 부동 수수점 방식

컴퓨터는 모든 숫자를 2진수로 바꿔서 저장해야하는데

0.1같은 경우는 0.1(10진수) = 0.00011001100110011001100110011... (2진수)
이렇게 무한소수가 되므로 정확한 표현이 불가능해서 컴퓨터는 중간에서 끊고 근사값을 저장함

- 만약 1.03 - 0.42를 하는경우는
    
    1.03 → 1.02999997139…
    
    0.42 → 0.41999998688…
    
    따라서 둘을 빼면 0.6100000000000001 → 이런 이상한 값이 나옴
    

이 문제는 금융/회계 계산에서는 절대 허용x

```jsx
double funds = 1.00;
int itemsBought = 0;
for (double price = 0.10; funds >= price; price += 0.10) {
    funds -= price;
    itemsBought++;
}
System.out.println(itemsBought);
System.out.println(funds);

------

결과 
3개 구입
잔돈 : 0.3999999999999999
```

## 해결 방법

## BigDecimal 사용

반드시 **문자열 생성자를 사용해야 한다**

new BigDecimal("0.10");

숫자(double)를 그대로 넣으면 이미 오염된 이진 부동 소수점 값이 들어온다.

```jsx
final BigDecimal TEN_CENTS = new BigDecimal("0.10");
BigDecimal funds = new BigDecimal("1.00");
int itemsBought = 0;

for (BigDecimal price = TEN_CENTS;
     funds.compareTo(price) >= 0;
     price = price.add(TEN_CENTS)) {

    funds = funds.subtract(price);
    itemsBought++;
}

-----

4개 구입
잔돈 : 0
```

### **장점**

정확도 완벽

### **단점**

느림

### int 또는 long 사용

정수를 그대로 저

달러 대신 **센트**로 계산한다.

- $1.00 → 100센트
- $0.10 → 10센트

```jsx
int funds = 100;
int itemsBought = 0;

for (int price = 10; funds >= price; price += 10) {
    funds -= price;
    itemsBought++;
}
```

### 장점

- 빠름 (BigDecimal보다 훨씬)
- 코드가 간결함

### 단점

- 소수점 자리를 직접 관리해야 함
- 표현 가능한 범위가 제한됨 (int → 9자리, long → 18자리)

## 결론

- **float, double은 근사값 계산용 → 정확한 계산에는 절대 사용 금지**
- **금융 계산 → BigDecimal or 정수(int/long) 사용**
- 숫자를 정확히 표현해야 하면 **BigDecimal("0.10") 같은 문자열 생성자 사용**
- 표현 범위:
    - int: 최대 9자리
    - long: 최대 18자리
    - 그 이상 → BigDecimal 필요