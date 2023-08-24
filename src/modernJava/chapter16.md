# CompletableFuture
### 동기 API 와 비동기 API

동기 API : 메서드를 호출한 다음에 메서드가 계산을 완료할 때까지 기다렸다가 메서드가 반환되면 호출자는 반환되는 값으로 계속 다른 동작을 수행한다..
호출자와 피호출자가 각각 다른 스레드에서 실행되는 상황이었더라도 호출자는 피호출자의 동작 완료를 기다렸을 것이다. 이처럼 동기API를 사용하는
상황을 블록 호출이라고 한다.

비동기 API : 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당한다. 이와 같은
비동기 API를 사용하는 상황을 비블록 호출이라고 한다. 다른 스레드에 할당된 나머지 계산 결과는 콜백메서드를 호출해서 전달하거나 호출자가 
'계산 결과가 끝날때까지 기다림' 메서드를 추가로 호출하면서 전달된다. 즉, 계산 동작을 수행하는 동작 비동기적으로 디스크 접근을 수행한다.
더 이상 수행할 동작이 없으면 디스크 블록이 메모리로 로딩될때까지 기다린다.


### 비동기 API 구현
```java
public class Shop {

    String name; 
    public Shop(String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        Shop shop = new Shop("BestShop");
        long start = System.nanoTime();

        Future<Double> futurePrice = shop.getPriceAsync("my favorite Produce");

        long invocationTime = ((System.nanoTime() -start) /1_000_000);

        System.out.println("Invocation returned after " + invocationTime);

        try {
            double price = futurePrice.get();

            System.out.printf("Price is %.2f%n", price);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        long retrievalTime = ((System.nanoTime() - start) / 1_000_000);

        System.out.println("Price returned after " + retrievalTime + "msecs");
    }

    public static void delay() {
        try {
            Thread.sleep(3000L);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    public double getPrice(String product) {
        return calculatePrice(product);
    }

    private double calculatePrice(String product) {
        Random random = new Random();
        delay();
        return random.nextDouble() * product.charAt(0) + product.charAt(1);
    }

    public Future<Double> getPriceAsync(String product) {
        CompletableFuture<Double> futurePrice = new CompletableFuture<>();
        new Thread( () -> {
            try {
                double price = calculatePrice(product);
                futurePrice.complete(price);
            } catch (Exception ex) {
                futurePrice.completeExceptionally(ex);
            }

        }).start();
        return futurePrice;
    }
}
```
