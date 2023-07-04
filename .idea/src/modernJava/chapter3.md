# 3. 람다 표현식
람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이다.
1. 표현식 스타일의 람다
```java
    (parameters) -> expression
```
2. 블록 스타일의 람다
```java
    (parameters) -> { statements; }
```

```java
    {return "Alan" +i};
    // return문이 있을 경우 위와 같이 묶어주어야 한다.
```

## 3.1 어디에 어떻게 람다를 사용할까?
함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.

## 3.1.1 함수형 인터페이스
함수형 인터페이스는 오직 하나의 추상 메서드를 지정하는 인터페이스이다. 디폴트 메서드를 포함할 수 있으나
많은 디폴트 메서드가 있더라도 **추상 메서드가 오직 하나면 함수형 인터페이스다**.
함수형 인터페이스로 뭘 할 수 있을까? 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을
직접 전달할 수 있으므로 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.
```java
Runnable r1  = () -> System.out.println("hello world");

Runnable r2 = new Runable() {
    public void run() {
        System.out.println("hello world");
    };
};

public static void process(Runnable r) {
    r.run();
};

process(r1);
process(r2);
process(() -> System.out.println("hello world3")); //직접 전달된 람다표현식으로 hello world3 출력
```

## 3.2.2 함수 디스크립터
함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다. 람다 표현식의 시그니처를 서술하는 메서드를 함수 디스크립터라고 부른다.
예를 들어 Runnable 인터페이스의 유일한 추상메서드 run은 인수와 반환값이 없으므로(void 반환) Runnable 인터페이스는 인수와 반환값이 없는 시그니처
라고 생각할 수 있다.
```java
    () -> void // 파라미터 리스트가 없으며 반환 
    (Apple apple) -> int // 두 개의 Apple을 인수로 받아 int를 반환하는 함수
```
람다식은 변수에 할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달할 수 있으며,
함수형 인터페이스의 추상 메서드와 같은 시그니처를 갖는다.

## 3.2.3 Predicate
제네릭 형식의 T의 객체를 인수로 받아 불리언을 반환한다.
```
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> results = new ArrayList<>();
    for(T t: list) {
        if(p.test(t)) {
            results.add(t);
        }
        return results;
    }
}

Predicate<String> nonEmptyStringPredicate =  (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(list, nonEmptyStringPredicate);

```
## 3.2.4 Consumer
제네릭 형식 T 객체를 받아서 void를 반환하는 accept 추상메서드 정의
```
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
        Arrays.asList(1,2,3,4,5),
        (Integer i) -> System.out.println(i);
)
```

## 3.2.5 Function
java.util.function.Function<T,R> 인터페이스는 제네릭 형식 T를 인수로 받아서 제네릭 형식 R객체를 반환하는
추상 메서드 apply를 정의한다.
```
@FunctionalInterface
public interface Function<T,R> {
    R apply(T t);
}

public <T,R> List<R> map(List<T> list, Function<T,R> f) {
    List<R> result = new ArrayList<>();
    for(T t : list) {
        result.add(f.apply(t));
    }
    return result;
}

List<Integer> l = map(
        Arrays.asList("lambdas", "in", "action"),
        (String s ) -> s.length()
);
```

## 3.2.6 기본형 특화
제네릭은 참조형만 사용할 수 있으므로 오토박싱이 일어나 메모리를 더 소비하게 된다. 그래서 자바 8에서는
기본형을 입출력으로 사용하는 상황에서 오토박싱을 피할 수 있도록 특별한 버전의 함수형 인터페이스 제공
```
public interface IntPredicate {
    boolean test(int t);
}

IntPredicate evenNumbers = (int i) -> i % 2 == 0;
evenNumbers(1000); (박싱 없음)

Predicate<Integer> oddNumbers = (Integer i) -> i % 2 != 0;
oddNumbers.test(1000); (박싱)

```

## 3.2.7 함수형 인터페이스 예외처리
함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다. 즉, 예외를 던지는 람다표현식을
만들려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 람다를 try/catch 블록으로 감싸야 한다.

## 3.3 형식 검사, 형식 추론, 제약
1) 형식검사

```
List<Apple> heavierThan150g = filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```
여기서 두 번째 파라미터로 Predicate<Apple> 형식(대상형식)을 기대한다. Predicate<Apple> 인터페이스의 추상메서드는 무엇인가?
Apple을 인수로 받아 boolean을 반환하는 test 메서드다. 함수 디스크립터는 Apple -> boolean 이므로 람다의 시그니처와 일치한다.

2) 같은 람다, 다른 함수형 인터페이스

대상 형식이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다. 아래 코드는 모두 유효하다
```
Callable<Integer> c = () -> 42;
PrivlegedAction<Integer> p  = () -> 42;
```


3) 특별한 void 호환 규칙

람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다.
아래코드는 유효한 코드다.
```
Consumer<String> b = s -> list.add(s);
```


4) 형식 추론

자바 컴파일러는 람다 표현식이 사용된 콘텍스트(대상 형식)를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다.
즉, 대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다.
결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.

```
Comparator<Apple> c = (Apple a, Apple b) -> a1.getWeight().compareTo(a2.getWeight());
Comparator<Apple> c = (a, b) -> a1.getWeight().compareTo(a2.getWeight());
```

5) 지역변수 사용
람다식은 익명함수처럼 자유변수(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다. 이와 같은 동작을
람다캡처링이라고 부른다.
```
    int portNumber = 1377;
    Runnable r = () -> System.out.println(portNumber);
```
하지만 자유변수에도 약간의 제약이 있는데 
