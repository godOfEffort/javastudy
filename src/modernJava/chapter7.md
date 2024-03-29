# 병렬 데이터 처리와 성능
### 순차 스트림을 병렬 스트림으로 변환하기

순차 스트림에 parallel 메서드를 호출하면 기존의 함수형 리듀싱 연산이 병렬로 처리된다.
사실 순차 스트림에 parallel을 호출해도 스트림 자체에는 아무 변화도 일어나지 않는다. 
내부적으로는 불리언 플래그가 설정된다. 반대로 sequential로 병렬 스트림을 순차 스트림으로 
바꿀 수 있다. 이 두 메서드를 이용해서 어떤 연산을 병렬로, 순차로 실행할지 제어할 수 있다.
```java
stream.parallel()
     .filter(..)
     .sequential()
     .map(..)
     .paralle()
     .reduce();
```
parallel과 sequential 두 메서드 중 최종적으로 호출된 메서드가 전체 파이프라인에 영향을 미친다.
위 예제에서 파이프라인의 마지막 호출은 paralle이므로 파이프라인은 전체적으로 병렬로 실행된다.


### 병렬 스트림에서 사용하는 스레드 풀 설정

병렬 스트림은 내부적으로 ForkJoinPool을 사용한다. ForkJoinPool은 프로세서 수, 즉 Runtime.getRuntime().availableProcessors()가
반환하는 값에 상응하는 스레드를 갖는다.

### 병렬 스트림의 올바른 사용 방법

공유된 상태를 바꾸는 알고리즘을 사용하면 문제가 발생한다.

```java
public long sideEffectSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).forEach(accumulator::add);
    return accumulator.total;
}

public class Accumulator {
    public long total  = 0;
    public void add(long value) {
        total += value;
    }
}
```

위 코드는 순차적으로 실행하도록 구현되어 있으므로 병렬로 실행하면 안된다. 특히 total에 
접근할때마다 데이터 레이스 문제가 일어난다.

```java
public long sideEffectParallelSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);
    return accumulator.total;
}
```

병렬로 위와 같이 실행하면 올바른 결과값이 나오지 않는데 이유는 여러 스레드에서 동시에 누적자, 
total += value를 실행하면서 이런 문제가 발생한다. 아토믹 연산같지만 total += value는 아토믹
연산이 아니다. 여러 스레드에서 공유하는 객체의 상태를 바꾸는 forEach 블록 내부에서 add 메서드를
호출하면서 이같은 문제가 발생한다.

### 병렬 스트림 효과적으로 사용하기

특정 양을 기준으로 병렬 스트림 사용을 결정하는 것은 적절하지 않다. 아래와 같은 약간의 수량적 힌트가 도움이 될 때도 있다.

1) 직접 측정하기. 언제나 병렬 스트림이 순차 스트림보다 빠른건 아니기 때문에 적절한 벤치마크로 직접
성능을 측정하는 것이 바람직하다.

2) 박싱을 주의하라.

자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있다. 자바 8은 박싱 동작을 피할 수 있도록 기본형 특화 스트림을 제공한다.
되도록이면 기본형 특화 스트림을 사용하자.

3) 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다.

limit이나 findFirst처럼 요소의 순서에 의존하는 연산을 병렬스트림에서 수행하려면 비싼 비용을 치뤄야 한다.
정렬된 스트림에 unordered를 호출하면 비정렬된 스트림을 얻을 수 있다.

4) 스트림에서 수행하는 전체파이프라인 연산 비용 고려

처리해야 할 요소수가 N이고 하나의 요소를 처리하는데 드는 비용을 Q라 하면 전체 처리비용은 N*Q이다.
Q가 높아지는 것은 병렬 스트림으로 성능을 개선할 수 있는 가능성이 있음을 의미한다.

5) 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다.

6) 스트림을 구성하는 자료구조를 확인해야 한다.

예를 들어 ArrayList를 LinkedList보다 효율적으로 분할할 수 있다. LinkedList를 분할하려면
모든 요소를 탐색해야 하지만 ArrayList는 요소를 탐색하지 않고도 리스트를 분할할 수 있기 때문이다.
또한 range 팩토리 메서드로 만든 기본형 스트림도 쉽게 분해 가능하다.

7) 스트림의 특성과 파이프라인 중간 연산이 스트림의 특성을 어떠헤 바꾸는지에 따라 분해 과정의 성능이
달라질 수 있다. 예를 들어 SIZED 스트림은 정확히 같은 크기의 두 스트림으로 분할할 수 있으므로 효과적으로
스트림을 병렬 처리할 수 있다. 반면 필터 연산이 있으면 스트림의 길이를 예측할 수 없으므로
효과적으로 병렬 처리할 수 있을지 알 수 없게 된다.

8) 최종 연산의 병합과정(예를 들면 Collector의 combiner 메서드) 비용을 살펴보라. 병합 과정의
비용이 비싸다면 병렬 스트림으로 얻은 성능의 이익이 서브스트림의 부분 결과를 합치는 과정에서
상쇄될 수 있다.  


## 포크/조인 프레임워크

병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음에 서브태스크 각각의 결과를 합쳐서
전체 결과를 만들도록 설계되었다.

