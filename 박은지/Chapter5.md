# Chapter 5) 스트림 활용

&nbsp;  

명시적 반복 대신 filter, collect 연산 지원하는 스트림 API 이용해 데이터 컬렉션 반복을 내부적으로 처리 가능하다

데이터를 어떻게 처리할 지 → 스트림 API가 관리 ⇒ 편리하게 데이터 관련 작업 가능

⇒ 스트림 API 내부적으로 다양한 최적화가 이루어질 수 있다

내부 반복 뿐 병렬 실행 여부도 결정 가능!

→ 이런 일은 (순차적인 반복을 단일 스레드로 구현하는) 외부 반복으로는 달성 불가하다

&nbsp;   
&nbsp;   

# 5.1 필터링

## 프레디케이트로 필터링

**filter** 메서드는 프레디케이트를 인수로 받아서 프레이케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다

```java
List<Dish< vegetarianMenu = menu.stream().filter(Dish::isVegetarian).collect(toList());
```
&nbsp;  
## 고유 요소 필터링

**distinct** 메서드는 고유 요소로 이루어진 스트림을 반환한다

(고유 여부는 스트림에서 만든 객체의 hashCode, equals로 결정)

```java
List<Integer> numbers = Arrays.asList(1,2,1,3,3,2,4);
numbers.stream().filter(i->i%2==0).distinct().forEach(System.out::println);
```
&nbsp;  
&nbsp;  
# 5.2 스트림 슬라이싱

## 프레디케이트를 이용한 슬라이싱

자바 9는 스트림의 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 메서드를 지원한다

&nbsp;  

### TAKEWHILE 활용

```java
List<Dish> filteredMenu = specialMenu.stream().filter(dish -> dish.getCalories() < 320).collect(toList());
```

- 칼로리 순으로 정렬되어있는 예제

→ filter 연산 이용하면 전체 스트림을 반복하면서 각 요소에 프레디케이트를 적용

→ 리스트가 이미 정렬되어 있다는 사실을 이용해 조건에 따라 반복 작업을 중단할 수 있다

(작은 리스트에서는 별 차이 없어 보여도 많은 요소를 포함하는 큰 스트림에서는 상당한 차이가 될 수 있다)

**takeWhile 활용하면 스트림을 포함한 모든 스트림에 프레디케이트를 적용해 스트림 슬라이스가 가능하다**

&nbsp;  
### DROPWHILE 활용

나머지 요소를 선택하는 경우에는 **dropWhile**을 사용하여 작업을 완료할 수 있다

(dropWhile은 takeWhile과 정반대의 작업을 수행)

```java
List<Dish> sliceMenu2 = specialMenu.stream().dropWhile(dish -> dish.getCalories() < 320).collect(toList());
```

→ 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버린다!

(거짓이 되면 그 지점에서 작업 중단하고 남은 모든 요소 반환)

**⇒ 무한한 많은 요소를 가진 무한 스트림에서도 동작**

&nbsp;  

## 스트림 축소

**limit(n)** 메서드는 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 limit(n) 메서드를 지원한다

스트림이 정렬되어 있으면 최대 요소 n개를 반환 가능하다

정렬되지 않은 스트림 (Set)에도 limit 사용 가능 → 정렬되지 않은 상태로 반환

&nbsp;  

## 요소 건너뛰기

**skip(n)** 메서드는 처음 n개 요소를 제외한 스트림을 반환한다

limit(n)과 skip(n)은 상호 보완적인 연산을 수행한다

&nbsp;  
&nbsp;  

# 5.3 매핑

특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행되는 연산

→ **map**과 **flatMap** 메서드는 특정 데이터를 선택하는 기능을 제공한다

```java
List<Dish> dishes = menu.stream().filter(d -> d.getCalories() > 300).skip(2).collect(toList());
```

&nbsp;  

## 스트림의 각 요소에 함수 적용하기

**map** 메서드는 함수를 인수로 받는다

인수로 제공된 함수는 각 요소에 적용됨 + 함수를 적용한 결과가 새로운 요소로 매핑됨

