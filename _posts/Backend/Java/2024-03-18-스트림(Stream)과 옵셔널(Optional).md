---
layout: post
title:  "스트림(Stream)과 옵셔널(Optional)"
date:   2024-03-18 21:00:00 +0900
categories: [Backend, Java]
tags: [Java, Stream, Optional]
lastmod : 2024-03-18 21:00:00 +0900
sitemap :
  changefreq : daily
  priority : 1.0
---
# 스트림이란?

java8 부터 추가된 기능으로 데이터의 흐름에서 원하는 조건을 거는 filter, 필터링된 값을 담는 map, 최종 결과물 만들기(Collect) 를 수행한다.<br>
이전에도 외부반복자(for, while 등)를 이용하여 위 행위가 가능했으나, 스트림은 내부반복자를 이용하기 때문에 병렬처리가 쉬워지며 코드가 간결해진다.<br>

![스트림 외부,내부 반복자](/assets/img/Backend/Java/2024-03-18-Stream-Optional/stream-img-1.jpg)

## 스트림 단계
스트림을 생성, 중간연산 마다 중간 스트림 생성, 최종 스트림 생성 과정에서 중간 스트림이 생성될때마다 바로 연산이 진행되지 않는다.(**지연-lazy**)<br>
최종 연산이 시작되면 최초 컬렉션 요소가 중간 스트림에서 연산되기 시작하여 최종 연산까지 수행한다.<br>

### 스트림 생성
 `Collection`과 `Arrays`는 stream() 메서드가 정의되어 있기 때문에 스트림을 생성 할 수 있다.<br>

```java
// Collection to Stream
List<String> list = Arrays.asList("a","b","c");
Stream<String> stream = list.stream();

// Array to Stream
String[] arr = {"a","b","c"};
Stream<String> arrStream = Arrays.stream(arr);
```

### 스트림 연결
`Stream.concat` 을 이용해 **타입이 같은 스트림**을 하나로 연결 할 수 있다.<br>

```java
List<String> list = Arrays.asList("a","b","c");
Stream<String> listStream = list.stream();

List<String> list2 = Arrays.asList("d","e","f");
Stream<String> listStream2 = list.stream();

Stream<String> concatStream = Stream.concat(listStream, listStream2);
```

### 스트림은 일회용이다.
닫힌 스트림은 재사용 불가<br>

```java
int[] findList = {1,0,4,8};
IntStream stream = Arrays.stream(findList);

int firstResult = stream.findFirst().getAsInt();
System.out.println("firstResult = " + firstResult);

int firtAnyRst = stream.findAny().getAsInt();  // Error
System.out.println("firtAnyRst = " + firtAnyRst);

--- console ---
firstResult = 1
Exception in thread "main" java.lang.IllegalStateException: stream has already been operated upon or closed
	at java.base/java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:229)
	at java.base/java.util.stream.IntPipeline.findAny(IntPipeline.java:557)
	at Scratch.main(scratch_2.java:11)
```


### 중간연산
중간연산의 결과를 **스트림으로 반환**하기 때문에 연속해서 여러번 수행이 가능하다.<br>

| 메소드                      | 설명                                                                     |
| ------------------------ | ---------------------------------------------------------------------- |
| `filter(Predicate)`      | 조건이 참이되는 요소들만 필터링 (필터링)                                                |
| `distinct()`             | 중복된 요소 제거 (필터링)                                                        |
| `mapXXX(Function<T,R>)`  | Function<T,R> 을 통해 데이터를 가공한다. (매핑)                                     |
| `flatMap(Function<T,R>)` | 스트림의 요소가 배열 여러개라면 하나로 합친다. (매핑)                                        |
| `sorted()`               | 비교 및 정렬, 비교자가 없다면 사전순으로 정렬 (정렬)                                        |
| `limit(long)`            | 스트림의 개수를 제한하는 기능                                                       |
| `skip(long)`             | 0번째부터 선택한 개수만큼을 제외한 스트림 반환                                             |
| `peek(Consumer<T>)`      | 연산 중간 결과를 확인하기 위해 디버깅에 많이 사용되는 메서드로 forEach와 달리 스트림 요소를 소모하지 않는다. (루핑) |


