# 멤버 클래스는 되도록 static으로 만들라
> ### 💡 멤버 클래스 vs 중첩 클래스
> `멤버 클래스`는 클래스의 구성 요소이다.  
> `중첩 클래스`는 다른 클래스 안에 정의된 클래스를 말한다.
> 
> 모든 `중첩 클래스`는 `멤버 클래스`인가 ? **아니다.**
## 중첩 클래스 4종류
### 정적 멤버 클래스
``` java
class A {
    static class B {}
}

void foo() {
    A.B b = new B();
}
```
`outer class`에 있는 `private` 멤버에 접근할 수 있고, `outer class`의 인스턴스를 필요로 하지 않는다.  
`정적 멤버 클래스`는 `outer class`와 함께 쓰일 때만 유용한 **publc 도우미 클래스**로 쓰인다.  
###
계산기 클래스를 예시로 들어보자.
``` java
public class Calculator {
    public enum Operation { // 열거 타입도 암시적 static 이다.
        PLUS, MINUS, SUBTRACT
    }
}
```
`Calculator` 클래스는 해당 클래스의 클라이언트에서 `Calculator.Operation.PLUS`나 `Calculator.Operation.MINUS` 같은 형태로 원하는 연산을 참조할 수 있다.  
### 비정적 멤버 클래스
``` java
class A {
    class B {}
}

void foo() {
    A.B b = new A().new B();
}
```
주로 `어댑터`를 정의할 때 자주 쓰인다.  
즉, 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용하는 것이다.
``` java
public class MySet<E> extends AbstractSet<E> {
    ... // 생략
    
    @Override 
    public Iterator<E> iterator() {
    	return new MyIterator();
    }
    
    // 비정적 멤버 클래스
    private class MyIterator implements Iterator<E> {
        ...
    }
}
```
### 
`outer class`의 인스턴스를 암묵적으로 참조하고 있기 때문에 생성 시 `outer class`의 인스턴스가 필요하다.  
또한 `outer class`가 더 이상 사용되지 않아도 `inner class` 의 참조로 인해 `GC`가 수거하지 못해 메모리 누수가 발생할 여지가 있다.  
따라서 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 `static`을 붙여서 정적 멤버 클래스로 만들자.
### 익명 클래스
이름이 없는 클래스로, Non-static class 이다.
### 지역 클래스
네 가지 중첩 클래스 중 가장 드물게 사용된다.
``` java
class A {
    void doSomething() {
        class B {}
    }
}
void foo() {
    A a = new A();
    a.doSomething();
}
```


## 참조
- https://tecoble.techcourse.co.kr/post/2020-11-05-nested-class/