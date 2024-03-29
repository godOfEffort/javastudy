# CompletableFuture와 리액티브 프로그래밍 컨셉
### 동시성을 구현하는 자바 지원의 진화

자바 8의 CompletableFuture와 java.util.concurrent.Flow의 궁극적인 목표는 가능한한 동시에 실행할 수 있는 독립적인
태스크를 가능하게 만들면서 멀티코어 또는 여러 기기를 통해 제공되는 병렬성을 쉽게 이용하는 것.


### Executor와 스레드 풀

1) 스레드 풀이 좋은 이유

  - 하드웨어에 맞는 수의 태스크를 유지함과 동시에 수 천 개의 태스크를 스레드 풀에 아무 오버헤드 없이 제출할 수 있다는 점이다.

2) 스레드풀이 나쁜 이유

  - k 스레드를 가진 스레드풀은 오직 k만큼의 스레드를 동시에 실행 가능.
  평소에는 큰 문제가 되지 않으나 어떤 태스크가 I/O를 기다리거나 네트워크 연결을 기다리면
  나머지 태스크는 앞의 태스크가 끝날때까지 대기해야 하는 상황이 된다.
  중요한 코드를 실행하는 스레드가 죽는 일이 발생하지 않도록 자바 프로그램은 main이 반환하기 전에 모든 스레드의 작업이
  끝나길 기다린다. 따라서 프로그램을 종료하기 전에 모든 스레드 풀을 종료하는 습관을 갖는것이 좋다. 자바는 이런 상황에
  대비할 수 있도록 Thread.setDaemon 메서드를 제공한다.

### 스레드의 다른 추상화 : 중첩되지 않은 메서드 호출

스레드 실행은 메서드를 호출한 다음의 코드와 동시에 실행되므로 데이터 경쟁 문제를 일으키지 않도록 주의해야 한다.

기존 실행 중이던 스레드가 종료되지 않은 상황에서 자바의 main() 메서드가 반환하면 어떻게 될까?
아래 두 가지 방법이 있으나 모두 안전하지 못하다.

  - 애플리케이션을 종료하지 못하고 모든 스레드가 실행을 끝날때까지 기다린다.
  - 애플리케이션 종료를 방해하는 스레드를 강제종료 시키고 애플리케이션을 종료한다. 데몬스레드는 애플리케이션이 종료될 때 강제종료되므로
  디스크의 데이터 일관성을 파괴하지 않는 동작을 수행할 때 유용하게 활용 가능. main() 메서드는 모든 비데몬 스레드가 종료될 때까지 프로그램을
  종료하지 않고 기다린다.
  
### CompletableFuture와 콤비네이터를 이용한 동시성

```java
public class CfComplete {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    int x = 1337;
    
    CompletableFuture<Integer> a = new CompletableFuture<>();
    executorService.submit(() -> a.complete(f(x)));
    int b = g(x);
    System.out.prinln(a.get() + b);
    executorService.shutdown();
}
```

위 코드는 f(x)의 실행이 끝나지 않거나 g(x)의 실행이 끝나지 않는 상황에서 get()을 기다려야 하므로 프로세싱 자원을 낭비할 수 있다.
이 문제는 Future를 조합해 해결할 수 있다.

```java
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    int x = 1337;
    
    CompletableFuture<Integer> a = new CompletableFuture<>();
    CompletableFuture<Integer> b = new CompletableFuture<>();
    CompletableFuture<Integer> c = a.thenCombine(b, (y,z) -> y+z);

    executorService.submit(() -> a.complete(f(x)));
    executorService.submit(() -> b.complete(g(x)));

    System.out.prinln(c.get());
    executorService.shutdown();
```

thenCombine이 핵심이다. Future a와 b의 결과를 알지 못한 상태에서 thenCombine은 두 연산이 
끝났을때 스레드 풀에서 실행된 연산을 만든다. 결과를 추가하는 세 번째 연산 c는 다른 두 작업이
끝날때까지는 실행되지 않는다.

### 발행-구독 리액티브 프로그래밍

Future와 CompletableFuture는 독립적 실행과 병렬성이라는 정식적 모델에 기반한다. 연산이 끝나면
get()으로 Future의 결과를 얻을 수 있다. 따라서 Future는 한 번만 실행해 결과를 제공한다.

반면 리액티브 프로그래밍은 시간이 흐르면서 여러 Future 같은 객체를 통해 여러 결과를 제공한다.
자바 9에서는 java.util.concurrent.Flow의 인터페이스에 발행-구독 모델을 적용해 리액티브 프로그래밍을 제공한다.

아래와 같이 세 가지 플로 API로 정리할 수 있다.
1) 구독자가 구독할 수 있는 발행자
2) 이 연결을 구독이라 한다.
3) 이 연결을 이용해서 메시지(또는 이벤트로 알려짐)를 전송한다.