### 중간연산 예시
```java
int[] filterList = {1,2,3,4,5,6,7};
Arrays.stream(filterList).filter(n -> n>=5).forEach(System.out::print);
// console : 567
// filterList 중 5이상인 값만 필터링하여 새로운 스트림으로 반환


String[] distinctList = {"a","a","b","c"};
Arrays.stream(distinctList).distinct().forEach(System.out::print);
// console : abc
// distinctList 중 중복되는 a를 하나 제거하여 반환

String[] mapList = {"a","b","c"};
Arrays.stream(mapList).map(s->s.toUpperCase()).forEach(System.out::print);
// console : ABC
// 배열요소를 대문자로 변환하는 람다 Function 수행

String[] flatMapList1 = {"a","b","c"};
String[] flatMapList2 = {"x","y","z"};
Stream.of(
        flatMapList1,
        flatMapList2
).flatMap(Arrays::stream).forEach(System.out::print);
// console : abcxyz
// 2개의 배열을 합쳐서 하나의 스트림으로 반환

int[] sortList = {99,24,65,5,12};
Arrays.stream(sortList).sorted().forEach(n -> System.out.print(n+","));
// console : 5,12,24,65,99,
// 작은 순으로 정렬해줌

String[] limitList = {"a","b","c","d","e","f","g"};
Arrays.stream(limitList).limit(3).forEach(System.out::print);
// console : abc
// 배열의 0~2까지 총 3개의 데이터만 조회됨

Arrays.stream(limitList).skip(3).forEach(System.out::print);
// console : defg
// 배열의 0~2까지 총 3개의 데이터를 제외하고 스트림 생성


int[] peekList = {99,24,65,5,12};
Arrays.stream(peekList).skip(1).peek(n->System.out.print(n+",")).sum();
// console : 24,65,5,12,
// skip(1) 을 수행한 중간 결과를 print 한다.
// 최종연산자가 아니므로 맨 뒤 sum()을 이용해 최종연산을 마친다.
// 최종연산자가 없으면 오류가 나진 않으나 print또한 되지 않는다.

```

### 최종연산
중간 연산을 통해 변환된 중간 스트림을 마지막 최종연산을 통해 결과를 표시한다.<br>
중간 연산에서 지연되어있던 모든 연산이 최종 연산시 수행된다.<br>
중간연산과 달리 최종연산은 한번만 사용이 가능하다.<br>

| 메소드                                                                                                 | 설명                                                                                                                                                                                                                                                                                         |
| --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `void forEach(Consumer)`                                                                            | 모든 요소를 for문 돌며 반환 타입이 void 이므로 보통 요소 출력 또는 스케줄링과 같은 리턴값이 필요없는 경우에 사용한다. (루핑)                                                                                                                                                                                                               |
| `boolean anyMatch(Predicate)`                                                                       | 최소 한 개의 요소가 조건을 만족할 경우 true (매칭)                                                                                                                                                                                                                                                           |
| `boolean allMatch(Predicate)`                                                                       | 모든 요소가 조건을 만족해야 true (매칭)                                                                                                                                                                                                                                                                  |
| `noneMatch(Predicate)`                                                                              | 모든 요소가 조건을 만족하지 않아야 true (매칭)                                                                                                                                                                                                                                                              |
| `long count()`                                                                                      | 해당 스트림의 총 개수 (집계)                                                                                                                                                                                                                                                                          |
| `Optional max(Comparator)`                                                                          | Comparator 로 비교한 요소의 최고값 (집계)                                                                                                                                                                                                                                                              |
| `Optional min(Comparator)`                                                                          | Comparator 로 비교한 요소의 최소값 (집계)                                                                                                                                                                                                                                                              |
| `int sum()`                                                                                         | 모든 요소의 총 합 (int, double 만 가능) (집계)                                                                                                                                                                                                                                                         |
| `OptionalDouble average()`                                                                          | 모든 요소의 평균 (int, double 만 가능) (집계)                                                                                                                                                                                                                                                          |
| `T reduce()`                                                                                        | 복잡하니 예시에서 설명                                                                                                                                                                                                                                                                               |
| `<R> R collect (Supplier<R> supplier, ObjDoubleConsumer<R> accumulator, BiConsumer<R, R> combiner)` | Collectors 객체로 리턴<br>- 배열 또는 컬렉션 변경<br>`toArray()`,`toCollection()`, `toList()`, `toSet()`, `toMap()`\|<br>- 연산 메서드<br>`counting()`, `maxBy()`,`minBy()`, `summingBy()`, `averagingInt()`<br>- 소모 메서드<br>`reducing()`,`joining()`<br>- 그룹화 분할<br>`groupingBy()`,`partitioningBy()`<br>(수집) |
| `Optional<T> findFirst()`                                                                           | 스트림의 첫번째 요소를 가지고온다.<br>순서가 중요하다면 `findFirst()`를 써야한다.<br>병렬 스트림의 경우 `findAny()` 를 써야한다.<br>(집계)                                                                                                                                                                                            |
| `Optional<T> findAny()`                                                                             | 스트림의 첫번째 요소를 가지고 온다.<br>순서가 중요하지 않다면 `findFirst()` 보다 좋다.<br>병렬 스트림일 경우 `findAny()` 를 써야한다.                                                                                                                                                                                                |

