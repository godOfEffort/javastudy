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