### 두 플로를 합치는 예제
엑셀의 C1이나 C2의 값이 갱신되면 C3에도 새로운 값이 반영되는것처럼 예제를 구성해보자
```java
public class ArithmeticCell extends SimpleCell{
    private int left;
    private int right;

    public ArithmeticCell(String name) {
        super(name);
    }

    public void setLeft(int left) {
        this.left = left;
        onNext(left + this.right);
    }

    public void setRight(int right) {
        this.right = right;
        onNext(right + this.left);
    }
}
```

```java

interface Publisher<T> {
    void subscribe(Subscriber<? super T> subscriber);
}

interface Subscriber<T> {
    void onNext(T t);
}

public class SimpleCell implements Publisher<Integer>, Subscriber<Integer> {
    private int value = 0;
    private String name;
    private List<Subscriber> subscribers = new ArrayList<>();
    public SimpleCell(String name) {
        this.name = name;
    }

    @Override
    public void subscribe(Subscriber<? super Integer> subscriber) {
        subscribers.add(subscriber);
    }

    private void notifyAllSubscribers() {
        subscribers.forEach(subscriber -> subscriber.onNext(this.value));
    }

    @Override
    public void onNext(Integer newValue) {
        this.value = newValue;
        System.out.println(this.name + ":" + this.value);
        notifyAllSubscribers();
    }

    public static void main(String[] args) {
//        SimpleCell c3 = new SimpleCell("C3");
//        SimpleCell c2 = new SimpleCell("C2");
//        SimpleCell c1 = new SimpleCell("C1");
//
//        c1.subscribe(c3);
//
//        c1.onNext(10);
//        c2.onNext(20);

        ArithmeticCell c3 = new ArithmeticCell("C3");
        SimpleCell c2 = new SimpleCell("C2");
        SimpleCell c1 = new SimpleCell("C1");

        c1.subscribe(c3::setLeft);
        c2.subscribe(c3::setRight);

        c1.onNext(10);
        c2.onNext(20);
        c1.onNext(15);
    }
}
```
데이터가 발행자(생산자)에서 구독자(소비자)로 흐름에 착안해 개발자는 이를 업스트림 또는 다운스트림이라 부른다.
위 예제에서 데이터 newValue는 업스트림 onNext() 메서드로 전달되고 notifyAllSubscribers() 호출을 통해 다운 스트림
onNext() 호출로 전달된다.

### 압력과 역압력

빠른 속도로 발생하는 이벤트를 처리해야 하는 상황을 압력이라고 한다.
자바 9 플로 API에서는 발행자가 무한의 속도로 아이템을 방출하는 대신 요청 했을때만 다음 아이템을
보내도록 하는 request() 메서드를 제공한다.

Subscriber에서 Pulisher로 정보를 요청해야 할 필요가 있을 수 있다. 자바 플로 API의 Subscriber
인터페이스는 아래 메서드를 포함한다.

```java
    void onSubscribe(Subscription subscription);
```

Publisher와 Subscriber 사이에 채널이 연결되면 첫 이벤트로 이 메서드가 호출된다.

Subscription 객체는 아래와 같은 메서드를 포함한다.

```java
    interface Subscription {
        void cancel();
        void request(long n);
    }
``` 
Publisher는 Subscription 객체를 만들어 Subscriber로 전달하면 Subscriber는 이를 이용해
Publisher로 정보를 보낼 수 있다.

### 역압력의 간단한 형태

한 번에 한개의 이벤트를 처리하도록 발행-구독 연결을 구성하려면 다음과 같은 작업이 필요
  - Subscriber가 Onsubscribe로 전달된 Subscription 객체를 subscription 같은 필드에 로컬로 저장
  - Subscriber가 수많은 이벤트를 받지 않도록 onSubscribe, onNext, onError의 마지막 동작에 channel.request(1)을 추가해
    오직 한 이벤트만 요청
  - 요청을 보낸 채널에만 onNext, onError 이벤트를 보내도록 Publisher의 notifyAllSubscribers 코드를 바꾼다.


### 리액티브 시스템 vs 리액티브 프로그래밍
리액티브 시스템이 가져야 할 세 가지 속성
  - 반응성 : 큰 작업을 처리하느라 간단한 질의의 응답을 지연하지 않고 실시간으로 입력에 반응
  - 회복성 : 네트워크가 고장났어도 관련없는 질의에는 아무 영향이 없어야 하며 반응이 없는 컴포넌트를 향한 질의는 다른 대안 컴포넌트를 찾아야 한다.
  - 탄력성 : 시스템이 자신의 작업 부하에 맞게 적응하며 작업을 효율적으로 처리함을 의미 