<br>
※ 스트림 하나에서 max, min, sum, average를 동시에 뽑아내고 싶을 땐 `summaryStatistics()` 을 사용하자<br>
<br>
<br>



### 최종연산 예시
```java
int[] matchList = {1,3,5,7};
boolean anyResult = Arrays.stream(matchList).anyMatch(s -> s == 1);
System.out.println("anyResult = " + anyResult);
boolean allResult = Arrays.stream(matchList).allMatch(s -> s < 10);
System.out.println("allResult = " + allResult);
boolean noneResult = Arrays.stream(matchList).noneMatch(s -> s > 10);
System.out.println("noneResult = " + noneResult);
// console
// anyResult = true
// allResult = true
// noneResult = true


int[] comparatorList = {2,5,8,20};
long countResult = Arrays.stream(comparatorList).count();
System.out.println("countResult = " + countResult);

int maxResult = Arrays.stream(comparatorList).max().getAsInt();
System.out.println("maxResult = " + maxResult);

int minResult = Arrays.stream(comparatorList).min().getAsInt();
System.out.println("minResult = " + minResult);

double averageResult = Arrays.stream(comparatorList).average().getAsDouble();
System.out.println("averageResult = " + averageResult);
// consoel
// countResult = 4
// maxResult = 20
// minResult = 2
// averageResult = 8.75

int reduceResult = Arrays.stream(comparatorList).reduce((a,b) -> a*b).getAsInt();
System.out.println("reduceResult = " + reduceResult);
// console : reduceResult = 1600
/*
* 배열 [2,5,8,20]
* 2 * 5 = 10
* 10 * 8 = 80
* 80 * 20 = 1600
*/


int reduceResult2 = Arrays.stream(comparatorList).reduce(2, (a,b) -> a*b);
System.out.println("reduceResult2 = " + reduceResult2);
// console : reduceResult2 = 3200
/*
* 초기값과 첫 요소와의 연산으로 시작
* 2 * 2 = 4
* 4 * 5 = 20
* 20 * 8 = 160
* 160 * 20 = 3200
*/

@Builder @AllArgsConstructor
static class User {
    String name;
    int age;
}

List<User> userList = new ArrayList<>();
userList.add(User.builder().name("Kim").age(10).build());
userList.add(User.builder().name("Park").age(12).build());
userList.add(User.builder().name("Kwon").age(14).build());
Map<String, Integer> userMap = userList.stream().collect(Collectors.toMap(s -> s.name, s -> s.age));
System.out.println("userMap.toString() = " + userMap.toString());
// console : userMap.toString() = {Kwon=14, Kim=10, Park=12}
// List<User> to Map
// key : name / value : age



int[] findList = {1,0,4,8};
int firstResult = Arrays.stream(findList).findFirst().getAsInt();
System.out.println("firstResult = " + firstResult);
int findAnyResult = Arrays.stream(findList).findAny().getAsInt();
System.out.println("findAnyResult = " + findAnyResult);
// console
// firstResult = 1
// findAnyResult = 1
// 단순 식에서 결과는 동일

```



