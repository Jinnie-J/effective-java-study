
## ordinal 인덱싱 대신 EnumMap을 사용하라

이따금 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 코드가 있다. 식물의 생애주기를 열거 타입으로 표현한 예를 살펴보자.

```java
public class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```
### ordinal 인덱싱의 단점
정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기(한해살이, 여러해살이, 두해살이)별로 묶어보자.
```java
public static void usingOrdinalArray(List<Plant> garden) {
        Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];
        for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
            plantsByLifeCycle[i] = new HashSet<>();
        }

        for (Plant plant : garden) {
            plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
        }
        // 결과 출력
        for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
            System.out.printf("%s : %s%n",
                    LifeCycle.values()[i], plantsByLifeCycle[i]);
        }
    }
```
위 코드는 집합들을 배열 하나에 넣고 생애주기의 orrinal 값을 그 배열의 인덱스로 사용한 코드이다.

1. Set 배열을 생성해 LifeCycle별로 관리한다. 3개의 배열이 만들어 질 것이고, 각 배열을 순회하며 빈 HashSet으로 초기화 해준다.
2. 각 plant 들을 배열의 Set에 추가한다. 이때 plant가 가지고 있는 LifeCycle 열거 타입의 ordinal 값으로 배열의 인덱스를 결정한다. 그 결과 식물의 LifeCycle 별로 Set에 추가된다.
3. 결과를 출력한다. 열거 타입의 values로 반환되는 열거 타입 상수 배열의 순서는 ordinal 값으로 결정되기 때문에 배열의 각 Set이 의미하는 LifeCycle은 values의 순서와 같을 것이다.

예시
```java
public class LifeCycleExample{
    public static void main(String[] args){
        Plant corn = new Plant("옥수수", LifeCycle.ANNUAL);
        Plant pea = new Plant("완두", LifeCycle.ANNUAL);
        Plant potato = new Plant("감자", LifeCycle.PERENNIAL);
        Plant alceaRosea = new Plant("접시꽃", LifeCycle.BIENNIAL);

        List<Plant> garden = Arrays.asList(corn,potato,alceaRosea,pea);

        usingOrdinaryArray(garden);
    }
}
/* 
ANNUAL: [완두, 옥수수]
PERENNIAL: [감자]
BIENNIAL: [접시꽃]
*/
```

동작은 하지만 문제가 한가득이다.
- 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 한다.
- 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
- 정수는 열거 타입과 달리 타입 안전하지 않기 때문에 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다.

### EnumMap
이러한 단점들을 해결할 멋진 해결책이 있다. 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체가 존재하는데, 바로 ***EnumMap*** 이다.

> Map 인터페이스 중 key 값으로 enum의 상수 객체만 사용하도록 하는 구현체.

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
        plantsByLifeCycle.put(lc, new HashSet<>());
}

for (Plant plant : garden) {
        plantsByLifeCycle.get(plant.lifeCycle).add(plant);
}

//EnumMap은 toString을 재정의하였다.
System.out.println(plantsByLifeCycle);

```
- 더 짧고 명료하고 안전하고 성능도 원래 버전과 비등하다.
- 안전하지 않은 형변환은 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 일도 없다.
- ordinal을 이용한 배열 인덱스를 사용하지 않으므로 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 없어진다.
- EnumMap은 내부에서 배열을 사용하기 때문에 내부 구현 방식을 안으로 숨겨서 Map의 타입 안정성과 배열의 성능을 모두 얻어낸 것이다. (생성자로 key type(enum) 을 넘겨주면, enum 에 정의된 열거형 상수의 개수만큼 내부적으로 Object 배열을 생성한다.)

여기서 EnumMap의 생성자는 한정적 타입 토큰인 키 타입의 Class 객체를 받는다. 이는 제네릭의 타입 정보가 런타임시에 소거되기 때문에 런타임 제네릭 타입 정보를 제공하기 위해 키 타입의 Class 객체를 받도록 하였다.

### 스트림 사용
스트림을 사용해 맵을 관리하면 코드를 더 줄일 수 있다.
```java
//streamV1 - EnumMap을 사용하지 않는다.
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));

//streamV2 - EnumMap을 이용해 데이터와 열거 타입을 매핑했다.
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet())));
```
Collectors의 groupingBy 메소드를 이용하여 맵을 구성하였는데, streamV1과 streamV2 메소드의 차이는 groupingBy 메소드에 원하는 맵 구현체를 명시하였는가의 차이다.
- V1 메소드의 반환 맵은 HashMap을 사용하고 Key에 대응되는 Value는 ArrayList로 구성된다.
- V2 메소드는 맵 구현체를 명시하였기 때문에 EnumMap을 사용하고 Value는 HashSet으로 구성된다.

#### EnumMap과 HashMap의 차이점
- EnumMap은 key값으로 enum의 상수 객체만을 가질 수 있다. HashMap은 키값에 대한 특별한 제한이 없다.
- EnumMap은 내부적으로 배열로 구현되어있고, HashMap은 내부적으로 HashTable로 구현되어있다.
- EnumMap은 내부적으로 배열로 구현되어 있으므로 EnumMap이 HashMap 보다 성능이 좋다.
- EnumMap은 해싱작업을 하지 않으므로 해시 충돌이 없다.
- EnumMap은 키가 순서대로 표시되지만, HashMap은 순서를 보장하지는 못한다.
- HashMap은 일정 이상의 자료가 들어오면 자체적으로 resizing을 하지만 EnumMap은 enum이 가지고있는 상수 객체의 수가 정해져있기 때문에 크기가 변하거나 하지 않는다.


스트림을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작한다.
- EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.
- 정원에 한해살이와 여러해살이 식물만 살고 두해살이는 없다면, EnumMap 버전에서는 맵을 3개 만들고 스트림 버전에서는 2개만 만든다.

```
//EnumMap 사용
{ANNUAL=[원두, 옥수수], PERNNIAL=[], BIENNIAL=[접시꽃]}

