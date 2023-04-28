# Chapter 3) 람다 표현식 - 2


# 메서드 참조

→ 기존의 메서드 정의를 재활용해서 람다처럼 전달 가능

상황에 따라 람다 표현식보다 메서드 참조가 더 가독성이 좋고 자연스러울 수도 있다

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

```java
inventory.sort(comparing(Apple::getWeight));
```
&nbsp;  

### 메서드 참조 ⇒ 특정 메서드만을 호출하는 람다의 축약형

- 메서드를 어떻게 호출해야 하는지 설명을 참조하기 보다는 메서드명을 직접 참조하는 것이 편리하다
- 실제로 메서드 참조를 이용하면 기존 메서드 구현을 람다 표현식으로 만들 수 있다
    - 가독성이 높아진다
- 새로운 기능 X. 하나의 메서드를 참조하는 람다를 편리하게 표현할 수 있는 문법이다
    - 같은 기능을 더 간결하게 구현 가능

&nbsp;

## 메서드 참조 만드는 방법

### 1. 정적 메서드 참조

- Integer의 parseInt 메서드 → Integer::parseInt

### 2. 다양한 형식의 인스턴스 메서드 참조

- String의 length 메서드 → String::length
- (String s) → s.toUpperCase() 람다표현식을 String::toUpperCase

### 3. 기존 객체의 인스턴스 메서드 참조

- Transaction 객체를 할당받은 expensiveTransaction 지역 변수와 Transaction 객체에는 getValue 메서드가 있다면 → expensiveTransaction::getValue
- 아래와 같은 비공개 헬퍼 메서드를 정의했을 때 유용하게 활용 가능

    ```java
    private boolean isValidName(String string) {
    		return Character.isUpperCase(string.charAt(0));
    // 메서드 참조를 사용하면
    
    filter(words, this::isValidName)
    ```

&nbsp;

생성자, 배열 생성자, super 호출 등에 사용할 수 있는 특별한 형식의 메서드 참조도 있다

- List에 포함된 문자열을 대소문자를 구분하지 않고 정렬하는 예제

```java
List<String> str = Arrays.asList("a","b","A","B");
str.sort((s1, s2) -> s1.compareToIgnoreCase(s2));
// 메서드 참조를 사용하면

List<String> str = Arrays.asList("a","b","A","B");
str.sort(String::compareToIgnoreCase);

```

컴파일러는 메서드 참조가 주어진 함수형 인터페이스와 호환하는지 확인한다

(람다 표현식의 형식을 검사하던 방식과 비슷한 과정)

**⇒ 메서드 참조는 콘텍스트의 형식과 일치해야 한다**

&nbsp;

## 문제

```java
// 문제
1. ToIntFunction<String> stringToInt = (String s) -> Integer.parseInt(s);
2. BiPredicate<List<String>, String> contains = (list, element) -> list.contains(element);
3. Predicate<String> startsWithNumber = (String string) -> this.startsWithNumber(string);

// 풀이
1. Funtion<String, Integer> stringToInteger = Integer::parseInt;
2. BiPredicate<List<String>, String> contains = List::contains;
3. Predicate<String> startsWithNumber = this::startsWithNumber
```
&nbsp;  
&nbsp;

# 생성자 참조

→ ClassName::new 처럼 클래스명과 new 키워드를 이용해 기존 생성자의 참조를 만들 수 있다

(정적 메서드의 참조를 만드는 방법과 비슷)

&nbsp;

### Supplier

```java
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();
// 아래 코드와 같다
Supplier<Apple> c1 = () -> new Apple();  // 람다 표현식은 디폴트 생성자를 가진 Apple을 만든다
Apple a1 = c1.get();
```
&nbsp;
### Function

```java
Function<Integer, Apple> c2 = Apple::new;
Apple a2 = c2.apply(110);
// 아래 코드와 같다
Function<Integer, Apple> c2 = (weight) -> new Apple(weight);
Apple a2 = c2.apply(110);ㅠㅑ
```
&nbsp;
### BiFunction

```java
BiFunction<Color, Integer, Apple> c3 = Apple::new;
Apple a3 = c3.apply(GREEN, 110);
// 아래 코드와 같다
BiFunction<Color, Integer, Apple> c3 = (color, weight) -> new Apple(color, weight);
Apple a3 = c3.apply(GREEN, 110);
```
&nbsp;
### 인스턴스화 하지 않고 생성자에 접근

- Map으로 생성자와 문자열값을 관련시킬 수 있다
- String과 Integer가 주어졌을 때 다양한 무게를 갖는 여러 종류의 과일을 만드는 메서드를 만들 수 있다

