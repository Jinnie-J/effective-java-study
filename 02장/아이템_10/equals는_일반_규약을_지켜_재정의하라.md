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
###


###
### 일관성 (consistency)

null이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`를 반복해서 호출하면 항상 `true`를 반환하거나 항상 `false`를 반환한다.

### null-아님
null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.
### 
## 양질의 equals 메서드 구현방법
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사한다.

## equals 메서드 구현 시 주의사항
- equals를 재정의할 땐 hashCode도 반드시 재정의하자.
- 너무 복잡하게 해결하려 들지 말자.
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
## 결론
꼭 필요한 경우가 아니면 equals를 재정의하지 말자.  
대부분의 경우에 Object의 equals가 원하는 비교를 정확히 수행해준다.  
재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.  