##  병렬 스트림

<br>
우선 동시성과 병렬성을 알아보자.<br>

![동시성과 병렬성](/assets/img/Backend/Java/2024-03-18-Stream-Optional/stream-img-2.jpg)

<br>
동시성은 하나의 cpu가 하나의 작업만을 실행하지만, 번갈아가면서 작업을 하기 때문에 동시에 처리되는 것 처럼 보일 뿐이다. (하나의 작업을 나누어 실행-병렬스트림일때만)<br>
병렬성은 멀티 코어(CPU)를 이용하여 진짜로 병렬로 실행하는 것이다.<br>
<br>
병렬성 : 데이터 병렬성과 작업 병렬성으로 나뉜다.<br>
<br>

**데이터 병렬성** <br>
전체 데이터를 분할하여 서브 데이터셋으로 만들고 병렬처리해서 작업ㅇ르 빠르게 끝내는것으로 병렬 스트림은 데이터 병렬성을 구현한것.<br>

<br>

**작업 병렬성** <br>
서로 다른 작업을 병렬 처리하는 것으로, 스레드가 이에 해당한다.<br>

<br>

`.stream()` 은 기본적으로 순차 스트림이다.<br>
병렬 스트림을 만드는 방법은 아래와 같다.<br>

1. `.parallelStream()` 이용 <br>
   `List<Integer> list = Arrays.asList(1,2,3);` <br>
   `list.parallelStream().reduce(0, Integer::sum);` <br>
   바로 병렬 스트림 return <br>
2. `.parallel()` 이용 <br>
   `Arrays.stream(findList).parallel().reduce(0, Integer::sum);` <br>
   기존 순차 스트림을 병렬스트림으로 변경 <br>

<br>
<br>

병렬 스트림은 **Fork Join Pool** 을 이용한다. (즉, 병렬 처리를 하기 위해 Fork 하는 리소스와 Join 하는 리소스를 생각해야한다.) <br>
병렬 스트림이 사용할 코어 수는 `-D java.util.concurrent.ForkJoinPool.common.parallelism={core수}` 를 이용하거나 <br>
`ExecutorService` 를 이용해 아래와 같이 사용할 수 있다. <br>

```java
ForkJoinPool customThreadPool = new ForkJoinPool(4);
int first = customThreadPool.submit(
        () -> Arrays.stream(findList).parallel().findFirst().getAsInt()).get();
System.out.println("first = " + first);
customThreadPool.shutdown();
```

<br>

### 병렬스트림 문제
```java
int[] findList = {1,0,4,8};

// 1. 싱글스트림 결과

int singleSum = Arrays.stream(findList).reduce(1, (a,b) -> a+b);
System.out.println("singleSum = " + singleSum);
// console : singleSum = 14

ForkJoinPool customThreadPool = new ForkJoinPool(4);
int parallelSum = customThreadPool.submit(
        () -> Arrays.stream(findList).parallel().reduce(1, (a,b) -> a+b)).get();
System.out.println("parallelSum = " + parallelSum);
customThreadPool.shutdown();
// console : parallelSum = 17
```
<br>

각 cpu 별로 분할하여 처리하는데 reduce에 `identity`  값인 1 이 각 코어마다 연산되어 중복 연산되는 문제가 생김 <br>
identiy 1 값이 4 코어 만큼 연산됨 => 원하던 연산 : (1) + 1 + 0 + 4 + 8  / 실제 연산 : (1 + 1 + 1 + 1) + 1 + 0 + 4 + 8 <br>
<br>
병렬 스트림은 연산 초기화값을 **스트림 외부에서 추가 연산** 해줄 것<br>

```java
ForkJoinPool customThreadPool = new ForkJoinPool(4);
int parallelSum = customThreadPool.submit(
        () -> Arrays.stream(findList).parallel().reduce(0, (a,b) -> a+b)).get()+1;
System.out.println("parallelSum = " + parallelSum);
customThreadPool.shutdown();
// console : parallelSum = 14
```

