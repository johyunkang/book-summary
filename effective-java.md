# Effective Java

## 7장. 람다와 스트림

### 아이템42. 익명 클래스보다는 람다를 사용하라

```java
# 기존 방식
Collections.sort(words, new Comparator<String>(){
    public int compare(String s1, String s2){
        return Integer.compare(s1.length(), s2.length());
    }
})

# 람다식 사용
Collections.sort(words, (s1, s2) -> 
                 Integer.compare(s1.length(), s2.length()));
                 
# 람다 자리에 비교자 생성 메서드를 사용하여 코드를 더 간결하게
Collections.sort(words, comparingInt(String::length));

# java 8 List 인터페이스에 추가된 sort 메서드를 이용하여 더 간결하게
words.sort(comparingInt(String::length));
```

**타입을 명시해야 코드가 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.**

**람다에서 this 키워드는 바깥 인스턴스를 가리킨다.**


### 아이템43. 람다보다는 메서드 참조를 사용하라

**메서드 참조의 유형 5가지**

- 정적 메서드를 가리키는 메서드 참조

- 수신 객체(receiving object; 참조 대상 인스턴스)를 특정하는 한정적(bound) 인스턴스 메서드 참조

- 수신 객체를 특정하지 않는 비한정적(unbound) 인스턴스 메서드 참조

- 클래스 생성자를 가리키는 메서드 참조

- 배열 생성자를 가리키는 메서드 참조

| 메서드 참조 유형   | 예                       | 같은 기능을 하는 람다                              |
| ------------------ | ---------------------- | -------------------------------------------------- |
| 정적               | Integer::parseInt      | str -> Integer.parseInt(Str)                       |
| 한정적(인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now(); t -> then.isAfter(t) |
| 비한정적(인스턴스) | String::toLowerCase    | str -> str.toLowerCase()                           |
| 클래스 생성자      | TreeMap<K,V>::new      | () -> new TreeMap<K,V>()                           |
| 배열 생성자        | int[]::new             | len -> new int[len]                                |

**람다로는 불가능하나 메서드 참조로는 가능한 유일한 예는 바로 제네릭 함수 타입(generic function type) 구현이다**



### 아이템44. 표준 함수형 인터페이스를 사용하라

**필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라. (java.util.function 패키지)**

java.util.fuction 패키지에는 총 **43개** 의 인터페이스가 담겨 있다. 기본 인터페이스 **6개**만 기억하면 나머지는 유추가능

| 인터페이스        | 함수 시그니처       | 예                  |
| ----------------- | ------------------- | ------------------- |
| UnaryOperator<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add     |
| Predicate<T>      | boolean test(T t)   | Collection::isEmpty |
| Function<T, R>    | R apply(T t)        | Arrays::asList      |
| Supplier<T>       | T get()             | Instant::now        |
| Consumer<T>       | void accept(T t)    | System.out::println |

**기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자.** 동작은 하지만 "박싱된 기본 타입 대신 기본 타입을 사용하라"라는 아이템 61의 조언을 위배함. 특히 **계산량이 많을 때는 성능이 처참히 느려질 수 있다.**

직접 만든 함수형 인터페이스에는 항상 **@FunctionalInterface** 애너테이션을 사용하라.



### 아이템45. 스트림은 주의해서 사용하라

**스트림 API가 제공하는 추상 개념 중 핵심은 두 가지다.**

- 스트림(stream)은 데이터 원소의 유한 혹은 무한 시퀀스(sequence)를 뜻한다.

- 스트림 파이프라인(stream pipeline)은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

스트림 파이프라인은 **지연평가(lazy evaluation)**된다. 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 이러한 **지연평가**가 무한 스트림을 다룰 수 있게 해주는 열쇠다.


코드 45-2 스트림을 과하게 사용한 예 - 따라하지 말 것!!
```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        
        try (Stream<String> words = Files.lines(dictionary) {
            words.collect(
                groupingBy(word -> word.chars().sorted()
                          .collect(StringBuilder::new,
                            (sb, c) -> sb.append((char) c),
                            StringBuilder::append).toString()))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .map(group -> group.size() + ": " + group)
            .forEach(System.out::println);
        }
    }
}
```

스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.

코드 45-3 스트림을 적절히 활용하면 깔금하고 명료해진다.

```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                .values().stream.
                .fileter(group -> group.size() >= minGroupSize)
                .forEach(group -> System.out.println(group.size() +": "+ group));
        }
    }
    // alphabetize 메서드는 코드 45-1과 같다.
}
```

**기존 코드는 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아 보일 때만 반영하자**



**스트림 사용 좋은 예**

- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링 한다.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.(더하기, 연결하기, 최솟값 구하기 등).
- 원소들의 시퀀스를 컬렉션에 모은다.(아마도 공통된 속성을 기준으로 묶어가며).
- 원소들의 시퀀스에 특정 조건을 만족하는 원소를 찾는다.

    
### 아이템46. 스트림에서는 부작용 없는 함수를 사용하라.

**스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임. 스트림이 제공하는 표현력, 속도(사황에 따라서는) 병렬성을 얻으려면 API는 말할 것도 없고 이 패러다임까지 함께 받아들여야 함.**



코드 46-1 스트림 패러다임을 이해하지 못한 채 API만 사용한 예 - 따라하지 말 것
```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

foreEach가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는 것(이 예에서는 람다가 상태를 수정함)을 보니 나쁜 코드일 것 같은 냄새가 남.



코드 46-2 스트림을 제대로 활용해 빈도표를 초기화 함.

```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBY(String::toLowerCase, counting()));
}
```

**forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자.**

**수집기(collector) : 스트림을 사용하라면 꼭 배워야 하는 새로운 개념**

java.util.stream.Collectors 클래스는 메서드를 무려 **39개**나 가지고 있음.

수집기는 총 3가지로, **toList(), toSet(), toCollection(collectionFactory)**

코드 46-3 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인

```java
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```

* 마지막 toList는 Collectors의 메서드다. 이처럼 Collectors의 멤버를 정적 임포트하여 쓰면 스트림 파이프라인 가독성이 좋아져, 흔히들 이렇게 사용한다.

**Collectors 의 자세한 이해를 위해서는 java.util.stream.Collectors의 API 문서(http://bit.ly/2MvTOAR)을 참조**

**가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining 이다.**


