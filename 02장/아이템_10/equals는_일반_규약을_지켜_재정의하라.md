# equals는 일반 규약을 지켜 재정의하라

`equals` 메서드를 재정의하는 것은 매우 까다로운 일이다.
해당 메서드를 재정의하지 않아야 하는 경우와 재정의해야 하는 경우를 알아보자.

## equals 메서드를 재정의하지 않아야 하는 경우

- **각 인스턴스가 본질적으로 고유한 경우**
  값을 표현하는 경우가 아니라 동작하는 개체를 표현하는 클래스에 해당한다.
  ex) `Thread`
- **인스턴스의 '논리적 동치성'을 검사할 일이 없는 경우**
  논리적 동치성을 검사할 필요가 없다면, `Object`의 기본 `equals` 만으로 충분하다.
- **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우**
  ex) `AbstractSet`을 상속하는 `Set` 구현체, `AbstractList`를 상속하는 `List` 구현체, `AbstractMap`을 상속하는 `Map` 구현체
- **클래스가 `private` 이거나 `package-private` 이고 `equals` 메서드를 호출할 일이 없는 경우**

## equals 메서드를 재정의해야 하는 경우

두 객체가 물리적으로 같은지를 나타내는 `객체 식별성`이 아닌 `논리적 동치성`을 확인해야하는 경우이다.
상위 클래스의 `equals` 메서드가 논리적 동치성을 비교하도록 재정의되지 않았을 때 `equals`를 재정의해야한다.

예를 들어 `Integer`나 `String` 과 같은 값 클래스들이 위 경우에 해당한다.
값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 `인스턴스 통제 클래스`라면 `equals`를 재정의하지 않아도 된다. `Enum` 이 여기에 해당된다.

## equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 한다.

`Object` 명세에 적힌 규약을 살펴보자

> equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.
>
> - 반사성 (reflexivity)
> - 대칭성 (symmetry)
> - 추이성 (transitivity)
> - 일관성 (consistency)
> - null-아님

### 반사성 (reflexivity)
객체는 자기 자신과 같아야 한다는 뜻으로, null이 아닌 모든 참조 값 x에 대해 `x.equals(x)`는 `true`다.

### 대칭성 (symmetry)
두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻으로, null이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`가 `true`면 `y.equals(x)`가 `true`다.
###
#### 대칭성을 위배하는 케이스
```java
public final class CaseInsensitiveString {
    private final String s;
    
    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }
    
    @Override 
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String) {  // 한 방향으로만 작동한다 !
            return s.equalsIgnoreCase((String) o);
        }
    } 
}
```
`CaseInsensitiveString`의 `equals`는 순진하게 일반 문자열과도 비교를 시도한다.  
다음과 같이 `CaseInsensitiveString`과 일반 `String` 객체를 비교한다고 가정해보자.
###
```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish"

cis.equals(s) // true
s.equals(cis) // false
```
문제는 `CaseInsensitiveString`는 `String`을 알고있지만, `String`의 `equals`는 `CaseInsensitiveString`의 존재를 모른다는 데 있다.  
따라서 이 경우 대칭성을 명백히 위반하기 때문에 `CaseInsensitiveString`를 `String`과 연동하겠다는 꿈을 버리고, 아래와 같이 수정해야 한다.  
###

```java
@Override 
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
        ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```
###
### 추이성 (transitivity)
null이 아닌 모든 참조 값 x, y, z에 대해 `x.equals(y)`가 `true`이고 `y.equals(z)`도 `true`면 `x.equals(z)`도 `true`다.
  
이 요건은 간단해 보이지만 자칫하면 어기기 쉽다.  
상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 상황을 생각해보자.  
간단히 2차원에서의 점을 표현하는 클래스를 예로 들어보자.
###

```java
public class Point {
    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }


    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
}
```
위의 클래스를 확장해서 점에 색상을 더해보자.

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
}
```
`equals` 메서드를 여러 방식으로 구현해보자.
### 
#### 대칭성 위배 케이스
```java
@Override 
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color = color;
}
```
위의 경우 `Point`의 `equals`는 색상을 무시하고, `ColorPoint`의 `equals`는 입력 매개변수의 클래스 종류가 다르다며 매번 `false`를 반환할 것이다.
###
#### 추이성 위배 케이스
```java
@Override 
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;
    
    // o가 일반 Point면 색상을 무시하고 비교한다.
    if (!(o instanceof ColorPoint)) 
        return o.equals(this);
    
    // o가 ColorPoint면 색상까지 비교한다.
    return super.equals(o) && ((ColorPoint) o).color = color;
}
```
위의 경우 `대칭성`은 지켜주지만, `추이성`을 깨버린다.  
동일한 x, y 좌표를 가지고 색상은 다른 `ColorPoint`와 `Point`를 비교할 때, `ColorPoint`와 `Point`를 비교할 시에는 `true` 이지만, 색상만 다른 `ColorPoint` 객체를 비교할 시 `false`이다.  
####
<b>구체 클래스를 확장해 새로운 값을 추가하면서 `equals` 규약을 만족시킬 방법은 존재하지 않는다.</b>
그렇다면 `equals` 안의 `instanceof` 검사를 `getClass` 검사로 바꾸면 규약도 지키고 값도 추가하면서 구체 클래스를 상속할 수 있을까?