<br>
그 외 적은 데이터를 병렬스트림으로 작업하게 되면<br>
소스를 분할 / 병합 하는 리소스가 더 많이 들기에 좋은 방법이 아니다.<br>
데이터 이동의 오버헤드가 크므로 데이터 전송시간보다 작업 시간이 오래걸리는 일들을 병렬로 처리하는게 좋다.<br>
<br>

### 결론
병렬 스트림을 사용하기 전 체크리스트<br>
1. 요소의 수와 요소당 처리 시간<br>
   전체 요소 수가 적고 요소당 처리 시간이 짧으면 병렬 스트림보다 일반 스트림이 나을 수 있다.<br>
2. 스트림 소스 종류<br>
   ArrayList는 인덱스로 요소를 관리하여 분리(Fork)와 병합(Join)에 유리하다.<br>
   그 외 HashSet, TreeSet, LinkedList는 병렬 스트림에 불리할 수 있다.<br>
3. 코어 수 (core)<br>
   CPU가 많을수록 병렬 스트림의 성능은 좋다.<br>
   반대로 코어 수가 적다면 일반스트림이 더 빠를 수 있다.<br>
   병렬 스트림으로 할 경우 스레드 수가 증가하여 동시성이 많이 일어나면 오히려 느려질 수 있다.<br>

<br><br>

# `Optional<T>` 와 `OptionalInt`

<br>
optional은 java8에서 새로 나왔다.<br>
`Optional<T>` 객체를 이용해 NPE 를 방지할 수 있다.<br>
`Optional<T>`는 null이 올 수 있는 값을 감싸는 Wrapper 클래스이다.<br>
💡Wrapper 클래스 : Wrapper Class 는 자바 원천타입의 데이터를 서로 형 **변환이 가능하도록** 지원해주는 Class이다. 기본타입(byte, int...) -> 래퍼클래스 (Byte, Integer ..)<br>
<br>
<br>

* `Optional.of(T)` : 절대 null이 아닌 객체로 null이라면 NPE 발생함<br>
* `Optional.ofNullable(T)` : null일수 있는 객체를 ofNullable로 생성하면 orElse, orElseGet, orElseThrow 등을 통해 NPE없이 안전하게 처리할 수있다.<br>

<br>

```java
Optional<String> ofOptional = Optional.of(null);
System.out.println("ofOptional.get() = " + ofOptional.get()); // NPE 발생

Optional<String> ofnullOptional = Optional.ofNullable(null);
System.out.println("ofnullOptional.get() = " + ofnullOptional.get());  // NPE 발생
System.out.println("ofnullOptional.orElse() = " + ofnullOptional.orElse("default")); // console : default
```



## 사용법

### orElse
optional 객체가 null일 경우 orElse()를 통해 기본값을 지정해줄 수 있다.<br>
*orElse는 optional이 null이 아닐때도 실행된다.* <br>
*Optional의 null여부와 관계 없이 실행되므로 불필요한 연산이 실행될 수 있다.* <br>

```java
public T orElse(T other) {
    return value != null ? value : other;
}

Optional<String> ofnullOptional = Optional.ofNullable(null);
System.out.println("ofnullOptional.orElse() = " + ofnullOptional.orElse("default")); // console : default
// > 값이 비어있을경우 default 라는 텍스트로 기본갑시을 지정했다.
```

### orElseGet
Optional 객체가 null일 때 함수를 실행 할 수 있다. <br>
null일 경우 함수를 수행하고 기본값을 return해줘야 한다. <br>

```java
public T orElseGet(Supplier<? extends T> supplier) {
    return value != null ? value : supplier.get();
}

Optional<String> ofnullOptional = Optional.ofNullable(null);
System.out.println("ofnullOptional.orElseGet() = " + ofnullOptional.orElseGet(()->{
    System.out.println("outConsole!!");
    return "default";
}));
// console
// outConsole!!
// ofnullOptional.orElseGet() = default
```

