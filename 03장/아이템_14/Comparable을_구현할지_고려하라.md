# 아이템14. Comparable을 구현할지 고려하라

## CompareTo
- **Comparable 인터페이스의 유일한 메서드**
- Object의 메서드가 아니다.
- Object.equals에 더해서 순서까지 비교할 수 있으며 Generic을 지원한다.  
  (제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은컴파일 타임에 정해진다.)

## CompareTo 규약
- 자기 자신(this)이 compareTo에 전달된 객체보다 작으면 음수, 같으면 0, 크다면 양수를 리턴한다.
- 반사성, 대칭성, 추이성을 만족해야 한다.
    * 반사성 : 자기 자신과 비교
    * 대칭성 : a.compareTo(b), b.compareTo(a)
    * 추이성 : x.compareTo(y) > 0 && y.compareTo(z) > 0 이면 x.compareTo(z) > 0  
- 반드시 따라야 하는 것은 아니지만 compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.    
`x.compareTo(y) == 0이라면 x.equals(y)가 true여야 한다.`

``` java
/* BigDecimal 클래스는 compareTo와 equals가 일관되지 않는다. 이에 대해 문서화 하고 있음. */
BigDecimal n1 = new BigDecimal("1.0");
BigDecimal n2 = new BigDecimal("1.00");

n1.compareTo(n2) -> 0
n1.equals(n2) -> false
```

## CompareTo 구현 방법
- 자연적인 순서를 제공할 클래스에 implements Comparable<T> 을 선언한다.
- compareTo 메서드를 재정의한다.
- compareTo 메서드 안에서 기본 타입은 박싱된 기본 타입의 `compare`을 사용해 비교한다.  
  **compareTo 메서드에서 관계 연산자 `<` 와 `>` 를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니 추천하지 않는다.**
- 핵심 필드가 여러 개라면 가장 핵심적인 필드부터 비교해나가자. 순서를 비교하는 데 있어 가장 중요한 필드를 비교하고 그 값이 0이라면 다음 필드를 비교한다.
  ``` java
  public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode); // 가장 중요한 필드
    if (result == 0) {
      result = Short.compare(prefix, pn.prefix); // 두 번째로 중요한 필드
      if (result == 0) {
        result = Short.compare(lineNum, pn.lineNum); // 세 번째로 중요한 필드
    }
    return result;
  }
  ```
- 기존 클래스를 확장하고 필드를 추가하는 경우 compareTo 규약을 지킬 수 없다. 
  * Composition을 활용할 것. (Java 8 이전)
- Java 8부터 함수형 인터페이스, 람다, 메서드 레퍼런스와 Comparator가 제공하는 기본 메서드와 static 메서드를 사용해서 Comparator를 구현할 수 있었다.
  * 약간의 성능 저하가 뒤따른다.
  ``` java  
  /* 비교자 생성 메서드를 활용한 비교자 */
  private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

   public int compareTo(PhoneNumber pn) {
       return COMPARATOR.compare(this, pn);
   }
   ```
  * Comparator가 제공하는 메서드 사용하는 방법
    * Comparator의 static 메서드를 사용해서 Comparator 인스턴스 만들기 `(ex. comparingInt)`
    * 인스턴스를 만들었다면 default 메서드를 사용해서 메서드 호출 이어가기 (체이닝) `(ex. thenComparingInt)`
    * static 메서드와 default 메서드의 매개 변수로는 람다표현식 또는 메서드 레퍼런스`(ex. PhoneNumber::getAreaCode)`를 사용할 수 있다.
  
  
---  
> ### **핵심 정리**  
> * 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 **Comparable 인터페이스**를 구현하여 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야한다.  
> * compareTo 메서드에서 필드의 값을 비교할 때 < 와 > 연산자는 쓰지 말아야 한다.  
> * 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.  
