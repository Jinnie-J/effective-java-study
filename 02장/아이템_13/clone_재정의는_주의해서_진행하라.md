# clone 재정의는 주의해서 진행하라

## Clonable은 메서드가 없는 인터페이스이다.
clone 메서드는 원본 객체의 필드 값과 동일한 값을 가지는 새로운 객체를 생성해준다.  
clone 메서드를 사용하기 위해서는 해당 클래스에서 Cloneable 인터페이스를 구현해주어야 한다.

클래스에서 clone을 재정의하기 위해선 해당 클래스에 Cloneable 인터페이스를 상속받아 구현하여야 한다. 그런데 정작 clone 메소드는 Cloneable 인터페이스가 아닌 Object에 선언되어있다. Cloneable 인터페이스는 아무것도 선언되어 있지 않은 빈 인터페이스이다. 

### Cloneable 인터페이스의 역할
```java
/**
 * A class implements the <code>Cloneable</code> interface to
 * indicate to the {@link java.lang.Object#clone()} method that it
 * is legal for that method to make a
 * field-for-field copy of instances of that class.
 * <p>
 * Invoking Object's clone method on an instance that does not implement the
 * <code>Cloneable</code> interface results in the exception
 * <code>CloneNotSupportedException</code> being thrown.
 * <p>
 * By convention, classes that implement this interface should override
 * {@code Object.clone} (which is protected) with a public method.
 * See {@link java.lang.Object#clone()} for details on overriding this
 * method.
 * <p>
 * Note that this interface does <i>not</i> contain the {@code clone} method.
 * Therefore, it is not possible to clone an object merely by virtue of the
 * fact that it implements this interface.  Even if the clone method is invoked
 * reflectively, there is no guarantee that it will succeed.
 *
 * @author  unascribed
 * @see     java.lang.CloneNotSupportedException
 * @see     java.lang.Object#clone()
 * @since   1.0
 */
public interface Cloneable {
}
```
- Coneable 인터페이스는 상속받은 클래스가 복제해도 되는 클래스임을 명시하는 용도의 인터페이스이다. 
- Cloneable 인터페이스의 역할은 object의 clone의 동작 방식을 결정한다. 
- Cloneable을 상속한 클래스의 clone 메소드를 호출하면 해당 클래스를 필드 단위로 복사하여 반환한다.
- 만약 Cloneable을 상속받지 않고 clone 메소드를 호출했다면 'CloneNotSupportedException'을 던진다.

### Object clone 메소드의 일반 규약
- x.clone() != x은 참이다.
    - 복사한 객체와 원본 객체는 서로 다른 객체이다.
- x.clone().getClass() == x.getClass()은 일반적으로 참이다.
    - 하지만 반드시 만족해야 하는 것은 아니다.
- x.clone.equals(x) 은 참이다.
    - 복사한 객체와 원본객체는 논리적 동치성이 같다.
- x.clone().getClass() == x.getClass()
    - super.clone()을 호출해 얻은 객체를 clone 메소드가 반환한다면, 이 식은 참이다. 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.


getClass메소드는 현재 참조하고 있는 클래스를 확인할 수 있는 메소드이다.

Object.clone 메서드는 똑같은 클래스 타입으로 새로운 객체를 만드는 것과 같다고 보면 될 것같다. 각 멤버변수를 '=' 을 이용해서 복사한다고 생각하면 쉽게 이해할 수 있을 것 같다.



## clone 메서드의 사용
### 가변 상태를 참조하지 않는 클래스
```java
class PhoneNumber implements Cloneable {
    @Override
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone(); //object타입으로 반환하지 말자.
        } catch(CloneNotSupportedException e) {
            throw new AssertionError(); 
        }
    }
}
```
- 가변 상태를 참조하지 않는 클래스라면 super.clone()을 호출하는 것만으로도 충분하다.
- super.clone()를 실행하면 PhoneNumber에 대한 복제가 이루어진다.
- Object clone 메소드의 리턴 타입은 Object 이지만, 자바가 공변 반환 타이핑을 지원하여 PhoneNumber 타입으로 캐스팅하여 리턴하는 것이 가능하다.

#### 공변 반환 타입
- JDK 1.5부터 추가된 개념이다.
- 부모 클래스의 메소드를 오버라이딩하는 경우, 부모 클래스의 반환 타입은 자식 클래스의 타입으로 변경이 가능하다.