```java
static Map<String, Function<Integer, Fruit>> map = new HashMap<>();
static{
	map.put("apple", Apple::new);
	map.put("orange", Orange::new);
	// . . .
}

public static Fruit giveMeFruit(String fruit, Integer weight){
	return map.get(fruit.toLowerCase()).apply(weight);
} 
```

&nbsp;  
&nbsp;


# 람다, 메서드 참조 활용하기

### 1단계 : 코드 전달

- 자바 8의 List API에서 sort 메서드를 제공하므로 정렬 메서드를 직접 구현할 필요가 없다
    - sort 메서드의 시그니처
        - void sort(Comparator<? super E> c)

      → 객체 안에 동작을 포함시키는 방식으로 다양한 전략을 전달할 수 있다

      **⇒ sort의 동작은 파라미터화되었다**

      (= sort에 전달된 정렬 전략에 따라 sort의 동작이 달라질 것이다)


```java
public class AppleComparator implements Comparator<Apple> {
		public int compare(Apple a1, Apple a2) {
				return a1.getWeight().compareTo(a2.getWeight());
		}
}
inventory.sort(new AppleComparator());
```

&nbsp;

### 2단계 : 익명 클래스 사용

- 한 번만 사용할 경우 익명 클래스를 이용하는 것이 좋다

```java
inventory.sort(new Comparator<Apple>() {
		public int compare(Apple a1, Apple a2) {
				return a1.getWeight().compareTo(a2.getWeight());
		}
});
```

&nbsp;

### 3단계 : 람다 표현식 사용

- 함수형 인터페이스를 기대하는 곳 어디든 람다 표현식을 사용할 수 있다
    - 추상 메서드의 시그니처(함수 디스크립터)는 람다 표현식의 시그니처를 정의한다

```java
Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());
// 아래처럼 간소화 가능
import static java.util.Comparator.comparing;
inventory.sort(comparing(apple -> apple.getWeight()));
```

&nbsp;

### 4단계 : 메서드 참조 사용

- 메서드 참조를 통해 코드를 더 간소화할 수 있다

```java
inventory.sort(comparing(Apple::getweight));
```

- 코드의 길이 뿐만 아니라 코드의 의미도 명확해졌다
    - Apple을 weight별로 비교해서 inventory를 sort하라

&nbsp;  
&nbsp;

# 람다 표현식을 조합할 수 있는 유용한 메서드

- 자바 8 API의 몇몇 함수형 인터페이스는 다양한 유틸리티 메서드를 포함한다
    - Comparator, Function, Predicate 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있도록 유틸리티 메서드를 제공한다
    - ⇒ 여러 개의 람다 표현식을 조합해서 복잡한 람다 표현식을 만들 수 있다

**도대체 어떤 메서드를 제공하길래 ?.?**

- **디폴트 메서드 : 추상 메서드가 아니므로 함수형 인터페이스의 정의를 벗어나지 않는다**

&nbsp;
## Comparator 조합

```java
Comparator<Apple> c = Comparator.comparing(Apple::getWeight);
```

Comparator.comparing을 이용해 비교에 사용할 키 추출

&nbsp;
### 역정렬

사과의 무게를 내림차순으로 정렬하고 싶다면?

- reverse 디폴트 메서드 사용

```java
inventory.sort(comparing(Apple::getWeight).reversed());
```

&nbsp;
## Comparator 연결

하지만 무게가 같은 두 사과가 있다면?

- 무게가 같은 사과는 원산지 국가별로 사과를 정렬하자
    - thenComparing : 함수를 인수로 받아 첫 번째 비교자를 이용해 두 객체가 같다고 판단되면 두 번째 비교자에게 객체를 전달

```java
inventory.sort(comparing(Apple::getWeight).reversed().thenComparing(Apple::getCountry));
```

&nbsp;
## Predicate 조합

복잡한 프레디케이트를 만들 수 있도록 negate, and, or 세 가지 메서드를 제공한다

(negate는 특정 프레디케이트를 반전시킬 때 사용)

- 기존 프레디케이트 객체 redApple의 결과를 반전시킨 객체 생성

```java
Predicate<Apple> notRedApple = redApple.negate();
```

- and 메서드를 이용해 빨간색이면서 무거운 사과 선택하도록 람다 조합

```java
Predicate<Apple> redAndHeavyApple = 
redApple.and(apple -> apple.getWeight() > 150);
```

- or을 이용해 빨간색이면서 무거운 사과 또는 그냥 녹색사과

