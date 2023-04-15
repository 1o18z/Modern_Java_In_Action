# Chapter 2) 동작 파라미터화 코드 전달하기

&nbsp;  

### 🌱계속 변하는 소비자 요구사항에 대응할 때, 엔지니어링적인 비용이 가장 최소화될 수 있으면서 새로 추가한 기능은 쉽게 구현할 수 있어야 한다

### 🌱장기적인 관점에서 유지보수가 쉬워야 한다

&nbsp;  

# 동작 파라미터화

: 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록

- 나중에 호출되어지는 코드 블록
    - 나중에 실행될 메서드의 인수로 코드 블록을 전달할 수 있다

즉, 코드 블록에 따라 메서드의 동작이 파라미터화 된다

&nbsp;  

# 변화하는 요구사항에 대응하기

> ***요구사항 : 기존의 재고목록 애플리케이션에 리스트에서 녹색 사과만 필터링***
> 

 

&nbsp;  

### 2.1 - 녹색 사과만 필터링

```java
enum Color {
    RED,
    GREEN
}
```

```java
public static List<Apple> filterGreenApplesList(List<Apple> inventory){
				List<Apple> result = new ArrayList<>();
				for (Apple apple : inventory) {
							if (Color.GREEN.equals(apple.getColor())) {
									result.add(apple);
							}
				}
				return result;
}
```

- 빨간 사과도 필터링 하라는 요구사항이 추가로 들어왔다면 기존 filterGreenApples 메서드를 복사해 새로운 filterRedApples 메서드를 만들어 필터링 빨간 사과를 필터링 할 수 있다

→ 하지만 이렇게 되면 더 다양한 색으로 필터링하는 등의 변화에 적절히 대응 불가

**⇒ 이런 상황처럼 거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화하면 좋다**

&nbsp;  

### 2.2 - 색을 파라미터화

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color){
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (apple.getColor().equals(color)) {
                result.add(apple);
            }
        }
        return result;
}
```

여기서 색 이외에도 사과 무게를 구분할 수 있어야 한다는 요구사항이 들어오면 색이 아닌 무게 정보 파라미터를 받는 메서드를 만들어(복사해) 대응할 수 있다

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight){
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (apple.getWeight() > weight) {
                result.add(apple);
            }
        }
        return result;
}
```

하지만 이렇게 되면 중복되는 코드가 많다

→ **DRY**(같은 것을 반복하지 말 것) 원칙을 어기게 된다

&nbsp;  

### 2.3 - 가능한 모든 속성으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag){
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if((flag && apple.getColor().equals(color)) ||
                (!flag &&apple.getWeight() > weight)) {
                result.add(apple);
            }
        }
        return result;
}
```

이렇게 작성할 수는 있지만, 이렇게 되면 앞으로 요구사항이 바뀌면 유연하게 대응 불가하다

→ 요구사항이 더 추가된다면 중복된 필터 메서드를 만들거나 아니면 모든 것을 처리하는 거대한 하나의 필터 메서드를 구현해야 한다

⇒ 위와 같은 상황처럼 변화하는 요구사항에 보다 유연하게 대응하기 위해서

**동작 파라미터화**를 이용할 수 있다

&nbsp;  

### 2.4 - 추상적 조건으로 필터링

**Predicate**

: 선택 조건을 결정하는 인터페이스 (참 거짓을 반환)

```java
public interface ApplePredicate {
    boolean test(Apple apple);
}
```

```java
class AppleHeavyPredicate implements ApplePredicate{
    
    public boolean test(Apple apple) {
        return false;
    }
}

class AppleColorPredicate implements ApplePredicate{

    public boolean test(Apple apple) {
        return false;
    }
}
```

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if(p.test(apple)){
                result.add(apple);
            }
        }
        return result;
}
```

이제 필요한 대로 다양한 ApplePredicate를 만들어서 filterApples 메서드로 전달 가능하다

→ 유연한 코드

 ****

- **컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이 강점**
    - **한 메서드가 다른 동작을 수행하도록 재활용할 수 있다**
        - **따라서 유연한 API를 만들 때 동작 파라미터화가 중요한 역할**
 ****
&nbsp;  

# 복잡한 과정 간소화

현재 filterApples 메서드로 새로운 동작을 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의한 다음에 인스턴스화해야 한다

→ 번거롭고 쓸데없는 코드가 많이 추가된다

**익명 클래스**를 이용하면 코드의 양을 줄일 수 있다

