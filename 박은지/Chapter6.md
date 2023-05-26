# Chapter 6) 스트림으로 데이터 수집

&nbsp;  

스트림 연산은 중간 연산과 최종 연산으로 구분할 수 있다

- 중간 연산 → 한 스트림을 다른 스트림으로 변환 (여러 연산 연결 가능)
- 최종 연산 → 스트림의 요소를 소비해서 최종 결과 도출 (파이프라인 최적화하면서 계산 과정 생략)

collect 를 사요해 다양한 요소 누적 방식을 인수로 받아서 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행할 수 있다

- Collection, Collector, collect 헷갈리지 않게 주의!

&nbsp;   

### 통화별로 트랜잭션을 그룹화하는 예제

- 명령형 버전

```java
Map<Currency, List<Transaction>> transactionByCurrencies = new HashMap<>();
for(Transaction transaction : transactions) {
		Currency currency = transaction.getCurrency();
		List<Transaction> transactionsForCurrency = transactionByCurrencies.get(currency);
		if(transactionsForCurrency == null) {
				transactionsForCurrency = new ArrayList<>();
				transactionsByCurrencies.put(currency, transactionsForCurrency);
		}
		transactionsForCurrency.add(transaction);
}
```

→ 코드가 무엇을 실행하는지 한눈에 파악하기 어렵다

- 함수형 버전

```java
Map<Currency, List<Transaction>> transactionsByCurrencies = 
		transactions.stream().collect(groupingBy(Transaction::getCurrency));
```

&nbsp;   
&nbsp;   

# 6.1 컬렉터란 무엇인가?

Collector 인터페이스 구현은 **스트림의 요소를 어떤 식으로 도출할지 지정!**

(함수형 프로그래밍에서는 ‘무엇’을 원하는지 직접 명시할 수 있어서 어떤 방법으로 이를 얻을지는 신경 쓸 필요가 없다)

- 다수준으로 그룹화를 수행할 때 명령형 프로그래밍과 함수형 프로그래밍의 차이점이 더욱 잘 보인다
    - 명령형
        - 문제를 해결하는 과정에서 다중 루프, 조건문이 추가되며 가독성과 유지보수성이 크게 떨어진다
    - 함수형
        - 필요한 컬렉터를 쉽게 추가할 수 있다

&nbsp;  

## 고급 리듀싱 기능을 수행하는 컬렉터

collect의 최대 강점 ?.?

**→ 결과를 수집하는 과정을 간단 + 유연한 방식으로 정의할 수 있다**

- 스트림에 collect를 호출하면 스트림의 요소에 리듀싱 연산이 수행된다
    
    (리듀싱 연산을 이용해 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리)
    

보통 함수를 요소로 변환할 때?

→ 컬렉터를 적용하며 최종 결과를 저장하는 자료구조에 값을 누적한다

(ex. 각 트랜잭션에서 통화를 추출한 다음 통화를 키로 사용해서 트랜잭션 자체를 결과 맵에 누적)

**⇒ Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할지 결정!**

- Collectors 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 정적 팩토리 메서드를 제공한다
    - 예를 들어, 스트림의 모든 요소를 리스트로 수집하는 toList()

&nbsp;  

## 미리 정의된 컬렉터

Collectors에서 제공하는 메서드의 기능

- **스트림 요소를 하나의 값으로 리듀스하고 요약**
    - 트랜잭션 리스트에서 트랜잭션 총합을 찾는 등의 다양한 계산 수행
- **요소 그룹화**
    - 다수준으로 그룹화하거나 각각의 결과 서브그룹에 추가로 리듀싱 연산을 적용할 수 있도록 다양한 컬렉터를 조합
- **요소 분할**
    - 한 개의 인수를 받아 불리언을 반환하는 함수(프레디케이트)를 그룹화 함수로 사용

&nbsp;    
&nbsp;    

# 6.2 리듀싱과 요약

컬렉터로 스트림의 항목을 컬렉션으로 재구성할 수 있다

→ 컬렉터로 스트림의 모든 항목을 하나의 결과로 합칠 수 있다!

- 트리르르 구성하는 다수준 맵, 메뉴의 칼로리 합계를 가리키는 단순한 정수 등 다양한 형식으로 결과 도출 가능

&nbsp;  

### counting()

```java
long howManyDishes = menu.stream().collect(Collectors.counting());
```

