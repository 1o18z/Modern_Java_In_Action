# Chapter 5) 스트림 활용 - 2

&nbsp;

# 5.6 실전 연습

### 1. 2011년에 일어난 모든 트랜잭션을 찾아 오름차순으로 정리하시오

```java
List<Transaction> tr2011 = transactions.stream().filter(t -> t.getYear() == 2011)
	.sorted(comparing(Transaction::getValue)).collect(toList());
```

### 2. 거래자가 근무하는 모든 도시를 중복 없이 나열하시오

```java
List<String> tdCity = transactions.stream().map(t -> t.getTrader().getCity())
	.distinct().collect(toList());
```

- `distinct()` 대신 스트림을 집합으로 변환하는 `toSet()`을 사용할 수도 있다

### 3. 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오

```java
List<Trader> tdTrader = transactions.stream().map(Trasaction::getTrader)
	.filter(transaction -> transaction.getCity().equals("Cambridge"))
	.distinct()
	.sorted(comparing(Trader::getName))
	.collect(toList());
```

### 4. 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오

```java
String traderStr = transactions.stream()
	.map(transaction -> transaction.getTrader().getName())
	.distinct()
	.sorted()
	.reduce("", (n1, n2) -> n1 + n2);
```

### 5. 밀라노에 거래자가 있는가?

```java
boolean milanBased = transactions.stream()
		.anyMatch(transaction -> transaction.getTrader()
				.getCity()
				.equals("Milan"));
```

### 6. 케임브리지에 거주하는 거래자의 모든 트랜잭션값을 출력하시오

```java
transactions.stream()
		.filter(transaction -> "Cambridge".equals(transaction.getTrader().getCity()))
		.map(Transaction::getValue)
		.forEach(System.out::println);
```

### 7. 전체 트랜잭션 중 최댓값은 얼마인가?

```java
Optional<Integer> highesValue = transactions.stream()
		.map(Transaction::getValue)
		.reduce(Integer::max);
```

### 8. 전체 트랜잭션 중 최솟값은 얼마인가?

```java
Optional<Transaction> smallestTransaction = transactions.stream()
		.min(comparing(Transaction::getValue());
```
&nbsp;    
&nbsp;

# 5.7 숫자형 스트림

```java
int calories = menu.stream()
	.map(Dish::getCalories)
	.reduce(0, Integer::sum);
```

- reduce 메서드로 스트림 요소의 합을 구할 수 있다

→ 위 코드에는 박싱 비용이 숨어있음!

(내부적으로 합계를 계산하기 전에 Integer를 기본형으로 언박싱해야 한다)

sum 메서드를 직접 호출하면 좋겠지만 직접 호출할 수 없다

→ map 메서드가 Stream<T>를 생성하기 때문이다!

⇒ **기본형 특화 스트림** 제공!!

- 스트림 API 숫자 스트림을 효율적으로 처리 가능

&nbsp;

## 기본형 특화 스트림

**IntStream**, **DoubleStream**, **LongStream** 제공

- 각각 인터페이스는 sum, max 같이 자주 사용하는 숫자 관련 리듀싱 연산 수행 메서드를 제공
    - 다시 객체를 스트림으로 복원하는 기능도 제공

**→ 오직 박싱 과정에서 일어나는 효율성과 관련! 추가 기능을 제공하지는 X**

&nbsp;

### 숫자 스트림으로 매핑

스트림을 특화 스트림으로 변환할 때는?

- **mapToInt**, **mapToDouble**, **mapToLong** 메서드를 가장 많이 사용한다

→ map과 같은 기능을 수행하지만 Stream<T> 대신 특화된 스트림을 반환

```java
int calories = menu.stream()  // Stream<Dish> 반환
	.mapToInt(Dish::getCalories)  // IntStream 반환
	.sum();
```

- IntStream 인터페이스에서 제공하는 sum 메서드로 합계 구하기 가능
    - 스트림이 비어있으면 sum은 기본값 0을 반환!

  (max, min, average 등도 지원)

&nbsp;

### 객체 스트림으로 복원하기

IntStream은 기본형의 정수값만 만들 수 있다