### orElseThrow
Optional 객체가 null일때 NPE외에 직접 설정한 Exception을 throw할 수 있다. <br>

```java
Optional<String> ofnullOptional = Optional.ofNullable(null);
ofnullOptional.orElseThrow(() ->  {
    System.out.println("값이 비어있구나!!!");
    return new RuntimeException("값이 비어있습니다.");  // Error 발생
});
// console
// 값이 비어있구나!!!
// Exception in thread "main" java.lang.RuntimeException: 값이 비어있습니다.
//	at com.example.demo.OptionalStudy.lambda$main$1(OptionalStudy.java:20)
//	at java.base/java.util.Optional.orElseThrow(Optional.java:403)
//	at com.example.demo.OptionalStudy.main(OptionalStudy.java:18)

```



### 객체에 접근

기본적으로 Optional 객체는 `get()` 을 이용하여 접근할 수 있으나, null일 경우 `NoSuchElementException` 이 발생한다. <br>
때문에 객체가 NULL인지 확인 후 접근해야 하는데 관련된 메서드는 아래와 같다. <br>

#### `boolean isPresent()`
값이 존재하는지 boolean으로 반환해주는 옵션 <br>

```java
Optional<String> ofnullOptional = Optional.ofNullable(null);
System.out.println("ofnullOptional = " + ofnullOptional.isPresent());
// console : false
```

#### `void ifPresent(Consumer)`
값이 존재한다면 Consumer가 실행되고, 값이 없다면 실행되지 않는다. <br>

```java
Optional<String> ofnullOptional = Optional.ofNullable(null);
ofnullOptional.ifPresent(s -> System.out.println(s));
// console :

Optional<String> ofnullOptional = Optional.ofNullable("Hi");
ofnullOptional.ifPresent(s -> System.out.println(s));
// console : Hi
```


####  `void ifPresentElse(Consumer, Runnable)`
java9 부터 지원되는 기능으로 `ifPresent()` 가 값이 있을때만 실행 됐다면 없을때도 정의할 수 있다. <br>

```java
Optional<String> ofnullOptional = Optional.ofNullable(null);
ofnullOptional.ifPresentOrElse(
        s -> System.out.println(s),  // 값이 있을때 실행되는 함수
        () -> System.out.println("No Data!")  // 값이 null일때 실행되는 함수
);
// console : No Data!

```


### 객체 반환

#### `U map(Function<T>)`
map에 매개변수로 추출하고자 하는 객체를 선택하여 해당 객체에 대한 기본값을 설정할 수 있다.
```java

@AllArgsConstructor
@Getter static class User {
    private String name;
    private Integer age;
    private Address address;
}

@AllArgsConstructor
@Getter
static class Address{
    private String city;
}

Optional<User> userOptional = Optional.ofNullable(new User("Kim",null, null));
userOptional.map(User::getAddress).map(Address::getCity).ifPresentOrElse(
        s -> System.out.println("s = " + s),
        () -> System.out.println("Null Data")
);
// console : Null Data
```

<br>
User 객체의 getAddress를 이용해 Address객체를 가져온 뒤 다시 Address의 getCity를 이용해 city 객체의 null 여부를 검증한다.<br>
<br>

#### `Optional<T> filter(Predicate)`
옵셔널 객체에서 데이터를 검증하여 검증이 통과된 데이터 결과로 작업을 한다. <br>

```java

// 1. 필터 검증 성공
Optional<User> filterOptional = Optional.ofNullable(new User("Kim",10, null));
filterOptional.filter(user -> user.getAge() > 5)
        .ifPresentOrElse(
                user -> System.out.println("user = " + user.toString()),
                () -> System.out.println("Null!!")
        );
// console : user = OptionalStudy.User(name=Kim, age=10, address=null)

// 2. 필터 검증 실패
Optional<User> filterOptional = Optional.ofNullable(new User("Kim",3, null));
filterOptional.filter(user -> user.getAge() > 5)
        .ifPresentOrElse(
                user -> System.out.println("user = " + user.toString()),
                () -> System.out.println("Null!!")
        );
// console : Null!!
```


