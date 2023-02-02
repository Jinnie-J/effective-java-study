
## 정확한 답이 필요하다면 float과 double은 피하라

- float과 double 타입은 과학과 공학 계산용으로 설계되었다.
- 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 세심하게 설계되었다.
- 따라서 정확한 결과가 필요할 때는 사용하면 안된다.
- float과 double 타입은 특히 금융 관련 계산과는 맞지 않는다.

예 1) 주머니에 1.03달러가 있었는데 그중 42센트를 썼다고 해보면 남은 돈은 얼마인가?   
```java
System.out.println(1.03 - 0.42);
// 출력: 0.6100000000000001
```

예 2) 주머니에 1달러가 있었는데 10센트짜리 사탕 9개를 샀다고 해보면 얼마가 남았을까?
```java
System.out.println(1.00 - 9 * 0.10);
// 출력: 0.0999999999999998
```

예3) 주머니에 1달러가 있고, 10센트, 20센트, 30센트, 1달러짜리의 사탕이 있을 때 사탕을 몇 개나 살 수 있고, 잔돈은 얼마가 남을까?
```java
//오류 발생 - 금융 계산에 부동소수 타입 사용
public static void main(String[] args){
    double funds = 1.00;
    int itemBought = 0;
    for (double price = 0.10; funds >= price; price += 0.10){
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(달러):" + funds);
}
/*출력:
사탕 3개 구입
잔돈(달러): 0.399999999999999
*/
```

### 해결 방법 - 금융 계산에는 BigDecimal, int 혹은 long을 사용해야 한다.
앞의 코드에서 double 타입을 BigDecimal로 교체 했다.

```java
//BigDecimal을 사용한 해법. 속도가 느리고 쓰기 불편하다.
public static void main(String[] args){
    final BigDecimal TEN_CENTS = new BigDecimal(".10");

    int itemBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for (BigDecimal price = TEN_CENTS;
            funds.compareTo(price) >= 0;
            price = price.add(TEN_CENTS)){
        funds = funds.subtract(price);
        itemBought++;
            }
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(달러):" + funds);
}
/*출력:
사탕 4개 구입
잔돈(달러): 0
*/
```

### BigDecimal의 단점
기본 타입보다 쓰기가 훨씬 불편하고, 훨씬 느리다.
- 단발성 계산이리면 느리다는 문제는 무시할 수 있지만, 쓰기 불편하다.

BigDecimal의 대안으로 int 혹은 long 타입을 쓸 수도 있다. 
- 그럴 경우 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야 한다.



### 위의 예에서는 모든 계산을 달러 대신 센트로 수행하면 이 문제가 해결된다.
```java
//정수 타입을 사용한 해법
public static void main(String[] args){
    int itemBought = 0;
    int funds = 100;
    for(int price = 10; funds >= price; price += 10){
        funds -= price;
        itemBought++;
    }
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(센트): " + funds);
}
```

---
### 핵심 정리
- 정확한 답이 필요한 계산에는 float나 double을 피하라.
- 소수점 추적은 시스템에 맡기고, 코딩 시의 불편함이나 성능 저하를 신경 쓰지 않겠다면 BigDecimal을 사용하라.
- BigDecimal이 제공하는 여덟 가지 반올림 모드를 이용하여 반올림을 완벽히 제어할 수 있다.
- 반면, 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라.
- 숫자를 아홉 자리 십진수로 표현할 수 있다면 int를 사용하고, 열여덟 자리 십진수로 표현할 수 있다면 long을 사용하라.
- 열여덟ㅅ 자리를 넘어가면 BigDecimal을 사용해야 한다.