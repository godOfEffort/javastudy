# 2. 동작 파라미터화 코드 전달하기
동작 파라미터화란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미. 이 코드블록은 나중에 프로그램에서 호출한다.
즉, 코드 블록의 실행은 나중으로 미뤄진다.

## 2.1 변화하는 요구사항에 대응하기
동작 파라미터화를 사용하면 변화하는 요구사항에 유연하게 대응할 수 있다.

## 2.2 동작 파라미터 화
전략 디자인 패턴은 각 알고리즘을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택한다.
Applepredicate가 알고리즘 패밀리이고, AppleHeavyWeightPredicate와 AppleColorPredicate가 전략이다.
메서드가 다양한 전략을 받아서 내부적으로 다양한 동작을 수행할 수 있다.
```java

import java.util.ArrayList;
import java.util.List;

enum Color{
    GREEN("GREEN"), RED("RED");

    String color;

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    Color(String color) {
        this.color = color;
    }
}

class Apple {
    int weight;
    String color;

    public Apple(int weight, String color) {
        this.weight = weight;
        this.color = color;
    }

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }
}

interface ApplePredicate {
    boolean test(Apple apple);
}

class AppleHeavyWeightPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return apple.getWeight() > 300;
    }
}

class AppleColorPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return apple.getColor().equals("red");
    }
}


public class AppleMain {

    static <T> List<Apple> filter (List<Apple> list, ApplePredicate predicate) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple : list) {
            if(predicate.test(apple)) {
                result.add(apple);
            }
        }
        return result;
    }

    public static void main(String[] args) {
        List<Apple> red = List.of(
                new Apple(30, Color.GREEN.getColor()),
                new Apple(35, Color.RED.getColor()),
                new Apple(45, Color.GREEN.getColor()));

        List<Apple> filter = filter(red, new AppleColorPredicate());
        System.out.println("filter = " + filter);
    }
}
```
위와 같이 사용하면 한개의 파라미터로 다양한 동작을 구현할 수 있다.

## 2.3.1 익명 클래스
자바는 클래스의 선언과 인스턴스화를 동시에 수행하 수 있도록 익명클래스 기법을 제공한다.
동작 파라미터화의 경우 클래스를 계속 생성해야 하는데 익명클래스를 전달하면 이런 클래스 생성없이 바로
메소드의 동작을 직접 파라미터화 할 수 있다.
```java

    List<Apple> filter = filter(red, new AppleColorPredicate());
    // 이 코드를 아래와 같이 변경할 수 있다.
    List<Apple> filter = filter(red, new AppleColorPredicate(){
        @Override
        public boolean test(Apple apple) {
            return apple.getColor().equals("red");
        }
    });
``` 
## 2.3.3 람다 표현식 사용
익명클래스 코드를 사용하게 되면 코드가 길어지고 유지보수가 어려워 진다.
그래서 익명클래스를 람다표현식으로 변경하면 더욱 간단하게 동작 파라미터화를 사용할 수 있다.
```java

    List<Apple> filter = filter(red, new AppleColorPredicate());
    // 이 코드를 아래와 같이 변경할 수 있다.
    List<Apple> filter = filter(red, (Apple apple) -> RED.equals(apple.getColor()));
``` 

## 2.3.4 리스트 형식으로 추상화

```java
interface ApplePredicate {
    boolean test(T t);
}

static <T> List<T> filter (List<T> list, Predicate<T> predicate) {
    List<T> result = new ArrayList<>();
    for (T e : list) {
        if(predicate.test(e)) {
            result.add(e);
        }
    }
    return result;
}
```
이렇게 코드를 변경하면 Apple 뿐만 아니라 다양한 타입의 필터 메서드를 사용할 수 있다. 
