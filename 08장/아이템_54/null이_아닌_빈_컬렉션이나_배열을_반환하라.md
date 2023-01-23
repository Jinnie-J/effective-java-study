# 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

```Java
/* 컬렉션이 비었으면 null을 반환한다 - 따라하지 말 것! */
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 * 	단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmtpy() ? null
		: new ArrayList<>(cheesesInStock);
}
```
이 코드처럼 null을 반환한다면, 클라이언트는 이 null 상황을 처리하는 코드를 추가로 작성해야 한다.  

```Java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTION))
  System.out.println("좋았어, 바로 그거야.");
  ...
}
```
컬렉션이나 배열 같은 컨테이너(container)가 비었을 때 null을 반환하는 메서드를 사용할 때면 항시 이와 같은 **방어 코드를 넣어줘야 한다.**
클라이언트에서 방어 코드를 빼먹으면 오류가 발생할 수 있다.

## 빈 컨테이너보다는 null을 반환하는 것이 낫다?
때로는 빈 컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 쪽이 낫다는 주장도 있다.  
  
하지만 이는 두 가지 면에서 틀린 주장이다.
1. 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한 이정도의 성능 차이는 신경 쓸 수준이 못 된다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.  

```Java
/* 빈 컬렉션을 반환하는 올바른 예 */
public List<Cheese> getCheese() {
  return new ArrayList<>(cheesesInStock);
}
```
가능성은 작지만, 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수도 있다. 

## 해법 : 매번 똑같은 빈 '불변' 컬렉션을 반환하는 것
- 불변 객체는 자유롭게 공유해도 안전하다. (아이템 17)
- 예) Collections.emptyList, Collections.emptySet, Collections.emptyMap
- 단, 이 역시 최적화에 해당하니 꼭 필요할 때만 사용하자.

```Java
/* 최적화 - 빈 컬렉션을 매번 새로 할당하지 않도록 했다. */
public List<Cheese> getCheeses() {
  return cheeseInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);          
}
```

배열을 쓸 때도 절대 null을 반환하지 말고 길이가 0인 배열을 반환하라.  
```Java
/* 길이가 0일 수도 있는 배열을 반환하는 올바른 방법 */
public Cheese[] getCheeses() {
  return cheesesInStock.toArray(new Cheese[0]);
}
```
이 방식이 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열을 미리 선언해두고 매번 그 배열을 반환하면 된다. **길이 0인 배열은 모두 불변이기 때문이다.**

```Java
/* 최적화 - 빈 배열을 매번 새로 할당하지 않도록 했다. */
private static final Cheese[] EMTPY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {	
	return cheesesInStock.toArray(EMTPY_CHEESE_ARRAY);
}
```
단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는 건 추천하지 않는다. 오히려 성능이 떨어진다는 연구 결과도 있다.  

> ### 핵심 정리
> **null이 아닌, 빈 배열이나 컬렉션을 반환하라.** null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 그렇다고 성능이 좋은 것도 아니다.