→ 불필요한 과정 생략 가능!

```java
long howManyDishes = menu.stream().count();
```

- counting 컬렉터는 다른 컬렉터와 함께 사용할 때 더 굿

&nbsp;  

## 스트림값에서 최댓값과 최솟값 검색

**Collectors.maxBy, Collectors.minBy**

- 두 컬렉터는 스트림의 요소를 비교하는 데 사용할 Comparator를 인수로 받는다

```java
Comparator<Dish> dishCaloriesComparator = 
		Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = 
		menu.stream()
				.collect(maxBy(dishCaloriesComparator));
```

&nbsp;  

## 요약 연산

**Collectors.summingInt, Collectors.summingDouble**

→ 특별한 요약 팩토리 메서드!

- 객체를 int로 매핑하는 함수를 인수로 받는다
    - summingInt의 인수로 전달된 함수는 객체를 int로 매핑한 컬렉터를 반환
    - → summingInt가 collect 메서드로 전달되면 요약 작업 수행
        - 메뉴 리스트의 총 칼로리를 계산하는 예제
            
            ```java
            int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
            ```
            

**Collectors.averagingInt, averagingDouble, averagingLong**

- 평균값 계산
    
    ```java
    double avgCalories = menu.stream().collect(averagingInt(Dish::getCalories));	
    ```
    
- 개수, 최댓값/최솟값, 합계, 평균 이런 계산들 중 두 개 이상의 연산을 한 번에 수행해야 한다면?
    
    **→ summarizingInt가 반환하는 컬렉터 사용!**
    
    - 요소 수, 요리의 칼로리 합계, 평균, 최댓값, 최솟값 등을 계산하는 예제
    
    ```java
    IntSummaryStatistics menuStatistics = 
    		menu.stream().collect(summarizingInt(Dish::getCalories));
    ```
    
    → 아래와 같은 정보 확인 가능
    
    ```java
    IntSummaryStatistics{count=9, sum=4300, min=120,
    		average=477.777778, max=800}
    ```
    
    (summarizingLong, summarizingDouble과 관련된 LongSummaryStatistics, DoubleSummaryStatistics 클래스도 존재)
    

&nbsp;  

## 문자열 연결

**joining** 팩토리 메서드

→ 이용하면 스트림의 각 객체에 toString 메서드를 호출해서 추출한 모든 문자열을 하나의 문자열로 연결해서 반환

- 메뉴의 모든 요리명을 연결하는 예제
    
    ```java
    String shortMenu = menu.stream().map(Dish::getName).collect(joining());
    ```
    
- joining 메서드는 내부적으로 StringBuilder를 이용해 문자열을 하나로 만든다
    - Dish 클래스가 요리명을 반환하는 toString 메서드를 포함하고 있다면 아래처럼 작성 가능
    
    ```java
    String shortMenu = menu.stream().collect(joining());
    ```
    
    → 아래와 같은 결과 확인 가능
    
    ```java
    porkbeekchickenfrench friesriceseason fruitpizzaprawnssalmon
    ```
    
    - 하지만 결과 문자열 해석 불가!
    
    → 연결된 요소들 사이에 구분 문자열 넣을 수 있지롱
    
    **⇒ 오버로드된 joining 팩토리 메서드!**
    
    ```java
    String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
    ```
    
    → 아래와 같은 결과 확인 가능
    
    ```java
    pork, beek, chicken, french fries, rice, season fruit, pizza, prawns, salmon
    ```
    
    &nbsp;  
    

## 범용 리듀싱 요약 연산

**Collectors.reducing**

- 위에서 설명한 모든 컬렉터를 reducing 팩토리 메서드로 정의할 수 있다
    - 그럼에도 특화된 컬렉터를 사용한 이유는?
    
    **→ 프로그래밍적 편의성** 때문!
    
    - 메뉴의 모든 칼로리 합계를 계산하는 예제
        
        ```java
        int totalCalories = menu.stream().collect(reducing(
        		0, Dish::getCalories, (i, j) -> i+j));
        ```
        
        (세 개의 인수를 받는다)
        
        - 첫 번째 인수
            - 리듀싱 연산의 **시작값이거나 스트림에 인수가 없을 때는 반환값**
        - 두 번째 인수
            - **변환 함수**
        - 세 번째 인수
            - 같은 종류의 두 항목을 하나의 값으로 더하는 **BinaryOperator**
            
            (위 예제에서는 두 개의 int가 사용됨)
            