(IntStream의 map 연산은 int를 인수로 받아서 int를 반환하는 람다를 인수로 받는다)

- 하지만 정수가 아닌 다른 값을 반환하고 싶다면?

  → 스트림 인터페이스에 정의된 일반적인 연산을 사용해야 한다!


```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); // 스트림을 숫자 스트림으로
Stream<Integer> stream = intStream.boxed();  // 숫자 스트림을 스트림으로
```

- `boxed` 메서드를 이용해 특화 스트림을 일반 스트릠으로 변환할 수 있다

&nbsp;

### 기본값 : OptionalInt

숫자형 스트림에서 reduce 메서드를 사용해서 합계를 구할 때는 0이라는 기본값을 넣어줘서 딱히 문제가 없었다

하지만 IntStream에서는 0이라는 기본값 때문에 잘못된 결과가 날 수도!

- 스트림에 요소가 없는 상황과 최댓값이 0인 상황을 어떻게 구별할 수 있을까?

  → OptionalInt, OptionalDouble, OptionalLong 기본 특화형 스트림 버전 제공


```java
OptionalInt maxCalories = menu.stream()
		.mapToInt(Dish::getCalories)
		.max();
```

- OptionalInt를 이용해 최댓값을 구하는 예제
    - 이제 OptionalInt를 이용해 최댓값이 없는 상황에 사용할 기본값을 명시적으로 정의 가능!

    ```java
    int max = maxCalories.orElse(1);  // 값이 었으면 기본 최댓값 명시적으로 설정
    ```

&nbsp;

## 숫자 범위

- 특정 범위의 숫자를 이용해야 할 때?

→ 자바8의 **IntStream과 LongStream**에서는 **range**와 **rangeClosed**라는 정적 메서드 제공!

(range는 종료값 포함X, rangeClosed는 종료값 포함O)

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100)
		.filter(n -> n % 2 == 0);
System.out.println(evenNumbers.count()); 
```

- 이때 rangeClosed 대신 range를 사용하면 100을 포함하지 않으므로 짝수 49개를 반환한다
  &nbsp;    
  &nbsp;

# 5.8 스트림 만들기

## 값으로 스트림 만들기

**Stream.of**를 이용해 스트림을 만들 수 있다

- 문자열 스트림 만드는 예제

    ```java
    Stream<String> stream = Stream.of("Modern", "java", "In", "Action");
    stream.map(String::toUpperCase).forEach(System.out::println);
    
    Stream<String> emptyStream = Straem.empty(); // empty 메서드로 스트림을 비울 수 있다
    ```

&nbsp;

## null이 될 수 있는 객체로 스트림 만들기

**System.getProperty**는 제공된 키에 대응하는 속성이 없으면 null을 반환한다

```java
String homeValue = System.getProperty("home");
Stream<String> homeValueStream = 
	homeValue == null ? Stream.empty() : Stream.of(value);
```

- **Stream.ofNullable**을 이용해 아래처럼 구현 가능

```java
Stream<String> homeValueStream = 
	Stream.ofNullable(System.getProperty("home"));
```

- **flatMap**과 함께 사용하면 더 유용하게 사용 가능

```java
Stream<String> values = Stream.of("config", "home", "user")
	.flatMap(key -> Stream.ofNullable(System.getProperty(key)));
