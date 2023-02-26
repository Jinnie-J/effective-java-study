# 아이템 89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라

## 싱글턴 패턴과 직렬화
아이템 3에서 소개된 싱글턴 패턴 예제를 보면 바깥에서 생성자를 호출하지 못하게 막는 방식으로 인스턴스가 오직 하나만 만들어짐을 보장했다.

```Java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }

    public void leaveTheBuilding() { ... }
}
``` 
이 클래스는 `implements Serializable`을 추가하는 순간 더 이상 싱글턴이 아니게 된다.  
기본 직렬화를 쓰지 않더라도 (아이템 87), 명시적인 readObject를 제공하더라도 (아이템 88) 소용없다.  
어떤 readObject 메서드를 사용하든 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환한다.  

## 싱글턴 속성을 유지하는 방법 - readResolve() 메서드
readResolve 기능을 이용하면 readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다.  
역직렬화 후 새로 생성된 객체를 인수로 readResolve 메서드가 호출 되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환한다.
대부분의 경우 이때 새로 생성된 객체의 참조는 유지하지 않으므로 바로 가비지 컬렉션 대상이 된다.

![image](https://user-images.githubusercontent.com/9801031/221408862-3c100c80-b8ee-4950-9075-fe29081cfe48.png)

```Java
// 인스턴스 통제를 위한 readResolve
private Object readResolve() {
    // 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
}
```
이 메서드는 역직렬화한 객체는 무시하고 클래스 초기화 때 만들어진 Elvis 인스턴스를 반환한다.  
Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없으니 모든 필드를 `transient`로 선언해야 한다.

## 필드를 transient로 선언하지 않으면 발생하는 취약점
**readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다.**
그렇지 않으면 MutablePeriod 공격(아이템 88)과 비슷한 방식으로 readResolve 메서드가 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남는다.

### 공격 방법 예시
1. readResolve 메서드와 인스턴스 필드 하나를 포함한 '도둑(Stealer)' 클래스를 작성한다.
2. 이 클래스의 인스턴스 필드는 도둑이 '숨길' 직렬화된 싱글턴을 참조하는 역할을 한다.
3. 직렬화된 스트림에서 싱글턴의 비휘발성 필드를 도둑의 인스턴스 필드로 교체한다.
--> 이제 싱글턴은 도둑을 참조하고 도둑은 싱글턴을 참조하는 순환고리가 만들어졌다.
4. 싱글턴이 도둑을 포함하므로 역직렬화될 때 도둑의 readResolve 메서드가 먼저 호출된다.
5. 그 결과, 도둑의 readResolve 메서드가 수행될 때 도둑의 인스턴스 필드에는 역직렬화 도중인 싱글턴의 참조가 담겨 있게 된다.
6. 도둑의 readResolve 메서드는 이 인스턴스 필드가 참조한 값을 정적 필드로 복사하여 readResolve가 끝난 후에도 참조할 수 있도록 한다.
7. 이 메서드는 도둑이 숨긴 transient 가 아닌 필드의 원래 타입에 맞는 값을 반환한다.
(이 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려 할 때 VM이 `ClassCastException`을 던진다.)

```Java
// tranient가 아닌 참조 필드를 가지는 싱글턴
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
  private Elvis() { }

  private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }

  private Object readResolve() {
    return INSTANCE;
  }
}
```

```Java
// 싱글턴의 비휘발성 인스턴스 필드를 훔쳐러는 도둑 클래스
public class ElvisStealer implements Serializable {
  private static final long serialVersionUID = 0;
  static Elvis impersonator;
  private Elvis payload;

  private Object readResolve() {
    // resolve되기 전의 Elvis 인스턴스의 참조를 저장
    impersonator = payload;
    // favoriteSongs 필드에 맞는 타입의 객체를 반환
    return new String[] {"There is no cow level"};
  }
}
```


다음 프로그램은 수작업으로 만든 스트림을 이용해 2개의 싱글턴 인스턴스를 만들어낸다.
```Java 
// 직렬화의 약점을 이용해 싱글턴 객체를 2개 생성한다.
public class ElvisImpersonator {
  private static final byte[] serializedForm = new byte[]{
      (byte) 0xac, (byte) 0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
      // 코드 생략
  };

  private static Object deserialize(byte[] sf) {
        try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(sf)) {
      try (ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream)) {
        return objectInputStream.readObject();
      } catch (IOException | ClassNotFoundException e) {
        throw new IllegalArgumentException(e);
      }
  }

  public static void main(String[] args) {
    // ElvisStealer.impersonator 를 초기화한 다음,
    // 진짜 Elvis(즉, Elvis.INSTANCE)를 반환
    Elvis elvis = (Elvis) deserialize(serializedForm);
    Elvis impersonator = ElvisStealer.impersonator;
    elvis.printFavorites(); // [Hound Dog, Heartbreak Hotel]
    impersonator.printFavorites(); // [There is no cow level]
  }
}
```
이 프로그램을 실행하면 다음 결과를 출력한다.  
```Java
[Hound Dog, Heartbreak Hotel]
[A Fool Such as I]
```
이것으로 서로 다른 2개의 Elvis 인스턴스를 생성할 수 있음을 증명했다.

## 해결방법 - 열거 타입을 사용하자
`favoriteSongs` 필드를 `transient`로 선언하여 이 문제를 고칠 수 있지만 Elvis를 원소 하나짜리 열거 타입으로 바꾸는 편이 더 나은 선택이다. (아이템 3)  
  
직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다.
```Java
public enum Elvis {
    INSTANCE;

    private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```
위 코드는 Elvis 예를 열거 타입으로 구현한 모습이다.  
  
readResolve를 사용하는 방식이 완전히 쓸모없는 것은 아니다. 
직렬화 가능 인스턴스 통제 클래스를 작성해야 할 때, 컴파일타임에서는 어떤 인스턴스들이 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는 것이 불가능하기 때문이다.
  
## readResolve 메서드의 접근성
readResolve 메서드의 접근성은 매우 중요하다. final 클래스에서라면 readResolve 메서드는 private 이어야 한다.  

### final이 아닌 클래스에서의 주의사항
- priavet으로 선언하면 하위 클래스에서 사용할 수 없다.
- package-private으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다.
- protected나 public으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다. 
- protected나 public이면서 하위 클래스에서 재정의하지 않았다면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException을 일으킬 수 있다.

## 핵심정리
> 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 열거 타입을 사용하자.  
> 여의치 않은 상황에서 직렬화와 인스턴스 통제가 필요하다면 readResolve 메서드를 사용하고, 그 클래스에서 모든 참조 타입 인스턴스 필드를 transient로 선언해야 한다.




## 참고
https://jjingho.tistory.com/140