// 스트림 사용
{ANNUAL=[원두, 옥수수], BIENNIAL=[접시꽃]}
```

### ordinal을 사용한 예제
조금 더 복잡한 예제를 보자.

두 가지 상태(Phase)를 전이(Transition)와 매핑하는 예제이다. 액체(LIQUID)에서 고체(SOLID)의 전이는 응고(FREEZE)가 되고, 액체(LIQUID)에서 기체(GAS)로의 전이는 기화(BOIL)가 된 다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT,FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
        // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };
        // 한 상태에서 다른 상태로의 전이를 반환한다.
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```
이 예제는 앞의 정원 예제와 마찬가지로 컴파일러는 ordinal과 배열 인덱스의 관계를 알 도리가 없다.    
즉, Phase나 Pase.Transition 열거 타입을 수정하면서 상전이 표 TRANSITIONS를 함께 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 날 것이다.

### 중첩 EnumMap 사용
다시 이야기하지만 EnumMap을 사용하는 편이 훨씬 낫다.   
- 전이 하나를 얻으려면 이전 상태(from)와 이후 상태(to)가 필요하니, 맵 2개를 중첩하면 쉽게 해결할 수 있다.
- 안쪽 맵은 이전 상태와 전이를 연결하고 바깥 맵은 이후 상태와 안쪽 맵을 연결한다.
- 전이 전후의 두 상태를 전이 열거 타입 Transition의 입력으로 받아, 이 Transition 상수들로 중첩된 EnumMap을 초기화하면 된다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }
        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>> 
                m = Stream.of(values())
                    .collect(Collectors.groupingBy(t -> t.from, //바깥 Map의 Key
                            () -> new EnumMap<>(Phase.class), //바깥 Map의 구현체
                            Collectors.toMap(t -> t.to, //바깥 Map의 Value, 안쪽 Map의 Key
                                    t -> t, //안쪽 Map의 Value 
                                    (x,y) -> y, 
                                    () -> new EnumMap<>(Phase.class)))); //안쪽 Map의 구현체

        public static Transition from(Phase from, Phase to) {
            return transitionMap.get(from).get(to);
        }
    }
}
/* 결과
{SOLID={LIQUID=MELT, GAS=SUBLIME}, 
LIQUID={SOLID=FREEZE, GAS=BOIL}, 
GAS={SOLID=DEPOSIT, LIQUID=CONDENSE}}
*/
```
- Transition 열거 타입은 각 전이에 맞는 이전 상태와 이후 상태를 필드로 가지고 있는 것으로 수정하였다.
- 기존의 2차원 배열을 사용하던 것을 EnumMap으로 사용하기 위해 java.util.stream 패키지의 Collectors 메소드 중 groupingBy와 toMap을 사용하였다.
- 첫번 째 수집기인 groupingBy에서는 전이를 이전 상태를 기준으로 묶고, 두 번째 수집기인 toMap에서는 이후 상태를 전이에 대응시키는 EnumMap을 생성한다.
- GroupingBy는 하나의 Key의 여러 개의 Value를 가지는 Map을 반환하고 toMap은 하나의 Key에 하나의 Value를 가지는 Map을 반환한다.
- groupbingBy에서 Key 값을 전이(transition)를 이전 상태를 기준 으로 바깥 쪽 Map을 묶고 toMap에서 이후 상태를 기준으로 안쪽 Map을 묶는다.
- toMap 메소드에서 (x,y) -> y 부분은 mergeFunction 으로 만약 Key 값이 같은게 존재할때 Value를 기존값(x)으로 할지 새로운값(y)으로 갱신할지 정하는 부분이다. 이번 예제에선 중복되는 Key값이 존재하지 않으므로 실제로는 사용되지 않는다.

여기에 새로운 상태인 플라스마(PLASMA)를 추가해보자.  
- 플라스마와 연결된 전이는 2개다.
- 첫 번째는 기체에서 플라스마로 변하는 이온화(IONIZE)이고, 둘째는 플라스마에서 기체로 변하는 탈이온화(DEIONIZE)다.
- 배열로 만든 코드를 수정하려면 비용이 많이든다. 원소 수를 너무 적거나 많이 기입하거나, 잘못된 순서로 나열하면 런타임 오류가 발생할 것이다.

EnumMap 버전에서는 상태 목록에 플라스마를 추가하고, 전이 목록에 IONIZE(GAS, PLASMA)와 DEIONIZE(PLASMA, GAS)만 추가하면 끝이다.

```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA),
        DEIONIZE(PLASMA, GAS);
    }
}
```
- 나머지는 기존 로직에서 잘 처리해주어 잘못 수정할 가능성이 극히 작다.

---
### 핵심 정리
배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라.   
다차원 관계는 EnumMap<..., EnumMap<...>> 으로 표현하라. 