```java
List<String> dishNames = menu.stream().map(Dish::getName).collect(toList());
```

- 단어 리스트가 주어졌을 때 각 단어가 포함하는 글자 수의 리스트를 반환한다고 가정했을 때
    - 리스트의 각 요소에 함수를 적용해야 함
    - map 이용
        - 각 요소에 적용할 함수는 단어를 인수로 받아서 길이를 반환해야 한다
        - 메서드 참조 String::length를 map에 전달해서 문제를 해결할 수 있다
- 각 요리명의 길이를 알고 싶다면?
    - 다른 map 메서드를 연결
    
    ```java
    List<Integer> dishNameLengths = menu.stream().map(Dish::getName).map(String::length).collect(toList());
    ```
    

&nbsp;  

## 스트림 평면화

- 리스트에서 고유 문자로 이루어진 리스트를 반환하는 상황
    - [”Hello”, “World”] 리스트가 있다면 [”H”, “e”, “l”, “o”, “W”, “r”, d”]를 포함하는 리스트가 반환되어야 함
    
    ```java
    words.stream().map(word -> word.split("").distinct().collect(toList());
    ```
    
    위처럼 해결할 수 있다고 생각할 수 있지만 map 메서드는 Stream<String[]>을 반환한다
    
    (우리가 원하는 건 문자열의 스트림을 표현할 Stream<String>이다)
    
    → 이 문제를 **flatMap** 메서드를 통해 해결할 수 있다!
    

&nbsp;  

## map과 Arrays.stream 활용

```java
words.stream().map(word -> word.split("")).map(Arrays::stream).distinct().collect(toList());
```

배열 스트림 대신 문자열 스트림이 필요해서 Arrays.stream() 메서드를 사용했지만,

결국 스트림 리스트 (List<Stream<String>>)가 만들어지면서 문제가 해결되지 않았다

→ 문제를 해결하려면 먼저 각 단어를 개별 문자열로 이루어진 배열로 만든 다음에 각 배열을 별도의 스트림으로 만들어야 한다

&nbsp;  

## flatMap 사용

```java
List<String> uniqueCharacters = words.stream().map(word -> word.split("")).flatMap(Arrays::stream).distinct().collect(toList());
```

flatMap은 각 배열을 스트림이 아닌 스트림의 콘텐츠로 매핑한다

**→ 즉, map(Arrays::stream)과 달리 하나의 평면화된 스트림을 반환한다**

⇒ flapMap 메서드는 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다

&nbsp;  

---

## 퀴즈

- 두 개의 숫자 리스트가 있을 때 모든 숫자 쌍의 리스트를 반환하시오
    - [1,2,3]과 [3,4]가 주어진다면 [(1,3),(1,4),(2,3),(2,4),(3,3),(3,4)]를 반환해야 한다
        
        ```java
        List<Integer> list1 = Arrays.asList(1,2,3);
        List<Integer> list2 = Arrays.asList(3,4);
        List<int[]> pairs = list1.stream().flatMap(i -> list1.stream().map(j -> new int[]{i, j})).collect(toList());
        ```
        
    - 두 개의 map을 이용해서 리스트를 반복한 다음에 숫자 쌍을 만들 수 있다
        - 결과로 Stream<Stream<Integer[]>>가 반환되어버림!
        - 결과를 Stream<Integer[]>로 평면화한 스트림이 필요
            - flatMap 사용해 해결 가능
        
- 합이 3으로 나누어떨어지는 쌍만 반환하려면?
    - (2,4), (3,3)을 반환해야 한다
    
    ```java
    List<Integer> list1 = Arrays.asList(1,2,3);
    List<Integer> list2 = Arrays.asList(3,4);
    List<int[]> pairs = list1.stream().flatMap(i -> list2.stream().filter(j -> (i+j) % 3 == 0).map(j -> new int[]{i, j})).collect(toList());
    ```
    

---

&nbsp;  
&nbsp;  

# 5.4 검색과 매칭

특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용된다

