# 전통적인 for 문보다는 for-each 문을 사용하라
## for 문
for 문의 단점은 `Iterator`와 `index 변수`를 사용해 코드가 깔끔하지 못하고, 오류 가능성이 존재한다는 것이다.  
또한 컬렌션이냐 배열이냐에 따라 코드 형태가 많이 달라진다.  
```java
for(Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // e로 무언가를 한다.
}

for(int i = 0; i < a.length; i++) {
    ... // a[i]로 무언가를 한다.
}
```
## for-each 문
정식 이름은 향상된 for문(enhanced **for** statement) 이다.  
`Iterator`와 `index 변수`를 사용하지 않아 코드가 깔끔하고, 오류가 날 일도 없다.  
또한 하나의 관용구로 컬렉션과 배열 모두 처리할 수 있다.
```java
for (Element e : elements) {
    ... // e로 무언가를 한다.
}
```
여기서 콜론(:)은 "안의(in)"라고 읽으면 된다.

컬렉션을 중첩해 순회해야 하는 경우 for-each 문의 이점이 더욱 커진다.  
다음 코드에서 버그를 찾아보자.

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
            NINE, TEN, JACK, QUEEN, KING }
...

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext())
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext())
        deck.add(new Card(i.next(), j.next()));
```

마지막 줄의 i.next()를 보면, 이 메서드는 숫자(Suit) 하나당 한 번씩만 불려야 하는데, 카드(Rank) 하나당 한 번씩 불리고 있다.  
그래서 숫자가 바닥나면 반복문에서 NoSuchElementException 을 던질 것이다.

```java
for (Iterator<Suit> i = suits.iterator(); i.hasNext())
    Suit suit = i.next();
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext())
        deck.add(new Card(suit, j.next()));
```
잘못된 코드를 수정했지만, 보기에 좋지는 않은 코드이다.  
이는 for-each 문을 중첩하는 것으로 간단히 해결된다.


```java
for (Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank))
```

## for-each 문을 사용할 수 없는 경우
### 파괴적인 필터링
컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 Iterator의 remove 메서드를 사용해야 한다.  
자바 9부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.
### 변형
리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 Iterator나 배열의 Index를 사용해야 한다.
### 병렬 반복
여러 컬렉션을 병렬로 순회해야 한다면 각각의 Iterator와 Index 변수를 사용해 제어해야 한다.

## 결론
전통적인 for 문과 비교했을 때 for-each 문은 명료하고, 유연하고, 버그를 예방해준다.  
성능 저하도 없다. 가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자.