```java
Predicate<Apple> redAndHeavyAppleOrGreen = 
	redApple.and(apple -> apple.getWeight() > 150)
		.or(apple -> GREEN.equals(a.getColor()));
```

→ 단순한 람다 표현식을 조합해서 더 복잡한 람다 표현식을 만들 수 있다

&nbsp;
## Function 조합

Function 인터페이스를 반환하는 andThen, compose 디폴트 메서드를 제공한다

- andThen 메서드
    - 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환한다
    - 증가시키는 함수 f와 2를 곱하는 함수 g를 조합

    ```java
    Function<Integer, Integer> f = x -> x + 1;
    Function<Integer, Integer> g = x -> x * 2;
    Function<Integer, Integer> h = f.andThen(g);
    int result = h.apply(1); // 4 반환
    ```


- compose 메서드
    - 인수로 주어진 함수를 먼저 실행한 다음에 그 결과를 외부 함수의 인수로 제공한다
    - `f.andThen(g)`에서 andThen 대신 compose를 사용하면 g(f(x))가 아닌 f(g(x))가 된다

    ```java
    Function<Integer, Integer> f = x -> x + 1;
    Function<Integer, Integer> g = x -> x * 2;
    Function<Integer, Integer> h = f.compose(g);
    int result = h.apply(1); // 3을 반환
    ```


- 여러 유틸리티 메서드 조합

```java
public class Letter {
		public static String addHeader(String text) {
				return "From Raoul, Mairo and Alan: " + text;
		}	
		public static String addFotter(String text) {
				return text = " Kind regards";
		}
		public static String checkSpelling(String text) {
				return text.replaceAll("lambda', "lambda");
		}
}
```

```java
Function<String, String> addHeader = Letter::addHeader;
Function<String, String> transformationPipeline = 
		addHeader.andThen(Letter::checkSpelling)
							.andThen(Letter::addFooter);
// 헤더를 추가한 다음, 철자 검사를 하고, 마지막에 푸터를 추가
```

```java
Function<String, String> addHeader = Letter::addHeader;
Function<String, String> transformationPipeline = 
		addHeader.andThen(Letter::addFooter);
// 헤더를 추가한 다음, 푸터를 추가
```

&nbsp;  
&nbsp;

# 마치며

### 람다 표현식

- 익명 함수의 일종
- 이름 X, 파라미터 리스트, 바디, 반환형식, 예외 O
- 간결한 코드 작성 가능
- 표현식 스타일과 블록 스타일로 표현할 수 있다
    - 표현식 스타일 : (parameters) → expression
    - 블록 스타일 : (parameters) → {statements;}
- 함수형 인터페이스를 기대하는 곳에서만 사용 가능
    - 자바 프로그래머가 하나의 추상 메서드를 갖는 인터페이스에 이미 익숙하다
- 람다 표현식의 기대 형식을 대상 형식이라고 한다
- 실행 어라운드 패턴
    - 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다

### 함수형 인터페이스

- 하나의 추상 메서드만을 정의하는 인터페이스(디폴트 메서드는 많아도 상관X)
- 람다 표현식을 이용해 함수형 인터페이스의 추상 메서드를 즉석으로 제공 가능
    - 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급
- java.util.function 패키지는 Predicate<T>, Function<T, R>, Supplier<T>, Consumer<T>, BinaryOperator<T> 등을 포함해서 자주 사용하는 다양한 함수형 인터페이스를 제공
- 자바 8은 Predicate<T>와 Function<T, R> 같은 제네릭 함수형 인터페이스와 관련한 박싱 동작을 피할 수 있는 IntPredicate, IntToLongFunction 등과 같은 기본형 특화 인터페이스도 제공
- 확인된 예외를 던지는 동작을 허용 X
    - 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의
    - 람다를 try/catch 블록으로 감쌈
- Comparator, Predicate, Function 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메서드를 제공 (Function - addThen, compose)

### 메서드 참조

- 메서드 참조를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달 가능
- 새로운 기능이 아닌 하나의 메서드를 참조하는 람다를 편리하게 표현할 수 있는 문법
    - 정적 메서드 참조
    - 다양한 형식의 인스턴스 메서드 참조
    - 기존 객체의 인스턴스 메서드 참조
- 메서드 참조는 콘텍스트의 형식과 일치해야 한다
    - 컴파일러는 메서드 참조가 주어진 함수형 인터페이스와 호환하는지 확인해야 한다

---

**💭 메서드 참조를 사용했을 때 더 가독성이 떨어지는 경우는?**
