# equals를 재정의하려거든 hashCode도 재정의하라

`equals`를 재정의하고 `hashCode`를 재정의하지 않는 경우 `hashCode` 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 `HashMap`이나 `HashSet`같은 컬렉션의 원소로 사용할 때 문제가 생긴다.
###
Object 명세에서 발췌한 규약을 살펴보자

> 1. `equals`비교에 사용되는 정보가 변경되지 않았다면, 객체의 `hashcode` 메서드는 몇번을 호출해도 항상 일관된 값을 반환해야 한다.   
>     단, 애플리케이션을 다시 실행한다면 값이 달라져도 상관없다.
> 2. `equals(Object)`가 두 개의 객체를 같다고 판단했다면, 두 객체는 똑같은 hashcode 값을 반환해야 한다.
> 3. `equals(Object)`가 두 개의 객체를 다르다고 판단했더라도, 두 객체의 hashcode가 서로 다른 값을 가질 필요는 없다. (Hash Collision)   
>     단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

**`hashCode` 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다.  
즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**
###
예를 들어 `equals` 메서드가 정상적으로 재정의된 `PhoneNumber` 클래스의 인스턴스를 `HashMap`의 원소로 사용한다고 해보자.
```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

m.get(new PhoneNumber(707, 867, 5309)) // null 반환
```
위의 코드는 사실상 다른 2개의 `PhoneNumber` 인스턴스가 사용되었다.  
`PhoneNumber` 클래스는 `hashCode`를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 두 번째 규약을 지키지 못한다.  
`HashMap`은 해시코드가 다른 엔트리 끼리는 동치성 비교를 시도조차 하지 않도록 최적화되어 있어, 설사 두 인스턴스를 같은 버킷으로 담았더라도 `get` 메서드는 `null`을 반환한다.  
###
## hashCode 구현방법
### 최악의 hashCode 구현방법
```java
@Override
public int hashCode() { return 42; }
```
이 코드는 동치인 모든 객체에서 똑같은 해시코드를 반환하니 적법하나 모든 객체가 해시테이블 버킷 하나에 담겨 마치 연결리스트처럼 동작한다.  
그 결과 평균 수행 시간이 `O(1)`인 `HashTable`이 `O(n)`으로 느려진다.

### 좋은 해시함수 만들기
좋은 해시함수라면 서로 다른 인스턴스에 다른 해시코드를 반환하며 이것이 `hashCode`의 세 번째 규약이 요구하는 속성이다.

좋은 hashCode를 작성하는 간단한 요령을 알아보자.
```java
@Override
public int hashCode() {
    int c = 31;
    
    //1. int변수 result를 선언한 후 첫번째 핵심 필드에 대한 hashcode로 초기화 한다.
    int result = Integer.hashCode(firstNumber);

    //2. 기본타입 필드라면 Type.hashCode()를 실행한다
    //   Type은 기본타입의 Boxing 클래스이다.
    result = c * result + Integer.hashCode(secondNumber);

    //3. 참조타입이라면 참조타입에 대한 hashcode 함수를 호출 한다.
    //4. 값이 null이면 0을 더해 준다.
    result = c * result + address == null ? 0 : address.hashCode();

    //5-1. 필드가 배열이라면 핵심 원소 각각을 별도 필드처럼 다룬다.
           위의 규칙을 재귀적으로 적용해 각 핵심 원소를 해시코드를 계산한다.
           만약 배열에 핵심 원소가 하나도 없다면 단순히 상수(0 추천)를 사용한다.
    for (String elem : arr) {
        result = c * result + elem == null ? 0 : elem.hashCode();
    }

    //5-2. 배열의 모든 원소가 핵심필드이면 Arrays.hashCode를 이용한다.
    result = c * result + Arrays.hashCode(arr);

    //6. result = 31 * result + hashCode 형태로 초기화 하여 result를 리턴
    return result;
}
```
#### 그렇다면 왜 31을 사용할까?
`31`은 홀수이면서 소수이기 때문이다.  
만약 이 숫자가 예를 들어 짝수 `2`라면 `2`를 곱하는 것은 `시프트 연산`과 같은 결과를 내기 때문에 오버플로우가 발생한다면 정보를 잃게 된다.

### 전형적인 hashCode 메서드
```java
@Override 
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * Short.hashCode(prefix);
    result = 31 * Short.hashCode(lineNum);
    return result;
}
```

### 한줄짜리 hashCode 메서드
```java
@Override 
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```
이 경우 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본타입이 있다면 `박싱`과 `언박싱`도 거쳐야 하기 때문에 속도가 더 느리다.
### hashCode의 캐싱과 지연 초기화
클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 **캐싱**하는 방식을 고려해야 한다.  
이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해둬야 한다.

아래는 `hashCode`가 처음 불릴 때 계산하는 지연 초기화 전략을 사용하는 코드이다.  
필드를 지연 초기화 하려면 그 클래스가 thread-safe 되도록 동기화에 신경 쓰는 것이 좋다.
```java
private int hashCode;   // 자동으로 0으로 초기화된다.

@Override
public int hashCode() {
        int result = hashCode;
        
        if(result == 0) {
            int result = Integer.hashCode(areaCode);
            result = 31 * result + Integer.hashCode(areaCode);
            result = 31 * result + Integer.hashCode(areaCode);
            hashCode = result;
        }
        
        return result;
}
```

## 결론
`equals`를 재정의할 때는 `hashCode`도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다.  
재정의한 `hashCode`는 `Object`의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.  
`AutoValue` 프레임워크를 사용하면 멋진 `equals`와 `hashCode`를 자동으로 만들어준다.

## 참고
- https://velog.io/@nnakki/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%9411.-equals%EB%A5%BC-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%A0%A4%EA%B1%B0%EB%93%A0-hashcode%EB%8F%84-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC