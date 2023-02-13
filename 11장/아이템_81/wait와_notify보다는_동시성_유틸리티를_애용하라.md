

# wait와 notify보다는 동시성 유틸리티를 애용하라

자바 5에서 도입된 고수준의 동시성 유틸리티가 이전이라면 wait와 notify로 하드코딩해야 했던 전형적인 일들을 대신 처리해주기 때문에 지금은 wait과 notify를 사용해야 할 이유가 많이 줄었다.

>갖고 있던 고유 락을 해제하고, 스레드를 잠들게 하는 wait와 잠들어 있던 스레드 중 임의로 하나를 골라 깨우는 notify는 synchronized 블록이나 메소드에서 호출되어야하고, 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자.

### java.util.concurrent의 고수준 유틸리티는 세 범주로 나눌 수 있다.
실행자 프레임워크, 동시성 컬렉션, 동기화 장치다.

## 동시성 컬렉션
동시성 컬렉션은 List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다. 
- 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다.
- 따라서 동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.

동시성 컬렉션에서 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 일 역시 불가능하다.

그래서 여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메서드들이 추가되었다.   
이 메서드들은 아주 유용해서 자바 8에서는 일반 컬렉션 인터페이스에도 디폴트 메서드형태로 추가되었다.

#### 예) Map의 putIfAbsent(key, value)
예를 들어 Map의 putIfAbsent(key, value) 메서드는 주어진 키에 매핑된 값이 아직 없을때만 새 값을 집어넣는다.   
그리고 기존 값이 있었다면 그 값을 반환하고, 없었다면 null을 반환한다.

이 메서드 덕에 스레드 안전한 정규화 맵을 쉽게 구현할 수 있다.

#### 예) String.intern
String의 intern 메소드를 간략하게 말하면, String pool에서 literal이 존재하는지 확인하고, 존재한다면 그 문자열을 반환, 존재하지 않는다면(new String("")을 통해 생성되어 literal에 존재하지 않는 경우) String pool에 문자열을 등록하고 그 문자열을 반환한다.
```java
//ConcurrentMap으로 구현한 동시성 정규화 맵 - 최적은 아니다.
public class Intern {
    private static final ConcurrentMap<String, String> map =
            new ConcurrentHashMap<>();

   public static String intern(String s) {
       String previousValue = map.putIfAbsent(s, s);
       return previousValue == null ? s : previousValue;
   }
}
```
아직 개선할 게 남아있다. ConcurrentHashMap은 동시성을 보장하니 내부적으로 synchronized를 사용하지만, get 같은 검색 기능에는 동기화를 사용하지 않도록 최적화되어 있다. 

get을 먼저 호출하여 필요할 때만 putIfAbsent를 호출하면 더 빠르다.
```java
public static String intern(String s) {
    String result = map.get(s);
    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null)
            result = s;
    }
    return result;
}
```

### 컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 기다리도록 확장되었다.
#### 예) Queueq
Queue를 확장한 BlockingQueue 인터페이스에 추가된 메소드 중 take는 큐의 첫 원소를 꺼낸다. 이 때 만약 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다. 이런 특성 덕에 작업 큐(생산자-소비자 큐)로 쓰기에 적합하다.

작업 큐는 하나 이상의 생산자(producer) 스레드가 작업(work)를 큐에 추가하고, 하나 이상의 소비자(consumer) 스레드가 큐에 있는 작업을 꺼내 처리하는 형태다.

간단하다. BlocingQueue의 구현체에서 아무 작업이 없을 때 take 메소드를 호출하고, 다른 스레드에서 BlockingQueue에 어떠한 작업도 offer나 add하지 않는다면 그 스레드는 종료되지 않는다.

## 동기화 장치(synchronizer)
동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다.

가장 자주 쓰이는 동기화 장치는 CountDownLatch와 Semaphore다. 가장 강력한 동기화 장치는 바로 Phaser다.

#### CountDownLatch
CountDownLatch는 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
```java
public CountDownLatch(int count){
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```
유일한 생성자는 int값을 받으며 이 값이 Latch의 countDown 메서드를 몇번 호출해야 대기 중인 스레드들을 깨우는지를 결정한다.

예룰 들어 어떤 동작들을 동시에 시작해 모두 완료하기까지 시간을 재는 간단한 프레임워크를 구축한다고 해보자

```java
/**
 *
 * @param executor 동작들을 실행할 실행자
 * @param concurrency 동작을 몇 개나 동시에 수행할 수 있는지인 동시성 수준
 * @param action 진행할 동작 Runnable
 * @return 동작들이 모두 완료하기까지 걸린 시간
 * @throws InterruptedException
 */
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    // 모든 스레드가 동작 준비가 완료되는 순간을 위한 래치
    CountDownLatch ready = new CountDownLatch(concurrency);
    // 모든 스레드가 동작 준비가 완료되고 작업을 들어가게 하기 위한 래치
    CountDownLatch start = new CountDownLatch(1);
    // 모든 스레드가 동작을 완료한 순간을 위한 래치
    CountDownLatch done  = new CountDownLatch(concurrency);

    for (int i = 0; i < concurrency; i++) {
        executor.execute(() -> {
            // 타이머에게 준비를 마쳤음을 알린다.
            ready.countDown();
            try {
                // 메인 스레드에서 start하게 하기를 기다린다.
                start.await();
                action.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                // 작업이 종료되면 countdown한다.
                done.countDown();
            }
        });
    }
    //concurrency 만큼 countDown이 완료될 때까지 기다린다. 즉, 모든 스레드가 준비될 때까지 대기
    ready.await();
    long startNanos = System.nanoTime();
    // 모든 스레드가 action을 동시에 실행할 수 있도록 카운트를 0으로 만든다.
    start.countDown();
    // 모든 작업이 종료될 떄까지 대기한다.
    done.await();
    // 소요시간을 반환
    return System.nanoTime() - startNanos;
}
```
이상의 기능을 wait와 notify만으로 구현한다면 난해하고 지저분한 코드가 되지만, CountDownLatch를 쓰면 직관적으로 구현할 수 있다.

주의할 점이 있는데, time 메소드에 넘겨진 executor는 concurrency 매개변수로 지정한 동시성 수준만큼 스레드를 생성할 수 있어야한다. 그렇지 못하면 이 메소드는 결코 끝나지 않는다. 

ready CountDownLatch는 concurrency만큼 countdown이 되야 메인 스레드에서 시간을 측정할 수 있고, 다른 스레드들에서는 ready가 await가 끝나고 startNanos를 기록 후 start가 countdown이 될 때까지 await하고 있다. executor가 concurrency만큼 스레드를 생성할 수 없다면 countdown이 진행되지 못하니 이 메소드가 끝날 수 없는 것이다.


이런 상태를 ***스레드 기아 교착상태(thread starvation deadlock)*** 라 한다.   
여러 프로세스가 동일 자원 점유를 요청하며, 여러 프로세스가 부족한 자원을 점유하기 위해 경쟁을 해서, 특정 프로세스는 영원히 자원 할당이 되지 않는 것이다. 따라서 따로 예외 처리를 해줘야한다.

### wiat와 notify를 사용해야 하는 경우
새로운 코드라면 언제나 wait와 notify가 아닌 동시성 유틸리티를 써야 한다. 하지만 어쩔 수 없이 레거시 코드를 다뤄야 할 때도 있다.
- wait 메소드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다.
- 락 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다.

```java
synchronized (obj) {
    while (<조건 미충족>) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡기
    }
    ... // 조건 충족시 동작 수행
}
```

> wait 메소드를 사용할 땐 반드시 대기 반복문 (wait loop) 관용구를 사용하고 반복문 밖에서는 절대 호출하지 말자.

대기 전 조건을 검사해 조건이 이미 충족되었다면 wait을 건너뛰게 하는 것은 응답 불가 상태를 예방하는 조치다.
만약 조건이 충족되었는데 다른 스레드가 notify 혹은 notifyAll 메소드를 먼저 호출한 후 대기 상태로 빠지면 그 스레드는 다시 깨울 수 있다고 보장할 수 없다.

***대기 후에 조건을 검사해 조건이 충족되지 않았다면 다시 대기하는 것은 안전 실패를 막는 조치다.***
만약 조건이 충족되지 않았는데 스레드가 동작을 이어가면 락이 보호하는 불변식을 깰 위험이 있다. 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황의 예시를 살펴보자.

- 스레드가 notify를 호출한 다음 대기 중이던 스레드가 깨어나는 사이 다른 스레드가 락을 얻어 동기화 블럭 안의 상태를 변화시킬 수 있다.
- 조건이 만족되지 않았는데 다른 스레드가 실수 혹은 악의적으로 notify를 호출할 수 있다. 공개된 객체를 락으로 사용해 대기하는 클래스는 이런 위험에 노출되고, 외부에 노출된 객체의 동기화된 메소드 안에서 호출하는 wait는 모두 이 문제에 영향을 받는다.
- 깨우는 스레드의 관대함에, 대기 중인 스레드 중 일부만 조건이 충족되도 notifyAll을 호출해 모든 스레드를 깨울 수 있다.
- 대기중인 스레드가 notify 없이 깨어날 수 있는데 허위 각성(spurious wakeup) 현상이다.


### notify와 notify 중 무엇을 선택해야 할까?
notify는 스레드 하나만 깨우며, notifyAll은 모든 스레드를 깨운다.

일반적으로 언제나 notifyAll을 사용하는게 낫다.
깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 수 있을 것이고, 다른 스레드까지 깨어날 수도 있긴 하지만 정확성에는 영향을 주지 않는다. 깨어난 스레드들은 기다리던 조건이 충족되었는지 확인하여, 충족되지 않았다면 다시 대기할 것이기 때문이다.

--- 
### 핵심 정리
- wait과 notify를 직접 사용하는 것은 동시성 '어셈블리 언어'로 프로그래밍 하는 것에 비유할 수 있다.
- 반면 java.util.concurrent는 고수준 언어에 비유할 수 있다.
- 코드를 새로 작성한다면 wait과 notify를 쓸 이유가 거의 없다.
- 이들을 사용하는 레거시 코드를 유지보수해야 한다면 wait은 항상 표준 관용구에 따라 while 문 안에서 호출하도록 하자.
- 일반적으로 notify보다는 notifyAll을 사용해야 한다.


참고   
https://ktaes.tistory.com/87