(allMatch, anyMatch, noneMatch, findFirst, findAny 등 다양한 유틸리티 메서드 제공)

&nbsp;  

### 프레디케이트가 적어도 한 요소가 일치하는지 확인

- **anyMatch 메서드**

```java
if(menu.stream().anyMatch(Dish::isVegetarian)) {
	System.out.println("The menu is (somewhat) vegetarian friendly!!");
}
```

&nbsp;  

### 프레디케이트가 모든 요소와 일치하는지 검사

- **allMatch 메서드**

```java
boolean isHealty = menu.stream().allMatch(dish -> dish.getCalories() < 1000);
```

&nbsp;  

### NONEMATCH

- allMatch 메서드와 반대 연산 수행
    - 주어진 프레디케이트와 일치하는 요소가 없는지 확인

```java
boolean isHealty = menu.stream().allMatch(dish -> dish.getCalories() < 1000);
```

(allMatch 메서드 예제를 위처럼 표현 가능)

**anyMatch, allMatch, noneMatch 세 메서드는 스트림 쇼트서킷 기법, 즉 자바의 &&, ||와 같은 연산을 활용한다**

> 쇼트서킷 : 표현식에서 하나라도 거짓이라는 결과가 나오면 나머지 표현식의 결과와 상관없이 전체 결과도 거짓이 된다
> 

&nbsp;  

### 요소 검색

- **findAny 메서드**
    - 현재 스트림에서 임의의 요소를 반환한다
    - 다른 스트림 연산과 연결해서 사용 가능
        - filter와 findAny를 이용해서 채식요리를 선택할 수 있다
        
        ```java
        Optional<Dish> dish = menu.stream().filter(Dish::isVegetarian).findAny();
        ```
        

스트림 파이프라인은 내부적으로 단일 과정으로 실행할 수 있도록 최적화된다

→ 쇼트서킷을 이용해서 결과를 찾는 즉시 실행 종료

&nbsp;  

### Optional?

- Optional<T> 클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스이다
    - 위 findAny 메서드는 아무 요소도 반환하지 않을 수 있다
    
    → null은 쉽게 에러 발생시킬 수 있음
    
    → 이를 방지하기 위해 Optional 등장
    
    **⇒ 값이 존재하는지 확인하고 값이 없을 때 어떻게 처리할지 강제하는 기능**
    
    ```markdown
    - isPresent() : Optional이 값을 포함하면 true, 포함하지 않으면 false 반환
    - isPresent(Consumer<T> block) : 값이 있으면 주어진 블록 실행
        - Consumer 함수형 인터페이스에는 T 형식의 인수를 받으면 void를 반환하는 람다 전달 가능
    - T get() : 값이 존재하면 값을 반환, 없으면 NoSuchElementException 발생
    - T orElse(T other) : 값이 있으면 값을 반환, 없으면 기본값 반환
    ```
    
&nbsp;  

### 첫 번째 요소 찾기

- 리스트/정렬된 연속 데이터로부터 생성된 스트림처럼 일부 스트림에는 **논리적인 아이템 순서**가 정해져 있을 수도 있다
    - 이런 스트림에서 첫 번째 요소를 찾으려면?
        
        ```java
        List<Integer> someNumbers = Arrays.asList(1,2,3,4,5);
        Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream().map(n -> n * n).filter(n -> n % 3 == 0).findFirst();
        ```
        
    
    ### findFirstd와 findAny 사용?
    
    병렬성 때문!!
    
    → 병렬 실행에서는 첫 번째 요소 찾기 어렵다
    
    ⇒ 따라서 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다
    
    &nbsp;  
    &nbsp;  

# 5.5 리듀싱

스트림 요소를 조합해서 더 복잡한 질의를 표현할 수 있다

(’메뉴의 모든 칼로리 합을 구하시오’, ‘메뉴 중 가장 높은 칼로리는?’)

→ Integer 같은 결과가 나올 때까지 스트림의 모든 요소를 반복적으로 처리해야 한다

**⇒ 리듀싱 연산** 

(함수형 프로그래밍 언어로는 폴드)

