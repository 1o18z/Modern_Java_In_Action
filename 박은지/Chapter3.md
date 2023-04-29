# Chapter 3) 람다 표현식


## ~~이전 장

- 동적 파라미터화를 이용해 변화하는 요구사항에 효과적으로 대응하는 코드를 구현할 수 있다
    - 또한 정의한 코드 블록을 다른 메서드로 전달할 수 있다
        - 따라서 동작 파라미터화를 이용하면 더 유연하고 재사용할 수 있는 코드를 만들 수 있다
- 익명 클래스로 다양한 동작을 구현할 수 있지만 코드가 깔끔하지 못했고, 이는 실전에 동작 파라미터를 적용하는 것을 막는다
    - 람다를 통해 익명 클래스처럼 이름 없는 함수면서 메서드를 인수로 전달할 수 있다

&nbsp; 

# 람다
-> 메서드로 전달할 수 있는 익명 함수를 단순화한 것

&nbsp;

### 람다 특징

- 익명
    - 구현해야 할 코드에 대한 걱정거리 감소
- 함수
    - 메서드처럼 특정 클래스에 종속되지 X
    - 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트 포함
- 전달
    - 람다 표현식을 메서드 인수로 전달하거나 변수로 저장 가능
- 간결성

⇒ 동작 파라미터 형식의 코드를 더 쉽게 구현할 수 있어 코드가 간결해지고 유연해진다

```java
Comparator<Apple> byWeight = new Comparator<Apple>() {
		public int compare(Apple a1, Apple a2) {
				return a1.getWeight().compareTo(a2.getWeight());
		}
};
```

람다를 이용하면

```java
Comparator<Apple> byWeight = 
		(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

훨씬 간결해진 것을 볼 수 있다

&nbsp;


`(Apple a1, Apple a2) → a1.getWeight().compareTo(a2.getWeight());`

- 파라미터 리스트
- 화살표
- 람다 바디

&nbsp;


### 람다의 기본 문법

- **표현식 스타일 람다**

`(parameters) → expression`

- () → “Raoul”
- (String s) → “Iron Man”

&nbsp;


- **블록 스타일 람다**

`(parameters) → { statements; }`

- () → {}
- () → { return “Mario”; }
- (String s) → { return “Iron Man”; }

&nbsp;
&nbsp;



# 어디에, 어떻게 람다를 사용할까?

→ 함수형 인터페이스에서 사용 가능!

&nbsp;


## 함수형 인터페이스

-> 하나의 추상 메서드를 지정하는 인터페이스

(디폴트 메서드를 포함 가능하며, 디폴트 메서드가 여럿 있어도 추상 메서드가 하나라면 함수형 인터페이스이다)

- Comparator
- Runnable
- Callable
- . . .

&nbsp;

함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 전체 표현식을 함수형 인터페이스의 인스턴스로 취급 가능하다

```java
Runnable r1 = () -> System.out.println("hi1");  // 람다 사용

Runnable r2 = new Runnable() { //  익명 클래스 사용
		public void run() {
				System.out.println("hi2");
		}
};

public static void process(Runnable r) {
		r.run();
}

process(r1);
process(r2);
process(() -> System.out.println("hi3"));  // 직접 전달된 람다표현식으로 출력
```

&nbsp;
&nbsp;


## 함수 디스크립터
-> 람다 표현식의 시그니처를 서술하는 메서드

- Runnable 인터페이스의 추상메서드 run은 인수와 반환값이 없으므로 Runnable 인터페이스는 인수와 반환값이 없는 시그니처이다
- () → void 는 파라미터 리스트가 없으며 void를 반환하는 함수 의미
- (Apple, Apple) → int 는 두 개의 Apple을 인수로 받아서 int를 반환하는 함수를 의미

⇒ 뭘 받고 뭐가 반환되는지

&nbsp;

⇒ 람다는 변수에 할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달 가능하며, 함수형 인터페이스의 추상 메서드와 같은 시그니처를 갖는다

```java
public void process(Runnable r) {
		r.run();
}

process(() -> System.out.println("This is awesome"));
```

process(() -> System.out.println("This is awesome")); 은

인수가 없으며 void를 반환하는 람다 표현식으로 Runnable 인터페이스의 run 메서드 시그니처와 같다

&nbsp;
&nbsp;


# 람다 활용 : 실행 어라운드 패턴
-> 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는 패턴

```java
public String processFile() throws IOException {
		try (BufferedReader br = 
							new BufferedReader(new FileReader("data.txt"))) {
				return br.readLine(); // 실제 필요한 작업을 하는 행 
		}
}
```

&nbsp;


## 1단계 : 동작 파라미터화를 기억하라

파일을 한 번에 두 줄 읽으려면?

→ 기존의 설정, 정리 과정은 재사용하고 processFile 메서드만 다른 동작을 수행하도록 명령하자

→ processFile의 동작을 파라미터화 하자

```java
String result = processFile((BufferedReader br) -> 
															br.readLine() + br.readLine());
