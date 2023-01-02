# @Override 애너테이션을 일관되게 사용하라
자바가 기본으로 제공하는 애너테이션 중 보통의 프로그래머에게 가장 중요한 것은 `@Override`일 것 이다.  
`@Override`는 메서드 선언에만 달 수 있으며, 이는 상위 타입의 메서드를 재정의했음을 뜻한다.

## Bigram 프로그램
영어 알파벳 2개로 구성된 문자열을 표현하는 클래스인 Bigram 프로그램을 살펴보자.
``` java
public class Bigram {
    private final char first;
    private final char second;
    
    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }
    
    public boolean equal(Bigram b) {
        return b.first = first && b.second = second;
    }
    
    public int hashCode() {
        return 31 * first + second;
    }
    
    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```
main 메서드를 보면 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가한 다음, 그 집합의 크기를 출력한다.  
Set은 중복을 허용하지 않으니 26이 출력될 거 같지만, 실제로는 260이 출력된다.

문제는 Object의 equals를 오버라이딩하려다 매개변수 타입을 Object로 하지 않아 오버로딩이 되어버린 것이다.  
따라서 잘못한 부분을 컴파일 단계에서 명확히 알 수 있도록 `@Override` 애너테이션을 사용해야한다.

``` java
@Override public boolean equal(Object o) {
    if (!(o instanceof Bigram))
        return false;
    Bigram b = (Bigram) o;
    return b.first = first && b.second = second;
}    
```
**그러니 상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override` 애너테이션을 달자.**  
예외는 구체 클래스에서 상위 클래스의 추상메서드를 재정의할 때 뿐이다.  
구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아 있다면 컴파일러가 알려주기 때문에 굳이 `@Override`를 달지 않아도 된다.

`@Override`는 클래스뿐 아니라 인터페이스의 메서드를 재정의할 때도 사용할 수 있다.  
디폴드 메서드를 지원하기 시작하면서, 인터페이스 메서드를 구현한 메서드에도 `@Override`를 사용할 경우 올바른 지에 대해 재차 확신할 수 있다.  

그치만 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 `@Override`를 다는 것이 좋다.

## 결론
재정의한 모든 메서드에 `@Override` 애너테이션을 의식적으로 달면 실수했을 때 컴파일러가 바로 알려줄 것이다.  
예외는 한 가지뿐이다.  
구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우엔 달지 않아도 되나, 단다고 해서 해로울 것도 없다.