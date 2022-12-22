
# int 상수 대신 열거 타입을 사용하라
열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.
- 사계절, 태양계의 행성, 카드게임의 카드 종류 등이 좋은 예다.
- 자바에서 열거 타입을 지원하기 전에는 다음 코드처럼 정수 상수를 한 묶음 선언해서 사용하곤 했다.

정수 열거 패턴 - 상당히 취약하다.
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORNAGE_BLOOD = 2;
```
정수 열거 패턴(int enum pattern) 기법에는 단점이 많다.

### 정수 열거 패턴의 단점
- 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다.
- 오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자(==)로 비교하더라도 컴파일러는 아무런 경고 메시지를 출력하지 않는다.
    - 컴파일러 입장에서는 둘다 정수 0 이기 때문에 APPLE_PIE가 사용되는 메소드에 GRAPE_PIE가 전달되어도 컴파일때 아무런 문제가 없다.
- 평범한 상수를 나열한 것뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다. 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다.
- 정수 상수는 문자열로 출력하기가 다소 까다롭다. 값을 출력하거나 디버거로 살펴보면 단지 숫자로만 보여서 썩 도움이 되지 않는다.
- 정수 대신 문자열 상수를 사용하는 변형 패턴도 있다. 문자열 열거 패턴이라 하는 이 변형은 더 나쁘다.
    - 경험이 부족한 프로그래머가 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 만들기 때문이다.
    - 이렇게 하드코딩한 문자열에 오타가 있어도 컴파일러는 확인할 길이 없으니 자연스럽게 런타임 버그가 생긴다.

## 열거 타입(enum type)
열거 패턴의 단점을 말끔히 씻어주는 동시에 여러 장점을 안겨주는 대안인 열거 타입이 있다.
```java
public enum Apple{FUJI, PIPPIN, GRANNY_SMITH}
public enum ORange{NAVEL, TEMPLE, BLOOD}
```
- 자바의 열거 타입은 완전한 형태의 ***클래스***라서 (단순한 정숫값일 뿐인)다른 언어의 열거 타입보다 훨씬 강력하다.
### 열거 타입의 장점

#### 1. 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다. 
- 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.
- 싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 거꾸로 열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있다.

#### 2. 열거 타입은 컴파일타임 타입 안정성을 제공한다.
- Apple 열거 타입을 매개변수로 받는 메서드를 선언했다면, 건네받은 참조는 Apple의 세 가지 값 중 하나임이 확실하다.
- 다른 타입의 값을 넘기려 하면 컴파일 오류가 한다.

#### 3. 열거 타입에는 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.
```java
private enum Apple{
    PIE, JAM;
}
private enum Grape{
    PIE, JAM;
}
```
- 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다. 
- 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다.

#### 4. 열거 타입의 toString 메소드는 상수 이름을 문자열로 반환한다.
- APPLE.toString() => "APPLE"

#### 5. 열거 타입에는 임의의 메소드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.
- Apple과 Orange를 예로 들면, 과일의 색을 알려주거나 과일 이미지를 반환하는 메서드를 추가할 수 있다.
- 가장 단순하게는 그저 상수 모음일 뿐인 열거타입이지만, 고차원의 추상 개념 하나를 완벽하게 표현해낼 수도 있는 것이다.
- Object 메서드들을 높은 품질로 구현해놨고, Comparable과 Serializable도 구현해두었다.


#### 6. 열거 타입에 선언한 상수 하나를 제거하더라도 제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다.
- 제거된 상수를 참조하는 클라이언트 또한 제거된 상수를 참조하는 줄에서 디버깅에 유용한 메시지를 담은 컴파일 오류가 발생할 것이다.

### 상수별 메서드 구현
열거 타입을 구현하면서 열거 타입의 메소드가 상수에 따라 다르게 동작해야 하는 경우가 생길 수도 있다. 다음과 같이 switch 문으로 해결할 수도 있지만, 새로운 상수가 추가된다면 case 문도 추가해야 한다.

```java
public enum Operation {
    PLUS,MINUS,TIMES,DIVDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVDE:
                return x / y;
        }
        throw new AssertionError("알 수 없는 연산:" + this);
    }
}
```
- 동작은 하지만 예쁘지는 않다.
- 새로운 상수를 추가하면 해당 case 문도 추가해야 한다. 혹시라도 깜빡한다면, 컴파일은 되지만 새로 추가한 연산을 수행하려 할 때 "알 수 없는 연산"이라는 런타임 오류를 내며 프로그램이 종료된다.

열거 타입은 상수별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공한다.
- 열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 클래스 몸체, 즉 각 상수에서 자신에 맞게 재정의하는 방법이다.
- 이를 상수별 메서드 구현이라고 한다.

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);
```
- apply 메서드가 상수 선언 바로 옆에 붙어 있으니 새로운 상수를 추가할 때 apply도 재정의해야 한다는 사실을 깜빡하기는 어려울 것이다.
- 그뿐만 아니라 apply가 추상 메서드 이므로 재정의하지 않았다면 컴파일 오류로 알려준다.
- 위의 코드는 Operation의 toString을 재정의해 해당 연산을 뜻하는 기호를 반환하도록 했다.