- 클래스 선언과 인스턴스화를 동시에 할 수 있다
    - 즉, 즉석에서 필요한 구현을 만들어서 사용할 수 있다

&nbsp;  

### 2.5 - 익명 클래스 사용

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
            public boolean test(Apple apple) {
                return RED.equals(apple.getColor());
            }
});
```

- 하지만
    - 여전히 많은 공간을 차지한다
    - 많은 프로그래머가 익명 클래스의 사용에 익숙치 않다
- → 코드 조각을 전달하는 과정에서 결국은 객체를 만들고 명시적으로 새로운 동작을 정의하는 메서드를 구현한다는 점은 변하지 않는다

**⇒ 익명클래스로 인한 반복되는 지저분한 코드를 깔끔하게 해결하기 위해 람다 표현식 사용**

&nbsp;  

### 2.6 - 람다 표현식 사용

```java
List<Apple> result = 
	filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

훨씬 간결해진 걸 볼 수 있다

&nbsp;  

### 2.7 - 리스트 형식으로 추상화

```java
public static <T> List<T> filterList(List<T> inventory, Predicate<T> p) {
        List<T> result = new ArrayList<>();
        for (T e : inventory) {
            if (p.test(e)) {
                result.add(e);
            }
        }
        return result;
}
```

&nbsp;  

---

# 실전 예제

동작 파라미터화 패턴은 동작을 캡슐화한 후 메서드로 전달해 메서드의 동작을 파라미터화 한다

### Comparator로 정렬

```java
public interface Comparator<T> {
		int compare(T o1, T o2);
}
```

```java
inventory.sort(new Comparator<Apple>() {
		public int compare(Apple a1, Apple a2) {
				return a1.getWeight().compareTo(a.getWeight());
		}
});
```

```java
inventory.sort(
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWight()));
```

&nbsp;  

### Runnable로 코드 블록 실행

나중에 실행할 수 있는 코드를 구현하려면

자바 8까지는 Thread 생성자에 객체만을 전달할 수 있었으므로 보통 결과를 반환하지 않는 void run 메소드를 포함하는 익명 클래스가 Runnable 인터페이스를 구현하도록 하는 것이 일반적인 방법

```java
public interface Runnable {
		void run();
}
```

```java
Thread t = new Thread(new Runnable() {
		public void run() {
				System.out.println("Hello world");
		}
});
```

자바 8부터는 람다를 이용해 스레드 코드 구현 가능

```java
Thread t = new Thread(
		() -> System.out.println("Hello world"));
```

&nbsp;  

### Callable을 결과로 반환

ExecutorService 인터페이스는 태스크 제출과 실행 과정의 연관성을 끊어준다

- 테스크를 스레드 풀로 보내고 결과를 Future로 저장할 수 있다는 점이 스레드와 Runnable을 이용한는 방식과 다름
- Callable 인터페이스를 이용해 결과를 반환하는 태스크를 만든다

```java
public interface Callable<V> {
		V call();
}
```

```java
ExecutorService executorService = Executors.newCachedThreadPool();
Future<String> threadName = executorService.submit(new Callable<String>() {
		@Override
				public String call() throws Exception {
						return Thread.currentThread().getName();
		}
});
```

람다를 이용하면

```java
Future<String> threadName = executorService.submit(
		() -> Thread.currentThread().getName());
```

&nbsp;  

### GUI 이벤트 처리하기

GUI프로그래밍 : 마우스 클릭이나 문자열 위로 이동하는 버튼 등의 이벤트에 대응하는 동작을 수행

자바FX에서는 setOnAction 메서드에 EventHandler를 전달함으로써 이벤트에 어떻게 반응할지 설정할 수 있다

```java
Button button = new Button("Send");
button.setOnAction(new EventHandler<ActionEvent>() {
		public void handle(ActionEvent event) {
				label.setText("Sent~~");
		}
});
```

람다를 이용하면

```java
button.setOnAction(
		(ActionEvent event) -> label.setText("Sent~~"));
```

&nbsp;  

## 마무리

- 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있으며 나중에 엔지니어링 비용을 줄일 수 있다
- 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있지만 자바 8 이전에는 코드를 지저분하게 구현해야 했다
    - 익명 클래스로도 어느 정도 깔끔하게 할 순 있지만 자바 8에서는 인터페이스를 상속받아 여러 클래스를 구현하는 걸 개선하는 방법을 제공한다
- 자바 API의 많은 메서드는 정렬, 스레드, GUI 처리 등을 포함한 다양한 동작으로 파라미터화할 수 있다