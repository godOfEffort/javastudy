# 리팩터링, 테스팅, 디버깅
### 익명 클래스를 람다 표현식으로 리팩터링
1) 익명 클래스에서 this는 익명 클래스 자신을 가리키고, 람다에서 this는 람다를 감싸는 클래스를 가리킨다.
car가 null이면 빈 Optional 객체가 반환된다.
2) 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다.

### 람다 표현식을 메서드 참조로 리팩터링 하기
```java
   Map<CaloricLevel, List<Dish>> dishedByCaloricLevel = menu.stream()
                                                    .collect(groupingBy(dish -> {
                                                        if(dish.getCalories() <= 400) return Caloriclevel.DIET;
                                                        else if (dish.getCalories() <= 700) return Caloriclevel.NORMAL;
                                                        else return Caloriclevel.FAT;
                                                    }
```
아래와 같이 별도의 메서드로 추출해서 groupingBy에 인수로 전달 가능

```java
   public CaloricLevel getCaloricLevel() {
        if(dish.getCalories() <= 400) return Caloriclevel.DIET;
        else if (dish.getCalories() <= 700) return Caloriclevel.NORMAL;
        else return Caloriclevel.FAT;
   }
```

```java
   Map<CaloricLevel, List<Dish>> dishedByCaloricLevel = menu.stream()
                                                    .collect(groupingBy(Dish::getCaloricLevel));
```

### 명령형 데이터 처리를 스트림으로 리팩터링
```java
    List<String> dishNames = new ArrayList<>();
    for(Dish dish: menu) {
        if(dish.getCalories() > 300) {
            dishNames.add(dish.getName());
        }
    } 
```
아래와 같이 리팩토링 가능하다.
```java
    List<String> dishNames = new ArrayList<>();
    menu.parallelStream()
       .filter(d -> getCalories() > 300)
       .map(Dish::getName)
       .collect(toList());
```

### 코드 유연성 개선

1) 조건부 연기 실행
```java
   if(logger.isLoggable(Log.FINER)) {
        logger.finer("Problem" + generateDiagnostic());
   }
```
위 코드는 logger의 상태가 isLoggable이라는 메서드에 의해 클라이언트 코드로 노출된다.
메시지를 로깅할 때마다 logger객체의 상태를 매번 확인해야 한다.
```
   logger.log(Level.FINER, "problem" + generateDiagnostic());
```
위 코드와 같이 수정할 수 있으나 인수로 전달된 메시지 수준에서 logger가 활성화되어 있지 않더라도
항상 로깅 메시지를 평가하게 된다.
자바 8 API에서는 Supplier를 인수로 갖는 오버로드된 log메소드가 있다.
```java
   public void log(Level level, Supplier<String> msgSupplier)
```

다음처럼 log 메서드를 호출 가능하다.
```
   logger.log(Level.FINER, () -> "problem" + generateDiagnostic()); 
```
log메서드는 logger의 수준이 적절하게 설정되어 있을 때만 인수로 넘겨진 람다를 내부적으로 실행한다.
```java
   public void log(Level level, Supplier<String> msgSupplier) {
    if(logger.isLoggable(level)) {
        log(level, msgSupplier.get());
    }
   } 
```

### 람다로 객체지향 디자인 패턴 리팩터링
1) 전략 디자인 패턴
```java
   public interface ValidationStrategy {
        boolean execute(String s);
   } 

   public class IsAllLowerCase implements ValidationStrategy {
     public boolean execute(String s) {
        return s.matches("[a-z]+");
     }
   }

   public class IsNumberic implements ValidationStrategy {
     public boolean execute(String s) {
        return s.matches("\\d+");
     }
   }

   public class Validator {
       private final ValidationStrategy strategy;
       public Validator(ValidationStrategy v) {
            this.strategy = v;
       }
       
       public boolean validate(String s) {
           return strategy.execute(s);
       }
   }

    Validator numericValidator = new Validator(new IsNumberic());
    boolean b1 = numericValidator.validate("aaaa");
    Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
    boolean b2 = numericValidator.validate("bbbb");

    //람다 표현식 사용
    Validator numericValidator = new Validator((String s) -> s.matches("\\d+"));
    boolean b1 = numericValidator.validate("aaaa");
    Validator lowerCaseValidator = new Validator((String s) -> s.matches("\\d+"));
    boolean b2 = lowerCaseValidator.validate("bbbb");    
```