toString 메서드를 재정의했다면, toString이 반환하는 문자열을 해당 열거 타입 상수로 반환해주는 fromString 메소드도 함께 제공하는 걸 고려해보자.
- 위에서 toString 메소드를 이용해 기존 상수의 이름이 아닌 각 연산자의 기호를 반환하도록 구현하였다. 반대로 fromString 케소드를 구현하여 연산자 기호를 매개변수로 전달하면 알맞은 열거 타입의 객체를 반환하도록 해보자.

```java
private static final Map<String, Operation> stringToEnum =
        Stream.of(values()).collect(
            toMap(Object::toString, e -> e));

//지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```
- ```Optional<Operation>``` 을 반환하는 이유는 주어진 문자열이 가리키는 연산이 존재하지 않을 수 있음을 클라이언트에 알리고, 그 상황을 클라이언트에서 대처하도록 한 것이다.


#### 한편, 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.
- 급여 명세서에서 쓸 요일을 표현하는 열거 타입을 예로 생각해보자.

값에 따라 분기하여 코드를 공유하는 열거 타입
```java
public enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        //기본 급여
        int basePay = minutesWorked * payRate;
		//잔업수당
        int overtimePay;
        switch (this) {
        	//주말
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            //주중
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }

        return basePay + overtimePay;
    }
}
```
- 간결하지만, 관리 관점에서는 위험한 코드다.
- 휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case 문을 잊지 말고 쌍으로 넣어줘야 한다.

상수별 메서드 구현으로 급여를 정확히 계산하는 방법은 두 가지다.
- 첫 째, 잔업수당을 계산하는 모든 코드를 모든 상수에 중복해서 넣으면 된다.
- 둘 째, 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 적절히 호출하면 된다.

두 방식 모두 코드가 장황해져 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다.

### 전략 열거 패턴
가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 하는 것이다.
- 잔업수당 계산을 private 중첩 열거 타입(다음 코드의 PayType)으로 옮기고 PayrollDay 열거 타입의 생성자에서 이중 적당한 것을 선택한다.
- 그러면 PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, swtich 문이나 상수별 메서드 구현이 필요 없게 된다.
- 이 패턴은 switch 문보다 복잡하지만 더 안전하고 유연하다.
```java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY), WEDNESDAY(PayType.WEEKDAY), 
    THURSDAY(PayType.WEEKDAY), FRIDAY(PayType.WEEKDAY), SATURDAY(PayType.WEEKEND), 
    SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked,payRate);
    }
    //전략 열거 타입
    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked,payRate);
        }
    }
}
```
PayRollDay 열거 타입은 기존의 switch를 사용한 코드보다 더 안전하고 유연해졌다.

### 그래서 열거 타입을 과연 언제 쓰라는 말인가?
- 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 열거 타입을 사용하자.
- 태양계 행성, 한 주의 요일, 체스 말처럼 본질적으로 열거 타입인 타입은 당연히 포함된다.
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다. 
- 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계되었다.

--- 
#### 핵심 정리
- 열거 타입은 확실히 정수 상수보다 뛰어나다. 더 읽기 쉽고 안전하고 강력하다. 
- 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다.
- 드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다. 이런 열거 타입에서는 switch 문 대신 상수별 메서드 구현을 사용하자. 
- 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.