서브태스크를 스레드 풀의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현한다.
```
병렬화된 태스크가 생성하는 결과 형식이 있을때 : RecursiveTask<R>
병렬화된 태스크가 생성하는 결과 형식이 없을때 : RecursiveAction
```
RecursiveTask를 정의하려면 추상 메서드 compute를 구현해야 한다.
compute의 구현은 다음과 같은 의사 코드와 같음
```
if(태스크가 충분히 작거나 더 이상 분할할 수 없으면) {
    순차적으로 태스크 계산
} else {
    태스크를 두 서브태스크로 분할
    태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출함
    모든 서브태스크의 연산이 완료될 때까지 기다림
    각 서브 태스크의 결과를 합침
}
```


```java
public class ForkJoinSumCalculator extends RecursiveTask<Long> {

    private final long [] numbers;
    private final int start;
    private final int end;
    public static final long THRESHOLD = 10_000;

    public ForkJoinSumCalculator(long[] numbers) {
        this(numbers, 0, numbers.length);
    }

    private ForkJoinSumCalculator(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;
        if(length <= THRESHOLD) {
            return computeSequentially();
        }
        ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length/2);
        leftTask.fork();
        ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2, end);
        Long rightResult = rightTask.compute();
        Long leftResult = leftTask.join();
        return leftResult + rightResult;
    }

    private long computeSequentially() {
        long sum = 0;
        for(int i = start; i < end; i++) {
            sum += numbers[i];
        }
        return sum;
    }
    public static long forkJoinSum(Long n) {
        long[] numbers = LongStream.rangeClosed(1, n).toArray();
        ForkJoinSumCalculator task = new ForkJoinSumCalculator(numbers);
        return new ForkJoinPool().invoke(task);
    }

    public static void main(String[] args) {
        forkJoinSum(1000L);
    }

}

```

## 포크/조인 프레임워크를 제대로 사용하는 방법

1) join 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 떄까지 호출자를 블록 시킨다.
따라서 두 서브태스크가 모두 시작된 다음에 join을 호출해야 한다.
2) RecursiveTask 내에서는 invoke 메소드를 사용하지 말아야 한다. 순차 코드에서 병렬 계산을 시작할 때만 invoke를
사용한다.
3) 멀티코어에서 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠르지 않다. 성능을 측정해서 
사용해야 한다.

##  작업 훔치기

ForkJoinPool의 모든 스레드를 거의 공정하게 분할한다. 각각의 스레드는 자신에게 할당된 이중 연결 리스트를
참조하면서 작업이 끝날때마다 큐의 헤드에서 다른 태스크를 가져와서 작업을 처리한다. 태스크의
크기를 작게 나누어야 작업자 스레드 간의 작업부하를 비슷한 수준으로 유지할 수 있다.

## Spliterator 인터페이스
자바 8에서는 Spliterator인터페이스를 제공. Spliterator 인터페이스는 분할할 수 있는 반복자라는 의미.

```java
    public interface Spliterator<T> {
        boolean tryAdvance(Consumer<? super T> action);
        Spliterator<T> trySplit();
        long estimateSize();
        int characteristics();
    }
```

T는 Spliterator에서 탐색하는 요소의 형식. 
1)tryAdvance 메서드는 요소를 하나씩 순차적으로 소비하면서 탐색해야 할 요소가 남아있으면 참을 반환.
2)trySplit 메서드는 Spliterator의 일부 요소(자신이 반환한 요소)를 분할해서 두 번째 Spliterator를 생성하는 메서드.
3) estimateSize는 탐색해야할 요소 수 제공

### 분할 과정
trySplit를 호출하며 null이 될 때까지 반복. null은 더 이상 자료구조를 분할할 수 없음을 의미.

## 커스텀 Spliterator 구현하기
```
public class WordCounter {
    private final int counter;
    private final boolean lastSpace;

    public WordCounter(int counter, boolean lastSpace) {
        this.counter = counter;
        this.lastSpace = lastSpace;
    }

    public WordCounter accumulate(Character c) {
        if(Character.isWhitespace(c)) {
            return lastSpace ? this : new WordCounter(counter, true);
        } else {
            return lastSpace ? new WordCounter(counter+1, false) : this;
        }
    }

    public WordCounter combine(WordCounter wordCounter) {
        return new WordCounter(counter + wordCounter.counter, wordCounter.lastSpace);
    }

    public int getCounter() {
        return counter;
    }

    static int countWords(Stream<Character> stream) {
        WordCounter wordCounter = stream.reduce(new WordCounter(0, true),
                                                WordCounter::accumulate,
                                                WordCounter::combine);
        return wordCounter.counter;
    }

    public static void main(String[] args) {
        final String SENTENCE = "    public interface Spliterator<T> {  " +
                "        boolean tryAdvance(Consumer<? super T> action)   " +
                "        Spliterator<T> trySplit()     " +
                "        long estimateSize()       " +
                "        int characteristics()      " +
                "    }";
        Stream<Character> stream = IntStream.range(0, SENTENCE.length())
                .mapToObj(SENTENCE::charAt);
        System.out.println(countWords(stream));
    }
}
```