2) 템플릿 메서드
```java
    abstract class OnlineBanking {
        public void processCustomer(int id) {
            Customer c = Database.getCustomerWithId(id);
            makeCustomerHappy();
        }
        abstract void makeCustomerHappy(Customer c);       
    }

    public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
        Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy.accpet(c);        
    }
    new OnlineBanking().processCustomer(1337, (Customer c -> System.out.println("Hello" + c.getName())));
```
3) 옵저버
어떤 이벤트가 발생했을 때 한 객체(주제)가 다른 객체 리스트(옵저버)에 자동으로 알림을 보내야 하는 상황에서 사용하는 패턴
```java
    interface Observer{
        void notify(String tweet);
    }

    class NYTimes implements Observer {
        public void notify(String tweet) {
            if(tweet != null && tweet.contains("money")) {
                System.out.println("Breaking news in NY! " + tweet);
            }    
        }   
    }

    class Guardian implements Observer {
        public void notify(String tweet) {
            if(tweet != null && tweet.contains("queen")) {
                System.out.println("Breaking news in Guardian! " + tweet);
            }    
        }   
    }

    class LeMonde implements Observer {
        public void notify(String tweet) {
            if(tweet != null && tweet.contains("wine")) {
                System.out.println("Breaking news in LeMonde! " + tweet);
            }    
        }   
    }
    
    interface Subject {
        void registerObserver(Observer o);
        void notifyObservers(String tweet);
    }

    class Feed implements Subject {
        private final List<Observer> observers = new ArrayList<>();
        public void registerObserver(Observer o) {
            this.observers.add(o);
        }
        public void notifyObservers(String tweet) {
            observers.forEach(o -> o.notify(tweet));
        }
    }

    Feed f = new Feed();
    f.registerObserver(new NYTimes());
    f.registerObserver(new LeMonde());
    f.registerObserver(new Guardian());
    f.notifyObservers("The queen said her favourite book is Moder java");
    
    //람다표현식 사용
    f.registerObserver((String tweet) -> {
        if(tweet != null && tweet.contains("wine")) {
            System.out.println("Breaking news in LeMonde! " + tweet);
        }
    });
```
4) 의무체인
한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체도 해야 할 작업을 처리한 다음에 또다른 객체로 전달하는 방식.
```java
    public abstract class ProcessingObject<T> {
        protected ProcessingObject<T> successor;
        public void setSuccessor(ProcessingObject<T> successor) {
            this.successor = successor;
        }   
        
        public T handle(T input) {
            T r = handleWork(input);
            if(successor !=null) {
                return successor.handle(r);
            }
        }
        abstract protected T handleWork(T input);
    }

    public class HeaderTextProcessing extends ProcessingObject<String> {
        public String handleWork(String text) {
            return "From Raoul, Mario and Alan:" + text;
        }   
    }

    public class SpellCheckerProcessing extends ProcessingObject<String> {
        public String handleWork(String text) {
            return text.replaceAll("labda","lambda");
        }   
    }

    ProcessingObject<String> p1 = new HeaderTextProcessing();
    ProcessingObject<String> p2 = new SpellCheckerProcessing();
    p1.setSuccessor(p2);
    String result = p1.handle("Aren't labdas");
    System.out.println(result);

    //람다 표현식 사용
    UnaryOperator<String> headerProcessing = (String text) -> "From Raoul, Mario and Alan:" + text;
    UnaryOperator<String> pellCheckerProcessing = (String text) -> text.replaceAll("labda","lambda");
    Function<String, String> pipeline =  headerProcessing.andThen(pellCheckerProcessing);
    String result = pipeline.apply("Aren't labdas");
```
5) 팩토리
인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들때 팩토리 디자인 패턴을 사용
```java
public class ProductFactory {
  public static Produce createProduct(String name) {
    switch (name) {
        case "loan" : return new Loan();
        case "stock" : return new Stcok();
        case "bond" : return new Bond();
        default: throw new RuntimeException("No such product" + name);
    }
  }
}
Product p = ProductFactory.createProduct("loan");
//람다 표현식 사용
Supplier<Product> loanSupplier = Loan::new;
Loan loan  = loanSupplier.get();
//아래 처럼 상품명을 생성자로 연결하는 Map을 만들어서 코드를 재구현
final static Map<String, Supplier<Produce>> map = new HashMap<>();
static {
    map.put("loan", Loan::new);
    map.put("stock", Stock::new);
    map.put("bond", Bond::new);
}
//아래와 같이 사용가능
  public static Produce createProduct(String name) {
    Supplier<Product> p = map.get(name);
    if(p !=null) {
        return p.get();
    }
    throw new IllegalArgumentException("No such product" + name);
  }
//여러 인수를 전달하는 상황의 경우 별도의 인터페이스를 만들어야 한다.
    
```