```

&nbsp;


## 2단계 : 함수형 인터페이스를 이용해서 동작 전달

함수형 인터페이스 자리에 람다 사용 가능

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
		String process(BufferedReader b) throws IOException;
}
```

BufferedReader → String과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스 생성

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
		 . . .
}
```

위에서 정의한 인터페이스를 processFile 메서드의 인수로 전달

&nbsp;


## 3단계 : 동작 실행

람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으며 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리

→ 따라서 processFile 바디 내에서 BufferedReaderProcessor 객체의 process를 호출 가능

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
		try (BufferedReader br = 
							new BufferedReader(new FileReader("data.txt"))) {
				return p.process(br);  // BufferedReader 객체 처리
		}
}
```

&nbsp;


## 4단계 : 람다 전달

이제 람다를 이용해 다양한 동작을 processFile 메서드로 전달 가능

```java
String oneLines = processFile((BufferedReader br) -> br.readLine());
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

&nbsp;
&nbsp;


# 함수형 인터페이스

람다 표현식으로 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요

&nbsp;


## Predicate

test라는 추상 메서드를 정의하며 T의 객체를 인수로 받아 불리언 반환

→ T 형식의 객체를 사용하는 불리언 표현식이 필요한 상황에서 사용

```java
@FunctionalInterface
public interface Predicate<T> {
		boolean test(T t);
}
public <T> List<T> filter(List<T> list, Predicate<T> p) {
		List<T> results = new ArrayList<>();
		for(T t : list) {
				if(p.test(t)) {
						results.add(t);
				}
		}
		return result;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
// String 객체를 인수로 받는 람다 정의
```

&nbsp;


## Consumer

T 객체를 받아서 void를 반환하는 accept라는 추상 메서드 정의

→ T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 사용

(리스트의 각 항목이 어떤 동작을 수행하는지)

```java
@FunctionalInterface
public interface Consumer<T> {
		void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
		for(T t : list) {
				c.accept(t);
		}
}
forEach(
		Arrays.asList(1, 2, 3, 4, 5),
		(Integer i) -> System.out.println(i)
);
```

&nbsp;


## Function

T를 인수로 받아서 제네릭 형식 R 객체를 반환하는 추상 메서드 apply 정의

→ 입력을 출력으로 매핑하는 람다를 정의할 때  사용 가능

(사과의 무게 정보 추출)

```java
@FunctionalInterface
public interface Function<T, R> {
		R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, T> f) {
		List<R> result = new ArrayList<>();
		for(T t : list) {
				result.add(f.apply(t));
		}
		return result;
}

List<Integer> l = map(
				Arrays.asList("lambdas", "in", "action")
				(String s) -> s.length()  // Function의 appy 메서드를 구현하는 람다
);
```

&nbsp;


## 기본형 특화

자바의 모든 형식은 참조형 아니면 기본형에 해당한다

제네릭 파라미터에는 참조형만 사용 가능 (내부 구현 때문에)

→ 자바에서는 기본형을 참조형으로 변환하는 기능 제공

→ **박싱**

(참조형을 기본형으로의 변환은 **언박싱**)

박싱과 언박싱이 자동으로 이루어지는 오토박싱 기능도 제공

```java
List<Integer> list = new ArrayList<>();
for(int i=300; i<400; i++) {
		list.add(i);
}
```

하지만 이런 변환 과정은 비용이 소모된다

(기본형을 감싸는 래퍼가 힙에 저장되고, 기본형 가져올 때도 메모리 탐색 과정 필요)

&nbsp;


자바 8은 기본형을 입출력으로 사용하는 상황에서 오토박싱 동작을 피할 수 있는 함수형 인터페이스를 제공한다

→ 일반적으로 특정 형식을 받는 함수형 인터페이스 이름 앞에는 형식명이 붙는다

- DoubleRredicate, IntConsumer, LongBinaryOperator
- Function 인터페이스는 IntToDoubleFunction 등 다양한 출력 형식 파라미터 제공

&nbsp;


### 예외, 람다, 함수형 인터페이스의 관계

함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다

→ 예외를 던지는 람다 표현식을 만들고 싶으면

- 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의한다

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
		String precess(BufferedReader b) throws IOException;
}
BufferedReaderProcessor p = (BufferedReader br) -> br.readLine();
```

- 람다를 try/catch 블록으로 감싼다

```java
Function<BufferedReader, String> f = (BufferedReader b) -> {
		try {
				return b.readLine();
		}
		catch(IOException e) {
				throw new RuntimeException(e);
		}
}
```

&nbsp;
&nbsp;


# 형식 검사, 형식 추론, 제약

람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보는 포함되어 있지 않다

→ 람다의 실제 형식을 파악해야 한다

&nbsp;


## 형식 검사

람다가 사용되는 콘텍스트를 이용해 형식을  추론 가능하다

대상 형식 : 어떤 콘텍스트에서 기대되는 람다 표현식의 형식

&nbsp;


## 같은 람다, 다른 함수형 인터페이스

대상 형식으로 인해 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다

```java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

위의 두 할당문 모두 인수를 받지 않고 제네릭 형식 T를 반환하는 함수를 정의하는 유효한 코드다

대상 형식을 이용해 람다 표현식을 특정 콘텍스트에 사용 가능하고, 대상 형식으로 람다의 파라미터 형식도 추론 가능하다

&nbsp;


### 특별한 void 호환 규칙

람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다

(파라미터 리스트도 호환되어야 함)

```java
Predicate<String> p = s -> list.add(s); // boolean 반환값 가짐
Consumer<String> b = s -> list.add(s); // void 반환값 가짐
```

위 두 행에서 List의 add 메서드는 Consumer 콘텍스트(T → void)가 기대하는 void 대신 boolean을 반환하지만 유효한 코드다

&nbsp;
&nbsp;



## 형식 추론

대상 형식을 이용해 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론 가능하다

→ 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략 가능하다

```java
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()); // 형식 추론 X
Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()); // 형식 추론
```

정해진 규칙은 없으니 상황에 따라 어떤 코드가 더 가동성을 향상 시킬지 판단하여 작성하자

&nbsp;
&nbsp;


## 지역 변수 사용

람다 표현식에서는 익명 함수처럼 자유 변수를 활용할 수 있다

- 자유 변수 : 파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수

이런 동작을 람다 캡처링이라고 부른다

→ 이 람다들을 **캡처링 람다**라고 부른다

&nbsp;


캡처링 람다들은 인스턴스 변수와 정적 변수, 지역 변수를 사용 가능하다

→ 지역 변수는 effectively final이나 final이어야 한다

- 람다가 사용하는 지역 변수에 final이 없다면 컴파일러가 알아챈다
- 컴파일러가 final을 붙여주기 때문에 해당 변수를 조작하려는 시도가 있으면 컴파일 에러를 뛰운다 ⇒ effectivel final

&nbsp;


### 캡처링 람다 안의 지역 변수

```java
Supplier<Integer> incrementer(int test) {
		return () -> test++; 
} // test는 지역 변수이고 람다 표현식 안에서 test를 조작하려 한다
```

람다가 test 값의 복사본을 만들려고 한다 → 컴파일 에러 발생

- 이 변수를 final로 강제하는 것은 test를 람다 안에서 증가시키는 것이 실제로 test 메서드 파라미터를 조작할 수 있다는 생각을 막아준다

복사본을 만드는 이유?

- 위 메서드는 람다를 반환한다
    - 때문에 람다는 test 메서드 파라미터가 GC에게 수집된 이후에는 동작하지 않는다
    - → 이 람다가 메서드 밖에서도 살아남으려면 test의 복사본을 만들어야 한다

&nbsp;


### 동시성 문제

만약 final 제약이 없어진다면?

```java
public void localVariableMultithreading() {
    boolean run = true;
    executor.execute(() -> {
        while (run) {
           . . .
        }
    });
    run = false;
}
```

투명성에 대한 잠재적인 문제

→ 각각의 스택을 가지는 여러 스레드에서 실행된다면 이 while문이 다른 스택의 run 변수의 변화를 감지할 수 있다는 것을 보장해야 한다

→ synchronized 블럭이나 volatile 키워드 사용

⇒ 자바는 effectively final을 강제하기 때문에 이런 복잡한 문제 신경쓰지 않아도 된다!

&nbsp;


### 캡처링 람다 안의 인스턴스, 정적 변수

```java
private int test = 0;