#### 리스코프 치환 원칙 위배
```java
@Override 
public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass())
        return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

결론은 아니다.  
이 경우 같은 구현 클래스의 객체와 비교할 때만 `true`를 반환할 수 있기 때문에 실제로 활용할 수는 없다.  
####
예를 들어 주어진 점이 (반지름이 1인) 단위 원 안에 있는지를 판별하는 메서드가 필요하다고 해보자.

```java
private static final Set<Point> unitCircle = Set.of(
        new Point( 1, 0), new Point( 0, 1),
        new Point(-1, 0), new Point( 0, -1));

public static boolean onUnitCircle(Point p) {
    return unitCircle.contains(p);
}

public class CounterPoint extends Point {
    // AtomicInteger : 원자성을 보장하는 Integer, 멀티쓰레드 환경에서 동시성을 보장한다
    private static final AtomicInteger counter = new AtomicInteger();
    
    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }
    public static int numberCreated() { return counter.get(); }
}
```
만약 `CounterPoint`의 인스턴스를 `onUnitCircle` 메서드에 넘기면 어떻게 될까?  
`Point` 클래스의 `equals`를 `getClass`를 사용해 작성했다면 x, y 값이 같아도 `onUnitCircle` 메서드는 `false`를 반환한다.  
반면, `Point` 클래스의 `equals`를 `instanceof` 기반으로 올바르게 구현했다면 제대로 동작할 것이다.
####
`리스코프 치환 원칙`에 따르면, 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.  
**따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.**  
하지만 `equals`를 `getClass`를 사용해 작성한다면 이를 위반하게 된다.

#### 해결 방법 1 : 상속 대신 컴포지션을 사용하라
**컴포지션 이란?**  
기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 `private 필드`로 기존 클래스의 인스턴스를 참조하는 방법을 통해 기능을 확장시키는 것이다.

```java
public class ColorPoint {
  private final Point point;
  private final Color color;
  
  public ColorPoint(int x, int y, Color color) {
      point = new Point(x, y);
      this.color = Objects.requireNonNull(color);
  }
  
  // ColorPoint의 Point 뷰를 반환한다.
  public Point asPoint() { return point; }
  
  @Override
  public boolean equals(Object o) {
     if(!(o instanceof ColorPoint)) 
         return false;
     ColorPoint cp = (ColorPoint) o;
     return cp.point.equals(point) && cp.color.equals(color);
  }
}
```
컴포지션을 사용한 예로는 `java.util.Date` 를 확장한 `java.sql.Timestamp` 가 있다.

#### 해결 방법 2 : 추상클래스의 하위클래스 사용하기
추상 클래스의 하위 클래스에서는 equals 규약을 지키면서도 값을 추가할 수 있다.  
상위 클래스의 인스턴스를 직접 만드는 게 불가능하기 때문에, 하위 클래스끼리의 비교가 가능하다. 

### 일관성 (consistency)
null이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`를 반복해서 호출하면 항상 `true`를 반환하거나 항상 `false`를 반환한다.
####
가변 객체는 비교 시점에 따라 서로 다를 수도 있는 반면, 불변 객체는 한번 다르면 끝까지 달라야하기 때문에  
불변 객체를 만들 때는 심사숙고하자.  
만약 불변 객체를 만들기로 했다면 한번 같다고 한 객체와는 영원히 같다고 답하고, 다르다고 한 객체와는 영원히 다르다고 답하도록 만들어야한다.
####
또한 `equals` 를 구현할 때는 `IP`와 같은 신뢰할 수 없는 자원이 아닌, 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.

