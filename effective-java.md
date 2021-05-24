# Effective Java

## 7장. 람다와 스트림

------



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
------
      
      
      
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
------



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
------
    
    
    
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
------
    
    
    
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

**Collectors 의 자세한 이해를 위해서는 java.util.stream.Collectors의 API 문서( http://bit.ly/2MvTOAR )을 참조**

**가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining 이다.**

------
    
    
    
### 아이템47 반환 타입으로는 스트림보다 컬렉션이 낫다.

반환 원소들이 기본 타입이거나 **성능에 민감**한 상황이리면 **배열**을 써라.

코드 47-1 자바 타입 추론의 한계로 컴파일 되지 않는다.
```java
for (ProcessHandle ph : ProcessHandle.allProcess()::iterator) {
    // 프로세스를 처리한다.
}
```

아쉽게도 위 코드는 다음의 컴파일 오류를 낸다.

```java
Test.java:6: error: method reference not expected here
    for(ProcessHandle ph : "P"rocessHandle.allProcesses()::iterator){
```

이 오류를 잡으려면 메서드 참조를 매개변화된 Iterable로 적절히 형변환 해줘야 한다.

코드 47-2 스트림을 반복하기 위한 '끔직한' 우회 방법

```java
for (ProcessHandle ph : (Iterable<ProcessHandle>)ProcessHandle.allProcess()::iterator) {
    // 프로세스를 처리한다.
}
```

작동은 하지만 너무 난잡하고 직관성이 떨어짐

코드 47-3 Stream<E>를 Iterable<E>로 중개해주는 어댑터

```java
public static <E> Iterable<E> iterableOf (Stream<E> stream) {
    return stream::iterator;
}
```

위 어댑터를 사용해 어떤 스트림도 for-each 문으로 반복 가능

```java
for (ProcessHandle p : iterableOf(ProcessHandle.allProcess())) {
    // 프로세스를 처리한다
}
```



Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다. 따라서 **원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.**



원소 개수가 n개면 **멱집합**의 원소 개수는 2<sup>n</sup> 개가 된다.

예) {a,b,c} 의 멱집합은 { {}, {a}, {b}, {c}, {a,b}, {a,c}, {b,c}, {a,b,c} } 다.

코드 47-5 입력 집합의 멱집합을 전용 컬렉션에 담아 반환한다.

```java
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30) // 입력 집합의 원소 수가 30을 넘으면 PowerSet.of 가 예외 던짐
            throw new IllegalArgumentException ("집합에 원소가 너무 많습니다. 최대 30개.:"+s);
        
        return new AbstractList<Set<E>>() {
            @Override
            public int size() {
                // 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱 것과 같다.
                return 1 << src.size();
            }
            
            @Override
            public boolean contains (Object o) {
                return o instanceof Set && src.containsAll( (Set)o );
            }
            
            @Override
            public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i=0; index != 0; i++, index >>= 1)
                    if ( (index & 1) == 1 )
                        result.add(src.get(i));
                return result;
            }
        };
    }
}
```


------

    
    
### 아이템48 스트림 병렬화는 주의해서 적용하라

데이터 소스가 Stream.iterate거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.

대체로 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다.

참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 아주 중요한 요소로 작용한다. 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다. 기본 타입 배열에서는 (참조가 아닌) 데이터 자체가 메모리에 연속해서 저장되기 때문이다.

종단 연산 중 병렬화에 가장 적합한 것은 축소(reduction)다. (min, max, count, sum, anyMatch, allMatch 등)

직접 구현한 Stream, Iterable, Collection이 병렬화의 이점을 제대로 누리게 하고 싶다면 <u>spliterator 메서드를 반드시 재정의</u>하고 결과 스트림의 병렬화의 성능을 강도 높게 테스트하라.

스트림 안의 원소 수와 원소당 수행되는 코드 줄 수를 곱해보자. 이 값이 최소 수십만은 되어야 성능 향상을 맛볼 수 있다.

단, 조건이 잘 갖춰지면 parallel 메서드 호출 하나로 거의 프로세스 코어 수에 비례하는 성능 향상을 만끽할 수 있다.

코드 48-2 소수 계산 스트림 파이프라인 - 병렬화에 적합하다.

```java
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}
```

코드 48-3 소수 계산 스트림 파이프라인 - 병렬화 버전

```java
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
        .parallel()
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}
```

코드 48-2 로 파이(10<sup>8</sup>)을 계산하는데 31초, 코드 48-3은 parallel() 호출 하나 추가로, 9.2 초로 단축됨.



무작위 수들로 이뤄진 스트림을 병렬화하려거든 ThreadLocalRandom(혹은 구식인 Random)보다는 SplittableRandom 인스턴스를 이용하자. SplittableRandom은 정확히 이럴 때 쓰고자 설계된 것이라 <u>병렬화하면 성능이 선형으로 증가</u>한다.

그냥 Random은 모든 연산을 동기화하기 때문에 병렬 처리하면 최악의 성능을 보일 것이다.

------
    
    
    
    