```
&nbsp;

## 배열로 스트림 만들기

**Arrays.stream**을 이용해 스트림을 만들 수 있다

```java
int[] numbers = [2, 3, 5, 7, 11, 13];
int sum = Arrays.stream(numbers).sum();
```
&nbsp;

## 파일로 스트림 만들기

Files.lines는 주어진 파일의 행 스트림을 문자열로 반환한다

- 파일에서 고유한 단어 수를 찾는 예제

```java
long uniqueWords = 0;
try(Stream<String> lines = Files.lines(Paths.get("data.txt"), CharSet.defaultCharset())) {  // 스트림은 자우너을 자동으로 해제하므로 try-finally가 필요 없다
	uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" "))) // 고유 단어 수
		.distinct()
		.count(); // 단어 스트림 생성
}
	catch(IOException e) {  // 파일 열다가 예외 발생하면 처리
}
```

- 스트림의 소스가 I/O 자원 → try/catch 블록으로 감쌈 → 메모리 누수 막으려면 자원 닫아야 함
    - 기존에는?
        - finally 블록으로 자원을 닫았다
    - 하지만 Stream 인터페이스는 AutoCloseable 인터페이스를 구현한다!

      **→ try 블록 내의 자원은 자동으로 관리된다**

&nbsp;

## 함수로 무한 스트림 만들기

두 정적 메서드 **Stream.iterate**와 **Stream.generate**로 **무한 스트림**을 만들 수 있다

- iterate와 generate에서 만든 스트림은 요청할 때마다 주어진 함수를 이용해서 값을 만든다

  ⇒ 무제한 값 계산 가능!

  (하지만 보통 무한한 값 출력하지 않도록 limit(n) 함수 연결해서 사용)

  ### iterate 메서드

    ```java
    Stream.iterate(0, n -> n + 2)
    		.limit(10)
    		.forEach(System.out::println);
    ```

    - 초깃값과 람다를 인수로 받아서 새로운 값을 끈임없이 생성 가능하다

  ( 위 예제에서는 짝수 수트림 생성)

    - 기본적으로 iterate는 기존 결과에 의존해서 순차적으로 연산을 수행한다
        - 요청할 때마다 값을 생산하며 **무한 스트림**을 만든다

      ⇒ **언바운드 스트림**

      (스트림과 컬렉션의 가장 큰 차이)

    - 연속된 일련의 값을 만들 때 iterate를 사용한다 (날짜 생성 등)

---

### [ 퀴즈 ] 피보나치 수열

```java
Stream.iterate(new int[]{0,1}, t -> new int[]{t[1], t[0]+t[1]})
		.limit(20)
		.forEach(t -> System.out.println("(" + t[0] + "," t[1] + ")"));
```

- 일반적인 피보나치 수열

```java
Stream.iterate(new int[]{0,1}, t -> new int[]{t[1], t[0]+t[1]})
		.limit(10)
		.map(t -> t[0])
		.forEach(System.out::println);