&nbsp;  

### SUM

- for-each문을 사용해 리스트의 숫자 요소를 더하는 예제

```java
int sum = 0;
for(int x : numbers) {
		sum += x;
}
```

sum 변수의 초깃값 0과 리스트의 모든 요소를 조합하는 연산(+)을 파라미터로 사용해 reduce 연산으로 다시 위 예제를 작성해보자

```java
int sum = numbers.stream().reduce(0, (a,b) -> a+b);
```

- 함수형 인터페이스 BinaryOperator<T>를 람다 표현식으로 사용

- 숫자 요소를 곱하는 예제를 만들려면 초깃값과 연산자만 바꿔주면 된다

```java
int mul = numbers.stream().reduce(1, (a,b) -> a*b);
```

- 자바 8에서는 Integer 클래스에 sum 메서드를 제공한다 (메서드 참조 이용)

```java
int sum = numbers.stream().reduce(0, Integer::sum);
```

- Optional로 감싸진 초깃값을 받지 않는 reduce도 존재한다

```java
Optional<Integer> sum = numbers.stream().reduce((a,b) -> a+b));
```

→ 초깃값을 받지 않으면 reduce는 합계를 반환할 수 없기 때문에 Optional!

&nbsp;  

### MAX, MIN

**초깃값**과 **스트림의 두 요소를 합쳐서 하나의 값으로 만드는 데 사용할 람다**를 파라미터로 받아 최댓값과 최솟값을 구할 수 있다

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
```

```java
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

&nbsp;  

### COUNT

- map과 reduce 메서드를 이용해서 스트림의 요리 개수 계산하는 예제

```java
int count = 
		menu.stream()
				.map(d -> 1)
						.reduce(0, (a,b) -> a+b);
```

```java
long count = menu.stream().count();
```

### reduce 메서드의 장점과 병렬화

단계적 반복으로 합계를 구하는 것과 reduce를 이용해서 합계를 구하는 것은 어떤 차이가 있을까?

- 반복적인 합계에서는?
    - sum 변수 공유해야 함 → 병렬화 어려움
        - 강제적 동기화 시켜도 이득보다 스레드 소모적 경쟁 때문에 가성비 X
- reduce를 이용하면
    - 내부 반복이 추상화된다
        - 내부 구현에서 병렬로 reduce를 실행할 수 있게 된다

→ stream()을 parallelStream()으로 바꿔서 병렬로 만들기

```java
int sum = numbers.parallelStream().reduce(0, Integer::sum);

```

- 단, 병렬로 실행하려면 대가를 지불해야 한다
    - reduce에 넘겨준 람다의 상태가 바뀌지 말아야 하며, 연산이 어떤 순서로 실행되더라도 결과가 바뀌지 않는 구조여야 한다

&nbsp;  
### 스트림 연산 : 상태의 유무

스트림 연산은 다양한 연산을 수행하지만 각각 연산은 내부적인 상태를 고려해야 한다

- map, filter 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다

(사용자가 제공한 람다나 메서드 참조가 내부적인 가변 상태를 갖지 않는다는 가정 하에)

→ 보통 상태가 없는, **내부 상태를 갖지 않는 연산**

- reduce, max, sum 같은 연산은 결과를 누적할 내부 상태가 필요하다
    - 스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 한정되어 있다
- sorted나 distinct 같은 연산은 filter나 map처럼 스트림을 입력으로 받아 다른 스트림을 출력하는 것처럼 보일 수 있다(아님! 다름)
    - 스트림의 요소를 정렬하거나 중복을 제거하려면 **과거의 이력을 알고 있어야 한다**
        - **어떤 요소를 출력 스트림으로 추가하려면 모든 요소가 버퍼에 추가되어 있어야 한다**
        - 연산을 추가하는 데 필요한 저장소 크기는 정해져있지 X
            
            → 데이터 스트림의 크기가 크거나 무한이라면 문제가 생길 수도 있다
            
        
        ⇒ 이런 상태의 연산을 **내부 상태를 갖는 연산**
