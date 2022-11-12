
# private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 싱글턴(Singleton)이란?
싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 

### 싱글턴패턴이란?
생성자의 호출이 반복적으로 이루어져도 실제로 생성되는 객체는 최초 생성된 객체를 반환함을 의미한다.
```java
public class Animal{
    private static Animal animals = null;

    private Animals(){}

    public static Animals getInstance(){
        if(animals == null){
            animals = new Animals();
        }
        return animals;
    }
}
```
Animals라는 클래스가 있고 private static을 사용하여 최초로 메모리를 한번만 할당한다.

### 문제점
클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.
타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴을 가짜(mock) 구현으로 대체할 수 없기 때문이다. (싱글톤은 생성 방식이 제한적이기 때문에 Mock 객체로 테스트하기가 어려우며, 동적으로 객체를 주입하기도 힘들다.)

## 싱글턴을 만드는 방식
### 1. public static final 필드 방식의 싱글턴
```java
public class Elvis{
    public static final Elvis INSTANCE = new Elvis();
    private Elvis(){...}

    public void leaveTheBuilding(){...}
}
```
- private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한번만 호출된다. 
- 예외는 단 한 가지로, 권한이 있는 클라이언트는 리플렉션 API(구체적인 클래스 타입을 알지 못해도, 그 클래스의 메소드, 타입, 변수들에 접근할 수 있도록 해주는 자바 API)인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다. 이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.
#### 장점
- 해당 클래스가 싱글턴임이 API에 명확히 드러난다.
- 간결하다.

### 2. 정적 팩터리 방식의 싱글턴
     
#### 정적 팩토리 메서드
객체 생성을 흔히 사용하는 생성자가 아닌 정적(static) 메서드로 하는 것을 정적 팩토리 메서드라고 한다.
```java
public class Elvis{
    private static final Elvis INSANCE = new Elvis();
    private Elvis(){...}
    public static Elvis getInstance(){return INSANCE;}

    public void leaveTheBuilding(){...}
}
```
- getInstance는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis 인스턴스란 결코 만들어지지 않는다.
#### 장점
- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
- 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.
- 이러한 장점들이 굳이 필요하지 않다면 public 필드 방식이 좋다.

#### 싱글턴 클래스 직렬화
싱글턴 클래스를 직렬화하려면 모든 인스턴스 필드를 일시적이라고 선언하고 readResolve 메소드를 제공해야 한다. 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다. 
```java
//싱글턴임을 보장해주는 readResolve 메소드
private Object readResolve(){
    //진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSANCE;
}
```

### 3. 원소가 하나인 열거 타입을 선언
```java
public enum Elvis{
    INSANCE;

    public void leaveTheBuilding(){...}
}
```
- 더 간결하고, 추가 노력 없이 직렬화할 수 있고, 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.
- 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.
- 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.


#### 자바 싱글톤과 스프링 싱글톤의 차이
- Spring과 같은 프레임워크를 통해 만들어지는 싱글톤은 자바를 통해 직접 구현하는 싱글톤과 다르다. 스프링 프레임워크를 통해 직접 객체(빈)들을 싱글톤으로 관리함으로써 자바 클래스에 불필요한 코드들을 제거하여 객체를 재사용함과 동시에 객체지향스러운 개발을 할 수 있도록 해주었다.
- 객체의 생성을 스프링에게 위임함으로써 싱글톤 패턴을 자바 언어 레벨에서 직접 구현하기 위한 내용들이 모두 제거되어서 위에서 설명한 모든 문제점들이 해결되었다.