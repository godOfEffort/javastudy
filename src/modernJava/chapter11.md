# Optional
## Optional 적용패턴
### Optional 객체 만들기
1) 빈 Optional
```java
   Optional<Car> optCar = Optional.empty();
```
2) null이 아닌 값으로 Optional 만들기
정적 팩토리 메서드 Optional.of로 null이 아닌 값을 포함하는 Optional을 만들 수 있다.
```java
   Optional<Car> optCar = Optional.of(car);
```
이제 car가 null이라면 즉시 NullPointerException이 발생한다.

3) null 값으로 Optional 만들기
정적 팩토리 메서드 Optional.of로 null이 아닌 값을 포함하는 Optional을 만들 수 있다.
```java
   Optional<Car> optCar = Optional.ofNullable(car);
```
car가 null이면 빈 Optional 객체가 반환된다.

### 맵으로 Optional의 값을 추출하고 변환하기
```java
   String name = null;
   if(insurance != null) {
        name = insurance.getName();
   }
```
이런 유형의 패턴에 사용할 수 있도록 Optional에서 map 메서드 지원
```java
   Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
   Optional<String> name = optInsurance.map(Insurance::getName);   
```
스트림의 map은 스트림의 각 요소에 제공된 함수를 적용하는 연산이다. Optional 객체를
최대 요소의 개수가 한 개 이하인 데이터 컬렉션으로 생각. Optional이 값을 포함하면
map의 인수로 제공된 함수가 값을 바꾼다. Optional이 비어 있으면 아무 일도 일어나지 않는다.

### flatMap으로 Optional 객체 연결
```
   Optional<Person> optPerson = Optional.of(person);
   Optional<String> name =
        optPerson.map(Person::getCar)
                 .map(Car::getInsurance)
                 .map(Insurance::getName); 
```
위 코드는 컴파일 되지 않는데 getCar의 결과가 Optional<Optional<Car>> 형식의 객체를 반환하기 때문이다.
스트림의 flatMap은 함수를 인수로 받아서 다른 스트림을 반환하는 메서드. 보통 인수로 받은 함수를 
스트림의 각 요소에 적용하면 스트림의 스트림이 만들어진다. 하지만 flatMap은 인수로 받은 함수를
적용해서 생성된 각각의 스트림에서 콘텐츠만 남긴다.

```java
   public String getCarInsuracneName(Optional<Person> person) {
        return perosn.flatMap(Person::getCar)
                     .flatMap(Car::getInsurance)
                     .map(Insurance::getName)
                     .orElse("Unkonwn");
   }
```

### 도메인 모델에 Optional을 사용했을 때 데이터를 직렬화할 수 없는 이유
Optional의 용도는 선택형 반환값을 지원하는 것이다.
Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 Serializable 인터페이스를 구현하지 않는다.
직렬화 모델이 필요하다면 Optional 값을 반환받을 수 있는 메서드를 추가하는 방식이 좋다.
```java
   public class Person {
        private Car car;
        public Optional<Car> getCarAsOptional() {
            return Optional.ofNullable(car);
        }
   }
```

### Optional 스트림 조작
자바 9에서는 Optional을 포함하는 스트림을 쉽게 처리하도록 Optional에 stream() 메서드를 추가.
```java
   public Set<String> getCarInsuranceNames(List<Person> persons) {
        return persons.stream()
               .map(Person::getCar)
               .map(optCar -> optCar.flatMap(Car::getInsurance))
               .map(optIns -> optIns.map(Insurance::getName))
               .flatMap(Optional::stream)
               .collect(toSet()); 
   }
```
getCar() 메서드가 단순히 Car가 아니라 Optional<Car>를 반환하므로 사람이 자동차를 가지지 않을 수 있는 상황.
첫 번째 map에서 Stream<Optional<Car>>를 얻는다. 

### 디폴트 액션과 Optional 언랩
1) get() 
래핑된 값이 있으면 반환하고 없으면 NoSuchElementException을 발생시킨다.
2) orElse 
Optional이 값을 포함하지 않을 때 기본값을 제공
3) orElseGet(Supplier<? extends T> other)
Optional에 값이 없을 때에만 Supplier가 실행된다.
디폴트 메서드를 만드는데 시간이 걸리거나 Optional이 비어있을 때만 기본 값을 생성하고 싶다면 이 메서드를 사용한다.
4) orElseThrow(Supplier<? extends T> exceptionSupplier) 
Optional이 비어있을 때 
예외를 발생시킨다. get메서드와 비슷하지만 발생시킬 예외의 종류를 선택할 수 있다.
5) ifPresent(Consumer<? super T> consumer) 
값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다. 값이 없으면 아무 일도 일어나지 않는다.
6) ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)
이 메서드는 Optional이 비어있을 때 실행할 수 있는 Runnable을 인수로 받는다.

  
### 두 Optional 합치기

```java
    public Optional<Insurance> nullSafeFindCheapestInsuracne( Optional<Person> person, Optional<Car> car) {
        if(person.isPresent() && car.isPresent()) {
            return Optional.of(findCheapestInsurance(person.get(), car.get()))
        } else {
            return Optional.empty();
        }
    }
```
아래와 같이 리팩토링 가능하다.
```java
    public Optional<Insurance> nullSafeFindCheapestInsuracne(Optional<Person> person, Optional<Car> car) {
        return person.flatMap(p -> car.map(c -> findCheapestInsurance(p,c)));
    }
```

### Optional을 사용한 실용 예제
1) 잠재적으로 null이 될 수 있는 대상을 optional로 감싸기
```java
Object value = map.get("key");
// 아래와 같이 수정
Optional<Object> value = Optional.ofNullabe(map.get("key"));
```
2) 예외와 Optional 클래스
```java
    public static Optional<Integer> stringToInt(String s) {
        try {
            return Optional.of(Integer.parseInt(s));  //문자열을 정수로 변환할 수 있으면 정수로 변환된 값을 포함하는 Optional을 반환
        } catch(NumberFormatException e) {
            return Optional.empty();
        }
    }
```
3) 기본형 Optional을 사용하지 말아야 한다.
    * 스트림이 많은 요소를 가질때는 기본형 특화 스트림을 이용해서 성능을 향상시킬 수 있으나 Optional의 최대 요소 수는
한 개이므로 Optional에서는 기본형 특화 클래스로 성능개선이 불가능.
    * Optional 클래스의 유용한 메서드 map, flatMap, filter등도 지원하지 않는다.
    * 기본형 특화 Optional 생성 결과는 다른 일반 Optional과 혼용이 불가능하다.