Supplier<Integer> incrementer() {
    return () -> test++;
} // 컴파일 성공
```

test를 어떻게 가져와서 쓰는 걸까?

→ 멤버 변수들이 어디에 저장되는 지를 보면 알 수 있다

- 지역 변수는 스택, 멤버 변수들은 힙에 저장된다
    - 힙 메모리를 사용하기 때문에 컴파일러는 람다가 test의 가장 최근의 값에 대해 접근할 수 있다는 것을 보장한다

```java
private volatile boolean run = true;

public void instanceVariableMultithreading() {
    executor.execute(() -> {
        while (run) {
            . . .
        }
    });

    run = false;
}
```

람다가 다른 스레드에서 실행되어도, volatile 키워드가 붙어있기 때문에 run 변수에 접근할 수 있게 됐다

- 인스턴스 변수를 캡처링할 때 final 변수인 this 캡처링 한다고 볼 수 있다

&nbsp;


### 피해야 할 해결법

변수 저장소를 사용해 제약을 피할 수 있지 않을까? 라는 생각이 들 수도 있다

```java
public int workaroundSingleThread() {
    int[] holder = new int[] { 2 };
    IntStream sums = IntStream
      .of(1, 2, 3)
      .map(val -> val + holder[0]);

    holder[0] = 0;

    return sums.sum();
}
```

스트림이 1,2,3에 holder[0]을 각각 더하는 예제이다

→ 람다가 실행될 때 가장 최근의 값을 사용하기 때문이다

일반적으로 이런 해결법들은 에러가 발생하기 쉽고 예상치 못한 결과가 나올 수도 있다

→ 피하자!