### null-아님
`null`이 아닌 모든 참조 값 x에 대해, `x.equals(null)`은 `false`다.
####
수많은 클래스가 다음 코드처럼 입력이 `null`인지를 확인해 자신을 보호한다.
#### 명시적 null 검사
```java
// 
@Override
public boolean equals(Object o){
    if (o == null)
        return false;
    ...
}
```
#### 묵시적 null 검사
```java
@Override
public boolean equals(Object o){
        if (!(o instanceof MyType))
            return false;
        MyType mt = (MyType) o;
```
`instanceof`를 사용하는 경우 첫번째 피연산자가 `null`이면 `false`를 반환하기 때문에, 명시적으로 `null` 검사를 하지 않아도 된다.

## 양질의 equals 메서드 구현방법
1. **`==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.**  
   자기 자신이면 `true`를 반환한다. 이는 단순 성능 최적화용으로, 비교 작업이 복잡한 상황일 때 가치가 있다.
2. **`instanceof` 연산자로 입력이 올바른 타입인지 확인한다.**  
   가끔 해당 클래스가 구현한 특정 인터페이스를 비교할 수 있다.  
   이런 인터페이스를 구현한 클래스라면 `equals`에서 클래스가 아닌 해당 인터페이스를 사용해야한다.  
   ex) `Set`, `List`, `Map`, `Map.Entry` 등의 컬렉션 인터페이스
3. **입력을 올바른 타입으로 형변환한다.**  
   2번에서 `instanceof` 연산자로 입력이 올바른 타입인지 검사 했기 때문에 이 단계는 100% 성공한다.
4. **입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사한다.**  
   모든 필드가 일치해야 `true`를 반환한다.

## equals 메서드 구현 시 주의사항
- **`float`, `double`을 제외한 기본 타입 필드**  
  `==` 사용
- **참조 타입 필드**  
  각각의 `equals` 메서드 사용
- **`float`, `double` 필드**  
  각각 정적 메서드인 `compare` 메서드 사용
  `equals` 메서드는 오토박싱을 수반하여 성능상 좋지 않다.
- **배열 필드**  
  원소를 위의 각각 지침대로 비교하되 모든 원소가 핵심 필드라면 `Arrays.equals` 메서드들 중 하나를 사용하자.
- **`NPE` 예방**  
  가끔 `null`도 정상 값으로 취급하는 참조 타입 필드도 있다.  
  이런 필드는 정적 메서드인 `Objects.equals(Object, Object)`로 비교해 `NPE` 발생을 예방하자.
- **비교하기 복잡한 필드를 가진 클래스**  
  필드의 표준형(canonical form)을 저장한 후 표준형끼리 비교
- **필드의 비교 순서는 equals 성능을 좌우한다.**
  최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자.  
  파생 필드가 객체 전체의 상태를 대표하는 상황이라면 파생 필드부터 비교하자.
- **`equals`를 재정의할 땐 `hashCode`도 반드시 재정의하자.**
- **너무 복잡하게 해결하려 들지 말자.**
- **Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.**

## AutoValue 프레임워크
구글이 만든 오픈소스로, 클래스에 애너테이션을 추가하면 자동으로 `equals` 메서드들을 알아서 작성해준다.

## 결론
꼭 필요한 경우가 아니면 equals를 재정의하지 말자.  
대부분의 경우에 Object의 equals가 원하는 비교를 정확히 수행해준다.  
재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.  


## 참고
- https://velog.io/@lychee/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EC%95%84%EC%9D%B4%ED%85%9C-10.-equals%EB%8A%94-%EC%9D%BC%EB%B0%98-%EA%B7%9C%EC%95%BD%EC%9D%84-%EC%A7%80%EC%BC%9C-%EC%9E%AC%EC%A0%95%EC%9D%98-%ED%95%98%EB%9D%BC