- **한 개의 인수를 갖는 reducing 팩토리 메서드 사용 가능**
    - 세 개의 인수를 갖는 reducing 메서드에서
        - 첫 번째 인수
            - 시작 요소
        - 두 번째 요소
            - 자신을 그대로 반환하는 **항등 함수**
    - **만약 한 개의 인수를 갖는 reducing 컬렉터는 시작값이 없다면?**
        - 빈 스트림이 넘겨졌을 때 시작값이 설정되지 않는 상황이 발생
        
        **⇒ 한 개의 인수를 갖는 reducing은 Optional<Dish> 객체를 반환!**
        
    - 가장 칼로리가 높은 요리를 찾는 예제
        
        ```java
        Optional<Dish> mostCaloriesDish = 
        		menu.stream().collect(reducing(
        				(d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
        ```
        

---

## collect와 reduce

- **collect와 reduce가 뭐가 다른데?.?**
- 만약 toList 컬렉터를 사용하는 collect 대신 reduce 메서드로 사용해 작성한다면?
    
    ```java
    Stream<Integer> stream = Arrays.asList(1,2,3,4,5,6).stream();
    List<Integer> numbers = stream.reduce(
    		new ArrayList<Integer>(),
    		(List<Integer> l, Integer e) -> {
    				l.add(e);
    				return l; },
    		(List<Integer> l1, List<Integer> l2) -> {
    				l1.addAll(l2);
    				return l1; });
    ```
    
    → 의미론적인 문제와 실용성 문제 발생!
    
    - collect 메서드는
        - **도출하려는 결과를 누적**하는 컨테이너를 바꾸도록 설계된 메서드
    - reduce는
        - **두 값을 하나로 도출**하는 **불변형 연산**
    
    이라는 점에서 의미론적인 문제 발생
    
    → 위 에제에서 reduce 메서드는 누적자로 사용된 리스트를 변환시키므로 reduce를 잘못 활용한 예에 해당
    
    ⇒ 의미론적으로 reduce 메서드를 잘못 사용하면서 실용성 문제도 발생!
    
    (여러 스레드가 동시에 같은 데이터 구조체를 고치면 리스트 자체가 망가져버리므로 리듀싱 연산을 병렬로 수행할 수 없다는 점도 문제)
    
    - 이 문제를 해결하려면?
        - 매번 새로운 리스트를 할당해야 함
        
        → 성능 저하
        
        ⇒ 가변 컨테이너 관련 작업ㄷ이면서 병렬성을 확보하려면 collect 메서드로 리듀싱 연산을 구현해야 함
        

---

### 컬렉션 프레임워크 유연성 : 같은 연산도 다양한 방식으로 수행할 수 있다

```java
int totalCalories = menu.stream().collect(reducing(
		0, Dish::getCalories, (i, j) -> i+j));
```

위 reducing 컬렉터를 사용한 예제에서 Integer 클래스의 sum 메서드를 사용해 좀 더 단순화할 수 있다

```java
int totalCalories = menu.stream().collect(reducing(
		0, Dish::getCalories, Integer::sum));
```

- 누적자를 초깃값으로 초기화 → 합계 함수를 이용해 각 요소에 변환 함수를 적용한 결과 숫자를 반복적으로 조합

- **counting** 컬렉터도 (세 개의 인수를 갖는) reducing 팩토리 메서드를 이용해 구현 가능

```java
public static <T> Collector<T, ?, Long> counting() {
		return reducing(0L, e -> 1L, Long::sum);
}
```

### 자신의 상황에 맞는 최적의 해법 선택

컬렉터를 이용하면?

- 코드가 더 복잡해진다
    - 대신 재사용성, 커스터마이즈 가능성 제공하는 높은 수준의 추상화/일반화를 얻을 수 있다!

→ 문제마다 특화된 해결책 골라서 적용하는게 가장 바람직하다

# 6.3 그룹화

- 명령형으로 그룹화를 구현하려면?

→ 번거로움

