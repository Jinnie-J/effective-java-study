# readObject 메서드는 방어적으로 작성하라

아이템 50에서 불변인 날짜 범위 클래스를 만드는 데 가변인 Date 필드를 이용했다. 그래서 불변식을 지키고 불변을 유지하기 위해 생성자와 접근자에 Date 객체를 방어적으로 복사하느라 코드가 길어졌다.

방어적 복사를 사용하는 불변 클래스
```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime()); // 가변인 Date 클래스의 위험을 막기 위해 새로운 객체로 방어적 복사
        this.end = new Date(end.getTime());

        if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + " 가 " + end + " 보다 늦다.");
        }
    }

    public Date start() { return new Date(start.getTime()); }
    public Date end() { return new Date(end.getTime()); }
    public String toString() { return start + " - " + end; }
    
    // ... 나머지 코드는 생략
}
```
이 클래스를 직렬화하기로 해보자. Period 객체의 물리적 표현이 논리적 표현과 부합하므로 기본 직렬화 형태를 사용해도 나쁘지 않다. 그러니 이 클래스 선언에 implements Serializable을 추가하는 것으로 해결이 될 것 같다.  
하지만 이렇게 하면 이 클래스의 주요한 불변식을 더는 보장하지 못하게된다.

문제는 readObject 메서드가 실질적으로 또다른 public 생성자이기 때문이다. 따라서 다른 생성자와 똑같은 수준으로 주의를 기울여야 한다.

- 인수가 유효한지 검사해야 한다. (아이템 49)
- 필요하다면 매개변수를 방어적으로 복사해야 한다. (아이템 50)

readObject가 이 두 작업을 제대로 수행하지 못하면 공격자는 아주 손쉽게 해당 클래스의 불변식을 깨뜨릴 수 있다.

## readObject
readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다. 
- 보통의 경우 스트림은 정상적으로 생성된 인스턴스를 직렬화해 만들어진다.
- 하지만 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면 문제가 생긴다. 
- 정상적인 생성자로는 만들어낼 수 없는 객체를 생성해낼 수 있기 때문이다.

단순히 Period 클래스 선언에 implements Serializable만 추가했다고 해보자. 그러면 다음의 괴이한 프로그램을 수행해 종료 시각이 시작 시각보다 앞서는 Period 인스턴스를 만들 수 있다.

```java
public class BogusPeriod {
    // 진짜 Period 인스턴스에서는 만들어질 수 없는 바이트 스트림
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
        0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
        // ... 생략
    }

    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }

    //주어진 직렬화 형태(바이트 스트림)로부터 객체를 만들어 반환한다.
    static Object deserialize(byte[] sf) {
        try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(sf)) {
            try (ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream)) {
                return objectInputStream.readObject();
            }
        } catch (IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```

이 프로그램을 실행하면 다음과 같이 출력된다.
```
Fri Jan 01 12:00:00: PST 1999 - Sun Jan 01 12:00:00 PST 1984
```
Period를 직렬화할 수 있도록 선언한 것만으로 클래스의 불변식을 깨뜨리는 객체를 만들 수 있게 된 것이다.

### readObject 메서드의 유효성 검사
위의 문제를 해결하려면 Period의 readObject 메서드가 ddefaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다. 이 유효성 검사에 실패하면 InvalidOBjectException을 던지게 하여 잘못된 역직렬화가 일어나는 것을 막을 수 있다.

```java
//유효성 검사를 수행하는 readObject 메서드 - 아직 부족하다.
private void readObject(ObjectInputStream s)
        throws IOException, ClassNotFoundException {
    // 불변식을 만족하는지 검사한다.
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + " 가 " + end + " 보다 늦다.");
    }
}
```
공격자가 허용되지 않는 Period 인스턴스를 생성하는 일을 막을 수 있지만, 아직도 문제가 하나 숨어 있다.

정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어 낼 수 있다. 
- 공격자는 ObjectInputStream에서 Period 인스턴스를 읽은 후 스트림 끝에 추가된 이 '악의적인 객체 참조'를 읽어 Period 객체의 내부 정보를 얻을 수 있다. 
- 이제 이 참조로 얻은 Date 인스턴스들을 수정할 수 있으니, Period 인스턴스는 더는 불변이 아니게 되는 것이다.

### 가변 공격의 예
```java
public class MutablePeriod {
    // Period 인스턴스
    public final Period period;

    // 시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date start;

    // 종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date end;

    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bos);

            // 유효한 Period 인스턴스를 직렬화한다.
            out.writeObject(new Period(new Date(), new Date()));

            /*
             * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
             */
            byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 참조 #5
            bos.write(ref); // 시작(start) 필드
            ref[4] = 4; // 참조 #4
            bos.write(ref); // 종료(end) 필드

            // Period 역직렬화 후 Date 참조를 '훔친다.'
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }

    public static void main(String[] args) {
        MutablePeriod mp = new MutablePeriod();
        Period p = mp.period;
        Date pEnd = mp.end;

        // 시간을 되돌리자!
        pEnd.setYear(78);
        System.out.println(p);

        // 60년대로 돌아가자!
        pEnd.setYear(69);
        System.out.println(p);
    }
}
```

실행 결과
```
Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978
Wed Nov 22 00:21:29 PST 2017 - Sat Nov 22 00:21:29 PST 1969
```

이처럼 Period 인스턴스는 불변식을 유지한 채 생성됐지만, 의도적으로 내부의 값을 수정할 수 있었다.    
이 문제의 근원은 Period의 readObject 메서드가 방어적 복사를 충분히 하지 않은 데 있다.

### 방어적 복사와 유효성 검사를 수행하는 readObject 메서드
객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다. 

다음의 readObject 메서드라면 Period의 불변식과 불변 성질을 지켜낼 수 있다.
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 불변식을 만족하는지 검사한다.
    if (start.compareto(end) > 0) {
        throw new InvalidObjectException(start + " 가 " + end + " 보다 늦다.");
    }
}
```
방어적 복사를 유효성 검사보다 앞서 수행하였다. 

final 필드는 방어적 복사가 불가능하니 주의하자. 
- 그래서 이 readObject 메서드를 사용하려면 start와 end 필드에서 final 한정자를 제거해야 한다. 
- 아쉬운 일이지만 앞서 살펴본 공격 위험에 노출되는 것보다는 낫다.

실행 결과
```
Wed Nov 22 00:23:41 PST 2017 - Wed Nov 22 00:23:41 PST 2017
Wed Nov 22 00:23:41 PST 2017 - Wed Nov 22 00:23:41 PST 2017
```

### readObject 메서드를 사용해도 되는지에 대한 판단
transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?
- Yes -> 기본 readObject 메서드 사용
- No -> 커스텀 readObject 메서드를 만들어 유효성 검사와 방어적 복사를 수행


---
### 핵심 정리
- readObject 메서드를 작성할 때는 언제나 public 생성자를 작성하는 자세로 임해야 한다.
- private 이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사한다. (불변 클래스 내의 가변 요소가 여기 속한다.)
- 모든 불변식을 검사하여 어긋나는 게 발견되면 InvalidObjectException을 던진다. 방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다.
- 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라
- 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자.