# 멤버 클래스는 되도록 static으로 만들라
> ### 💡 멤버 클래스 vs 중첩 클래스
> `멤버 클래스`는 클래스의 구성 요소이다.  
> `중첩 클래스`는 다른 클래스 안에 정의된 클래스를 말한다.
> 
> 모든 `중첩 클래스`는 `멤버 클래스`인가 ? **아니다.**
## 중첩 클래스 4종류
>- 정적 멤버 클래스
>- 비정적 멤버 클래스
>- 익명 클래스
>- 지역 클래스
### 📍 정적 멤버 클래스
`outer class`에 있는 `private` 멤버에 접근할 수 있고, `outer class`의 인스턴스를 필요로 하지 않는다.  
`정적 멤버 클래스`는 `outer class`와 함께 쓰일 때만 유용한 **publc 도우미 클래스**로 쓰인다.
``` java
class A {
    static class B {}
}

void foo() {
    A.B b = new B();
}
```
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
### 📍 비정적 멤버 클래스
주로 `어댑터`를 정의할 때 자주 쓰인다.  
즉, 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용하는 것이다.
``` java
class A {
    class B {}
}

void foo() {
    A.B b = new A().new B();
}
```
`어댑터`의 대표적인 예로는 `InputStreamReader`, `BufferedReader`가 있다.
``` java
// 어댑터 형식
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

// 어댑터 예제
public class AdapterEx {
    public static void main(String[] args){
        try(InputStream is = new FileInputStream("number.txt");
            InputStreamReader isr = new InputStreamReader(is);
            BufferedReader reader = new BufferedReader(isr);
            while(reader.ready()) {
                System.out.println(reader.readLine());
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
### 
`outer class`의 인스턴스를 암묵적으로 참조하고 있기 때문에 생성 시 `outer class`의 인스턴스가 필요하다.  
또한 `outer class`가 더 이상 사용되지 않아도 `inner class` 의 참조로 인해 `GC`가 수거하지 못해 메모리 누수가 발생할 여지가 있다.  
따라서 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 `static`을 붙여서 정적 멤버 클래스로 만들자.
### 📍 익명 클래스
이름이 없는 클래스로, `non-static class` 이다.   
또한 `정적 팩터리 메서드`를 구현할 때 사용된다.
``` java
static List<Integer> intArrayAsList(final int[] a){
    Objects.requireNonNull(a);
    
    return new AbstractList<Integer>() {
        public Integer get(int i) {
            return a[i];
        }
        ... 
    }
}
```
즉석에서 작은 함수 객체나 처리 객체를 만드는데 사용했지만, Java 8 버전부터는 `람다`를 사용한다.
``` java
// 익명 클래스 사용
Collections.sort(list, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});

// 람다 사용
Collections.sort(list, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```
### 📍 지역 클래스
블록 안에 정의된 클래스로 메서드 안에서만 클래스를 사용하고 싶을 때 사용한다.  
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
네 가지 중첩 클래스 중 가장 드물게 사용된다.  
지역 변수를 선언하는 곳이면 어디든 정의할 수 있고 유효 범위도 지역 변수와 같다.  
익명 클래스와 다르게 이름이 있어 반복 사용이 가능하지만, 익명 클래스처럼 `static`을 사용할 수 없고 가독성을 위해 짧게 작성해야한다.

## 결론 
중첩 클래스를 사용할 때는 각 특징에 맞게 사용하자.
#### 📍 멤버 클래스 vs 내부 클래스
> 메서드 밖에서 사용하거나 코드가 긴 경우 `멤버 클래스`
> 메서드 안에서만 사용하고 코드가 길지 않은 경우 `내부 클래스`(`익명 클래스`, `지역 클래스`)
#### 📍 정적 멤버 클래스 vs 비정적 멤버 클래스
> 바깥 인스턴스를 참조하는 경우 `비정적 멤버 클래스`  
> 바깥 인스턴스를 참조하지 않는 경우 `정적 멤버 클래스`
#### 📍 익명 클래스 vs 지역 클래스
> 인스턴스를 생성하는 시점이 단 한 곳인 경우 `익명 클래스`  
> 인스턴스를 생성하는 지점이 한 곳이 아닌 경우 `지역 클래스`
## 참조
- https://tecoble.techcourse.co.kr/post/2020-11-05-nested-class/
- https://ss-hoon.github.io/effective-java/item24/
- https://skasha.tistory.com/34