### 가변 상태를 참조하는 클래스용 clone 메서드
```java
public class Stack implements Cloneable{
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object o) {}
    }
```
- 이 클래스의 clone 메서드가 단순히 super.clone()의 결과를 그대로 반환한다면 elemenets 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것이다.
- 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다는 이야기다. 따라서 프로그램이 이상하게 동작하거나 nullPointerExcpeion을 던질 것이다.
- clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야한다.
- 그래서 Stack의 clone 메서드는 제대로 동작하려면 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출해주는 것이다.
```java
@Override
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch(CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

#### 배열 복사
배열을 복제하는 방법 중 가장 권장하는 방법은 array.clone()을 이용해 복사하는 방법이다.   
하지만, array 필드가 final이 적용되어 있다면 array.clone()을 통한 초기화를 할 수 없다. (final인 경우 새로운 값을 할당하지 못하기 때문에)   
따라서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.


### 복잡한 가변 상태를 갖는 클래스의 복제(HashTable)
```java
public class HashTable implements Clonable {
    private Entry[] bucket = ...;
    static class Entry{
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
      //나머지 코드는 생략 
    }
}
```
```java
@Override
public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.bucket = result.bucket.clone();
      return result;
    } catch (CloneNotSupportedException e) {
            throw new AssertionError();
    }
}
```
- 복제본은 자신만의 버킷 배열을 갖지만, 배열내의 Entry는 원본과 같은 연결리스트를 참조하여, 불변성이 깨지게 된다.
- 따라서 버킷 안에 있는 Entry의 요소들을 순회하여 각 버킷에 대해 깊은 복사를 수행해야 한다.

```java
Entry deepCopy() { //엔트리가 가르키는 연결리스트를 재귀적으로 복사.
    return new Entry(key, value, next == null ? null : next.deepCopy());
}
```
```java
@Override
public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];
        for (int i = 0; i < buckets.length; i++) {
            if (buckets[i] != null)
                result.buckets[i] = buckets[i].deepCopy();
        }
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```
- private 클래스인 HashTable.Entry는 깊은 복사(deep copy)를 지원하도록 보강되었다.
- HashTable의 clone 메서드는 먼저 적절한 크기의 새로운 버킷 배열을 할당한 다음 원래의 버킷 배열을 순회하며 비지 않은 각 버킷에 대해 깊은복사를 수행한다.
- 그런데 이때 Entry의 deepCopy 메서드는 자신이 가리키는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출하여 리스트의 원소 수만큼 스택 프레임을 소비하게 된다. -> 리스트가 길면 스택 오버플로우를 일으킬 위험이 있다.
- 이 문제를 피하려면 deepCopy를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정해야 한다.

```java
Entry deepCopy(){
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p=p.next)
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

## clone() 재정의 보다는 복사 생성자와 복사 팩터리를 사용하자
Cloneable을 이미 구현한 클래스를 확장하지 않는다면 복사 생성자와 복사 팩토리를 사용하여 더 나은 객체 복사방식을 제공하자.

### 복사 생성자
단순히 자신과같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.
```java
public Yum(Yum yum){...}; 
```
### 복사 팩터리
복사 팩터리는 복사 생성자를 모방한 정적 팩터리다.
```java
public static Yum newInstance(Yum yum){...};
```
예시)
```java
//복사 생성자
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(Stack s) {
        this.elements = s.elements.clone();
        this.size = s.size;
    }
}
//복사 팩터리
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public static Stack newInstance(Stack s) {
        return new Stack(s.elements, s.size);
    }
}
```

- 복사 생성자와 복사 팩터리 메서드는 Cloneable/clone 방식보다 나은 면이 많다
    - 언어 모순적이고 위험한 객체 생성 메커니즘을 사용하지 않는다. (super.clone())
    - clone 규약에 기대지 않는다.
    - 정상적인 final필드 용법과도 충돌하지 않는다.
    - 불필요한 check exception 처리가 필요없다.
    - 형변환이 필요없다.
    - 복사 생성자와 복사 팩터리는 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.


### 핵심 정리
- Cloneable이 물고 온 모든 문제를 되짚어 봤을 때, 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다.
- final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야 한다.
- 기본 원칙은 '복제 기능은 생성자와 팩터리를 이용하는 게 최고'라는 것이다.
- 단, 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.