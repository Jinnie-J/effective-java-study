# toString을 항상 재정의하라

## Object 클래스의 기본 toString 메서드
### Object 클래스
- 자바의 Object 클래스는 모든 자바 클래스의 최상위 클래스로, 전체 이름은 java.lang.Object 이다.
- Object클래스는 가장 최상위 클래스이므로 자바의 모든 클래스들은 Object 클래스를 상속받는데, 컴파일 과정에서 컴파일러가 자동으로 extends 해준다.

### toString 메서드
- toString은 말 그대로 객체의 정보를 String(문자열)형으로 형변환 해준다.
- Object 클래스를 상속받은 클래스들은 toString()을 오버라이딩(재정의)하여 사용할 수 있다.
- 자주 쓰이는 String이나 Integer 클래스에는 toString()이 이미 재정의 되어 있다.

예) Book 클래스
```java
public class Book{
    int bookNumber;
    String bookTitle;

    Book(int bookNumber, String bookTitle){
        this.bookNumber = bookNumber;
        this.bookTitle = bookTitle;
    }
}
```
```java
public class toStringEx{
    public static void main(String[] args){
        Book book = new Book(100,"개미");

        System.out.println(book);
        System.out.println(book.toString());
    }
}
/*출력 결과
object.Book@16f65612
object.Book@16f65612
```
- book을 출력한 결과와 book.toString()의 출력결과가 같다.
- System.out.println에 참조 변수(Object형)을 넣으면 toString()이 자동으로 호출된다.
- 출력 결과의 @를 기준으로 좌측은 클래스의 이름을 나타내고, 우측은 해시코드 값을 나타낸다. 해시코드란 해시 함수에 의해 자동으로 생성된 값인데 객체를 유일하게 식별할 수 있는 정수 값이다.

### toString() 재정의
Book 클래스에서 toString메서드를 사용했을 때 사용자가 원하는 문자열을 출력하도록 메서드를 재정의 해보자
```java
public class Book{
    int bookNumber;
    String bookTitle;

    Book(int bookNumber, String bookTitle){
        this.bookNumber = bookNumber;
        this.bookTitle = bookTitle;
    }
    @Override
    public String toString() {
        System.out.println(bookTitle+" "+bookNumber);
    }    
}
```
```java
public class toStringEx {
  public static void main (String[] args) {
    Book book = new Book(100,"개미");

    System.out.println(book);
    System.out.println(book.toString());
    }
}

/* 출력결과
개미 100
개미 100
*/
```

### Object의 기본 toString 메서드
- Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다.
- 이 메서드는 PhoneNumber@abdbd처럼 단순히 클래스_이름@16진수로_표현한_해시코드를 반환할 뿐이다.

### toString의 규약
- toString의 일반 규약에 따르면 '간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야 한다.
- 또한 toString의 규약은 "모든 하위 클래스에서 이 메서드를 재정의하라"고 한다.

### toString 재정의가 필요한 이유
- toString을 잘 구현한 클래스는 사용하기 편하고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.
    - map 객체를 출력하는 경우 {Jenny=PhoneNumber@addbb}보다는 {Jenny=707-867-5308}이라는 메시지가 가독성이 좋다.
- toString을 직접 호출하지 않더라도 다른 어디가에서 쓰일 것이다.
    - 객체를 println, printf, 문자열 연결 연산자(+), assert 구문에 넘길 때, 혹은 디버거가 객체를 출력할 때 자동으로 불린다.
- 작성한 객체를 참조하는 컴포넌트가 오류 메시지를 로깅할 때 자동으로 호출할 수 있다. toString을 제대로 재정의하지 않는다면 쓸모없는 메시지만 로그에 남을 것이다.
- phoneNumber용 toString을 제대로 재정의했다면 다음 코드만으로 문제를 진단하기에 충분한 메시지를 남길 수 있다.
    - System.out.println(phoneNumber + "에 연결할 수 없습니다.")

### 어떻게 재정의 해야할까?
- 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.
- 하지만 객체가 거대하거나 문자열로 표현하기에 적합하지 않다면 무리가 있다. 이럴때는 "맨해튼 거주자 전화번호부(총9412323개)"와 같이 요약정보를 담아야 한다.
- 이상적으로는 스스로를 완벽히 설명하는 문자열이어야 한다.

## 반환값의 포맷 문서화 여부
- toString을 구현할 때 반환값의 포맷을 문서화할지 정해야 한다.
- 전화번호나 행렬 같은 값 클래스라면 문서화하기를 권한다.
- 포맷을 명시하면 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 된다.
- 포맷을 명시하기로 했다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공해주면 좋다. ex) BigInteger, BigDecimal
- 포맷을 명시하든 아니든 의도를 명확히 밝혀야 한다.

### 포맷을 명시할 때의 단점
- 포맷을 한번 명시하면 평생 그 포맷에 얽매이게 된다.
- 처음에 명시한대로 프로그래머들이 코드를 작성할 것인데 향후 릴리스에서 포맷을 바꾼다면 이를 사용하던 코드들과 데이터들은 엉망이 될 것이다.
- 반대로 포맷을 명시하지 않는다면 향후 릴리스에서 정보를 더 넣거나 포맷을 개선할 수 있는 유연성을 얻게 된다.


### 포맷을 명시하는 경우
```java
/**
* 이 전화번호의 문자열 표현을 반환한다.
* 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
* XXX는 지역코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
* 각각의 대문자는 10진수 숫자 하나를 나타낸다.
* 
* 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
* 앞에서부터 0으로 채워나간다. 예컨데 가입자 번호가 123이라면
* 전화번호의 마지막 네 문자는 "0123"이 된다.
*/
@Override 
public String toString() {
    return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}
```

### 포맷을 명시하지 않는 경우
```java
/**
* 이 약물에 관한 대략적인 설명을 반환한다.
* 다음은 이 설명의 일반적인 형태이나,
* 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
* 
* "[약물 #9: 유형-사랑, 냄새=테러빈유, 겉모습=먹물]"
*/
@Override
public String toString() { ... }
```

### 주의사항
- 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.
    - toString에 포함되는 정보는 다른 API를 통해 가져갈 수 있도록 각 필드의 getter 메서드를 제공해야 한다.
- 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스라면 toString을 재정의해주어야 한다.

### toString 재정의가 필요 없는 경우
- 정적 유틸 클래스 (상태정보가 없으므로)
- enum 타입 (자바가 이미 완벽한 toString을 제공한다.)

### 핵심 정리
- 모든 구체 클래스에서 Object의 toString을 재정의하자. 상위 클래스에서 이미 알맞게 재정의한 경우는 예외다.
- toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다.
- toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.