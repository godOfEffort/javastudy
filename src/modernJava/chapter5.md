# 스트림 활용
## 5.2 스트림 슬라이싱
### 5.2.1 프레디케이트를 이용한 슬라이싱
자바 9는 takeWhile, dropWhile 메서드 지원
1) takeWhile
** 리스트가 정렬**되어 있다는 가정하에 조건에 맞지 않을때 반복 작업을 중단하고 조건에 맞는 데이터 반환
``` 
   List<Dish> filteredMenu = menu.stream().
                             takeWhile(dish -> dishg.getCalories() < 320)
                            .collect(toList());
    // 위 예제는 320을 넘어가는 요소를 발견하면 작업 중단하고 320 이전의 데이터 반환
```
2) dropWhile 
** 리스트가 정렬**되어 있다는 가정하에 조건에 처음으로 맞지 않을때 반복 중단하고 
거짓이되는 지점까지 발견된 요소를 버린다. 즉, 거짓이 되면 그 지점에서 작업을 중단하고 
남은 모든 요소 반환
``` 
   List<Dish> filteredMenu = menu.stream().
                             dropWhile(dish -> dishg.getCalories() < 320)
                            .collect(toList());
   // 위 예제는 320을 넘어가는 요소를 발견하면(처음으로 거짓이 되는 지점까지 발견된 요소는 버린다) 작업 중단하고 남은 모든 요소를 반환
```
filter 연산을 이용할 경우 전체 스트림을 반복하기 때문에 데이터가 대량일 경우 속도면에서 큰 차이를 보일 수 있다.

### 5.2.2 스트림 축소
스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 limit(n) 메서드를 지원.
** 스트림이 정렬되어 있으면** 최대 요소 n개를 반환할 수 있다. 정렬되지 않은 스트림에도 limit을 사용할 수 있으나
소스가 정렬되어 있지 않았다면 limit의 결과도 정렬되지 않은 상태로 반환된다.

### 5.2.3 요소 건너뛰기
스트림은 **처음 n개**요소를 제외한 스트림을 반환하는 skip(n) 메서드를 지원.

### 5.3 매핑
map과 flatMap 메서드는 특정 데이터를 선택하는 기능 제공
1) 스트림은 함수를 인수로 받는 map 메서드를 지원. 인수로 제공된 함수는 각 요소에 적용되며 **함수를 적용한 결과가
새로운 요소로 매핑된다.**
map 메서드의 출력 스트림은 **Stream<T>** 형식을 갖는다.

### 5.4 요소검색
findAny 메서드는 현재 스트림에서 임의의 요소를 반환한다. 스트림 파이프라인은 내부적으로 단일 과정으로 
실행할 수 있도록 최적화된다. 즉, 쇼트서킷을 이용해서 결과를 찾는 즉시 실행을 종료한다.
```
    Optional<Dish> dish = menu.stream()
                          .filter(Dish::isVegetarian)
                          .findAny(); 
```
Optional
1) isPresent() - 값을 포함하면 true, 포함하지 않으면 false 반환
2) ifPresent(Consumer<T> block) -> 값이 있으면 block 수행
3) T get()은 값이 존재하면 값 반환, 없으면 NoSuchElementException 일으킨다.
4) T orElse(T other)는 값이 있으면 값을 바노한하고, 값이 없으면 기본값 반환

findFirst() 메서드도 있는데 findFirst와 findAny메서드는 병렬성을 위해서 필요하다. 병렬 실행에서는
첫번째 요소를 찾기 어렵다. 따라서 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다.

** 참고 : 최종연산**
1. forEach: 스트림의 각 요소에 대해 주어진 동작을 수행합니다.
2. toArray: 스트림의 요소를 배열로 변환합니다.
3. reduce: 스트림의 모든 요소를 이항 연산자를 사용하여 결합하거나, 초기값과 이항 연산자를 사용하여 축소합니다.
4. collect: 스트림의 요소를 수집하여 컬렉션(List, Set 등)이나 맵으로 변환합니다.
5. min, max: 스트림의 최소값(min) 또는 최대값(max)을 반환합니다.
6. count: 스트림의 요소 개수를 반환합니다.
7. anyMatch, allMatch, noneMatch: 스트림의 요소 중 조건을 만족하는 요소가 하나라도 있는지(anyMatch), 모든 요소가 조건을 만족하는지(allMatch), 모든 요소가 조건을 만족하지 않는지(noneMatch)를 판별합니다.
8. findFirst, findAny: 스트림의 첫 번째 요소(findFirst) 또는 임의의 요소(findAny)를 반환합니다.

### 5.5 리듀싱
모든 스트림 요소를 처리해서 값으로 도출하는 것은 리듀싱 연산이라고 한다.
```
int sum = 0;
for (int x : numbers) {
  sum += x;
}
// 리스트에서 하나의 숫자가 남을때까지 reduce 과정을 반복한다. 코드에는 파라미터를 두 개 사용 했다.
    sum 변수의 조기값 0, 리스트의 모든 요소를 조합하는 연산(+)
```

```
int sum = numbers.stream().reduce(0, (a,b) -> a + b);
// 초기값0
// BinaryOperator<T> 사용

//Integer 클래스의 정적 sum 메서드 제공
int sum = numbers.stream().reduce(0, Integer::sum);
```
1) 최댓값과 최솟값
최댓값, 최솟값을 찾을때도 reduce 사용 가능
```
    Optional<Integer> min = numbers.stream().reduce(Integer::min);
    Optional<Integer> max = numbers.stream().reduce(Integer::max);

```
2) 스트림 연산 : 상태 없음과 상태 있음
map, filter 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다.
따라서 이들은 사용자가 제공한 람다나 메서드 참조가 내부적인 가변 상태를 갖지 않는다는 가정하에 
내부 상태를 갖지 않는 연산이다.
 하지만 reduce, sum, max 같은 연산은 결과를 누적할 내부 상태가 필요하다. max값을 비교하는 경우
예제의 내부 상태는 작은 값이다. 스트림에서 처리하는 요소 수와 관계없이 내부 상태 크기는 bounded 되어 있다.
 반면 sorted나 distinct 같은 연산은 과거의 이력을 알고 있어야 정렬이나 중복을 제거할 수 있다. 예를 들어 어떤 요소를 
출력 스트림으로 추가하려면 모든 요소가 버퍼에 추가되어 있어야 한다. 이러한 연산을 stateful operation이라 한다.


### 5.6 숫자형 스트림

```
int caloreis = menu.stream()
                .map(Dish::getCalories)
                .reduce(0, Integer::sum);
```
위 코드에는 박싱 비용이 있다. 내부적으로 합계를 계산하기 전에 Integer를 기본형으로 언박싱 해야 한다.

1) 기본형 특화 스트림
IntStream, LongStream, DoubleStream이 있다. 특화 스트림은 오로지 박싱 과정에서 일어나는 효율성과 관련있다.


숫자 스트림으로 매핑
스트림을 특화 스트림으로 변환할 때는 mapToInt, mapToDouble, mapToLong 세 가지 메서드를 가장 많이 사용. 이들 메서드는
map과 정확히 같은 기능을 수행하지만 Stream<T> 대신 특화된 스트림을 반환.

```
int caloreis = menu.stream()
                .mapToInt(Dish:getCalories()
                .sum();
```
mapToInt 메서드는 각 요리에서 모든 칼로리를 추출한 다음에 IntStream(Stream<Integer>가 아님)을 반환한다. 따라서 IntStream 인터페이스에서
제공하는 sum 메서드를 사용해서 칼로리를 계산할 수 있다. 스트림이 비어있으면 sum은 기본값 0을 반환한다.
