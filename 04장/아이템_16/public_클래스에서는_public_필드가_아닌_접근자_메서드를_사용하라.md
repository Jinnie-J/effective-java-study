# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## public 클래스에서의 public 필드 사용 예시
```java
class Point {  
    public double x;
    public double y;
    
    public static void main(String[] args) {
        Point point = new Point();
        point.x = 10;
        point.y = 20;
    }
}
```
위의 `Point` 클래스는 아래와 같은 단점들이 있다.  
- 데이터 필드에 직접 접근할 수 있어 `캡슐화`의 이점을 제공하지 못한다.  
- 필드를 변경하려면 API를 변경해야 한다.
- 필드에 접근할 때 부수 작업을 할 수 없다.
- 불변식을 보장할 수 없다.
  > **불변식 (Invariant) 이란?**  
  > 프로그램의 실행 중 일정 구간 동안 항상 참이 되는 조건이다.  

### 대표적인 예
`java.awt.package` 의 `Point`와 `Dimension` 클래스다.  
`Dimension` 클래스의 심각한 성능 문제는 오늘날까지도 해결되지 못했다.
###
### 철저한 객체 지향 프로그래머는 필드의 접근 제한자를 모두 `private`으로 바꾸고 `public` 접근자(`getter`)를 추가한다.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }

    public double getY() { return y; }

    public void setX(double x) { this.x = x; }

    public void setY(double y) { this.y = y; }
}
```
위와 같이 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.  
하지만 `package-private` 클래스 혹은 `private 중첩 클래스`라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.

### 불변 필드를 노출한 public 클래스 예시

```java
public final class Time {
    private static final int HOURS_PER_DAY      = 24;
    private static final int MINUTES_PER_HOUR   = 60;
    
    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute);
        this.hour = hour;
        this.minute = minute;
    }
}
```
`public` 클래스의 필드가 불변이라면 직접 노출할 때와 비교해서 불변식은 보장할 수 있지만, 다른 단점들이 여전히 존재한다.  
예를 들어 Time 클래스는 각 인스턴스가 유효한 시간을 표현함을 보장한다.

## 결론
`public` 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.  
불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.  
하지만 `package-private` 클래스너 `private 중첩 클래스`에서는 종종 불변 혹은 가변 필드를 노출하는 편이 나을 때도 있다.