```

---

- iterate 메소드는 프레디케이트 지원!
    - 0부터 100까지 숫자를 생성하고, 100보다 크면 중단하는 예제

    ```java
    IntStream.iterate(0, n -> n < 100, n -> n + 4)
    		.forEach(System.out::println);
    ```

    - 두 번째 인수로 프레디케이트를 받아 언제까지 작업을 수행할 것인지의 기준으로 사용

    ```java
    IntStream.iterate(0, n -> n + 4)
    		.filter(n -> n < 100)
    		.forEach(System.out::println);
    ```

    - 이렇게 filter를 사용해서도 되지 않을까?.?
        - **안됨!!**

  **→ filter 메서드는 언제 이 작업을 중단해야 하는지 알 수 없다! (종료되지 않는다)**

    - 스트림 쇼트서킷을 지원하는 t**akeWhile**을 이용할 수 있다

    ```java
    IntStream.iterate(0, n -> n + 4)
    		.takeWhile(n -> n < 100)
    		.forEach(System.out::println);
    ```

&nbsp;  

### generate 메서드
iterate와 달리 생성된 각 값을 연속적으로 계산하지 않는다

- Supplier<T>를 인수로 받아 새로운 값을 생성한다

    ```java
    Stream.generate(Math:random)
            .limit(5)
            .forEach(System.out::println);
    ```

    - 만약 위 코드에서 limit가 없다면?
        
        → 스트림 **언바운드 상태**가 된다!
    
    - 위 예제에서 사용한 발행자(supplier)는 상태가 없다 (나중에 계산에 사용할 어떤 값도 저장해두지 않는다)
        - 꼭 발행자의 상태가 없어야 하는 것은 아니다!
    
        발행자가 상태를 저장한 다음에 스트림의 다음 값을 만들 때 상태를 고칠 수 있다
    
        → 병렬 코드에서는 발행자에 상태가 있으면 안전하지 않다 (7장 설명)
    
        **⇒ 상태를 갖는 발행자는 실제로는 피해야 한다**
    
    - 위 예제에서 **IntStream**을 이용하면 박싱 연산 문제 피할 수 있다!
        - IntStream의 generate 메서드는 Supplier<T> 대신 **IntSupplier**를 인수로 받는다
    
        ```java
        IntStream ones = IntStream.generate(() -> 1);
        ```
    
        - 무한 스트림 생성하는 예제
    - IntSupplier 인터페이스에 정의된 getAsInt를 구현하는 객체를 명시적으로 전달할 수 있다
    
        ```java
        IntStream twos = IntStream.generate(new IntSupplier() {
                public int getAsInt() {
                        return 2;
                }
        });
        ```
    
        - 여기서 사용한 익명 클래스와 람다는 비슷한 연산을 수행한다
            - 하지만 익명 클래스에서는 getAsInt 메서드의 연산을 커스터마이크할 수 있는 상태 필드를 정의할 수 있다 **(람다와의 차이)**
                - **람다는 상태를 바꾸지 않는다**

    - 기존의 수열 상태를 저장하고 getAsInt로 다음 요소 계산하도록 IntSupplier를 만들어야 한다 (피보나치수열 작업)
        - 다음에 호출될 때는 IntSupplier의 상태를 갱신할 수 있어야 한다
    
        ```java
        IntSupplier fib = new IntSupplier() {
                private int previous = 0;
                private int current = 1;
                public int getAsInt() {
                        int oldPrevious = this.previous;
                        int nextValue = this.previous + this.current;
                        this.previous = this.current;
                        this.current = nextValue;
                        return oldPrevious;
                }
        };
        IntStream.generate(fib).limit(10).forEach(System.out::println);
        ```
    
        - 만들어진 객체는 기존 피보나치 요소와 두 인스턴스 변수에 어떤 피보나치 요소가 들어있는지 추적하므로 **가변 상태 객체다**!
            - getAsInt를 호출하면 객체 상태가 바뀌며 새로운 값을 생성한다
        
            (iterate를 사용했을 때는 각 과정에서 새로운 값을 생성하면서도 기존 상태를 바꾸지 않는 **순수한 불변 상태를 유지**했다)
        
    
        → 스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 불변 상태 기법을 고수해야 한다


**⇒ 무한 스트림의 요소는 무한적으로 계산이 반복되므로 정렬하거나 리듀스할 수 없다!**

&nbsp;    
&nbsp;

# 5.9 마치며

- 스트림 API를 이용하면 복잡한 데이터 질의 표현 가능
    - **filter, distinct, takeWhile, dropWhile, skip, limit** 메서드로 스트림을 필터링하거나 자를 수 있다
- **map, flatMap** 메서드로 스트림의 요소를 추출/변환 가능
- 소스가 정렬되어 있다? → takeWhile과 dropWhile 메서드를 효과적으로 사용 가능
- **findFirst, findAny** 메서드로 스트림의 요소 검색 가능
    - **allMatch, noneMatch, anyMatch** 메서드를 이용해 주어진 프레디케이트와 일치하는 요소를 스트림에서 검색 가능
- **이들 메서드는 쇼트서킷 (결과 찾는 즉시 반환), 전체 스트림을 처리하지 않는다**
- **reduce** 메서드로 스트림의 모든 요소를 반복 조합해 값 도출 가능
    - 최댓값, 합계 등 계산 가능
- **filter, map** 등은 상태를 저장하지 않는 **상태 없는 연산**
    - **reduce** 같은 연산은 값을 계산하는 데 **필요한 상태 저장**
    - **sorted, distinct** 등의 메서드는 새로운 스트림을 반환하기 전 스트림의 모든 요소를 버퍼에 저장해야 한다
        - **⇒ 상태 있는 연산**
- **IntStream, DoubleStream, LongStream은 기본형 특화 스트림**
- 컬렉션 외에도 값, 배열, 파일, iterate와 generate 같은 메서드로도 스트림 생성 가능
- **무한 스트림** : 무한한 개수의 요소를 가진 스트림