- 함수형 이용해 가독성 좋게 구현 가능!
    - Collectors.groupingBy 이용한 메뉴를 그룹화하는 예제
    
    ```java
    Map<Dish.Type, List<Dish>> dishesByType = 
    		menu.stream().collect(groupingBy(Dish::getType));
    ```
    
    - 아래와 같은 결과 확인 가능
    
    ```java
    {FISH=[prawns, salmon], OTHER=[french fries, rice, season fruit, pizze],
    MEAT=[pork, beef, chicken]}
    ```
    
    - 각 요리에서 Dish.Type이 일치하는 요리 추출하는 함수를 groupingBy 메서드로 전달
        - 이 함수를 기준으로 스트림이 그룹화 된다
        
        **⇒ 분류 함수**
        

## 그룹화된 요소 조작

요소를 그룹화 한 다음에는 각 결과 그룹의 요소를 조작하는 연산이 필요하다

- 500 칼로리 이상의 요리만 필터링 할 때
    
    ```java
    Map<Dish.Type, List<Dish>> caloricDishesByType = 
    		menu.stream().filter(dish -> dish.getCalories() > 500)
    				.collect(groupingBy(Dish::getType));
    ```
    
    - 이렇게 프레티케이트로 필터를 적용해 해결할 수 있다고 생각할 수도 있다
    
    → 해결 가능하긴 하지만 단점 존재!
    
    ```java
    {OTHER=[french fries, pizze], MEAT=[pork, beek]}
    ```
    
    - 메뉴 요리는 맵 형태로 되어 있기 때문에 위 기능 사용하려면 맵에 코드를 적용해야 한다
    
    → 프레디케이트를 만족하는 FISH 종류 없어서 결과 맵에서 해당 키 자체가 사라지는 문제 발생
    
    - Collectors 클래스는 일반적인 분류 함수에 Collector 형식의 두 번째 인수를 갖도록 groupingBy 팩토리 메서드를 오버로드해 문제 해결
    
    (⇒ 두 번째 Collector 안에 필터를 넣음)
    
    ```java
    Map<Dish.Type, List<Dish>> caloricDishesByType = 
    		menu.stream()
    				.collect(groupingBy(Dish::getType,
    						filtering(dish -> dish.getCalories() > 500, toList())));
    ```
    
    - **filtering** 메서드는 프레디케이트를 인수로 받는다
        - 이 프레디케이트로 각 그룹의 요소와 필터링 된 요소를 재그룹화
        
        ```java
        {OTHER=[french fries, pizze], MEAT=[pork, beek], FISH=[]}
        ```
        
    - 그룹화된 항목을 조작하는 또 다른 방법?
    
    → 매핑 함수를 이용해 요소를 변환!
    
    - Collectors 클래스는 **mapping** 메서드도 제공
        - 매핑 함수와 각 항목에 적용한 함수를 모으는 데 사용
        
        ```java
        Map<Dish.Type, List<Dish>> dishNamesByType = 
        		menu.stream()
        				.collect(groupingBy(Dish::getType, mapping(Dish::getName, toList())));
        ```
        
        - groupingBy와 연계해 세 번째 컬렉터를 사용해 flapMap 변환 수행 가능
            - 태그 목록을 가진 각 요리로 구성된 맵이 있는 상황이라면
            
            ```java
            Map<Dish.Type, Set<String>> dishNamesByType = 
            		menu.stream()
            				.collect(groupingBy(Dish::getType,
            						filtering(dish -> dishTags.get(dish.getName()).stream(), toSet());
            ```
            
            - 아래와 같은 결과 확인 가능
            
    

## 다수준 그룹화

Collectors.groupingBy 팩토리 메서드를 이용해 항목을 다수준으로 그룹화 할 수 있다

- 일반적인 분류 함수와 컬렉터를 인수로 받는다

→ groupingBy 메서드에 스트림의 항목을 분류할 두 번째 기준을 정의하는 내부 groupingBy를 전달해 두 수준으로 스트림의 항목을 그룹화할 수 있다

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = 
menu.stream().collect(
		groupingBy(Dish::getType,
				groupingBy(dish -> {
						if(dish.getCalories() <= 400)
								return CaloricLevel.DIET;
						else if(dish.getCalories() <= 700)
								return CaloricLevel.NORMAL;
						else 
								return CaloricLevel.FAT;
				})
		)
);
```

- 아래와 같은 결과 확인 가능

```java
{MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]}, FISH={DIET=[prawns],
NORMAL=salmon]},
OTHER={DIET=[rice, seasonal fruit], NORMAL=[french fries, pizza]}}
```
