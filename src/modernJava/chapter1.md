## 1. 자바8의 변경사항

1) 스트림 API

2) 메서드에 코드를 전달

3) 인터페이스의 디폴트 메서드

## 1.2.2 스트림 처리

스트림이란 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임. 자바 8에는 java.util.stream 패키지에 스트림 API가 추가 되었다. 
스트림 패키지에 정의된 Stream<T>는 T 형식으로 구성된 일련의 항목을 의미한다. 
스트림 API의 핵심은 기존에는 한 번에 한 항목을 처리했지만 자바 8에서는 고수준으로 추상화해서 일련의 스트림으로 만들어 처리할 수 있다는 것이다.

## 1.2.3 동작 파라미터화로 메서드에 코드 전달하기

자바 8에서는 메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공.

## 1.2.4 병렬성과 공유 가변 데이터

스트림 메서드로 전달하는 코드는 다른 코드와 동시에 실행하더라도 안전하게 실행되어야 한다. 
이를 위해선 공유된 가변데이터에 접근하지 않아야 한다. (값이 변하지 않아야 한다는 말) 이러한 함수를 순수함수, 부작용없는 함수, 상태없는 함수라고 부른다. 

## 1.3 자바 함수

자바 8에서는 함수를 새로운 값의 형식으로 추가했다. 이는 멀티코어에서 병렬 프로그래밍을 활용할 수 있는 스트림과 연계될 수 있도록 함수를 만들었기 때문이다. 
프로그래밍 언어의 핵심은 값을 바꾸는 것이다. 이 값을 일급 값이라고 부른다. 자바 8 에서는 메서드를 런타임에 전달할 수 있도록 바꾸었다. 즉 메서드를 일급시민으로 바꾸었다.

## 1.3.1 메서드와 람다를 일급 시민으로

메서드를 값으로 취급할 수 있도록 바꾸었다. 즉 자바8에서 메서드는 일급값이다.

## 1.3.2 익명함수

자바 8에서는 메서드를 일급값으로 취급할 뿐 아니라 람다를 포함하여 함수도 값으로 취급할 수 있다.

## 1.3.3 메서드 전달에서 람다로

메서드를 값으로 전달하는 것은 매우 유용한 기능이나 한두 번만 사용할 메서드를 매번 정의하는 것은 귀찮은 일이다. 
이 문제를 해결하기 위해 익명함수 또는 람다라는 새로운 개념을 도입했다. 즉, 한 번만 사용할 메서드는 따로 정의를 구현할 필요가 없다. 
컬렉션과 스트림 간에 변환할 수 있는 메서드(map, reduce)도 제공된다.

## 1.4 스트림

스트림 API에서는 라이브러리 내부에서 모든 데이터가 처리된다. 이를 내부반복이라 한다.

## 1.4.1 멀티스레딩은 어렵다

자바 8은 스트림 API로 컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드문제 그리고 멀티코어 활용 어려움이라는 두 가지 문제를 모두 해결했다.
컬렉션은 어떻게 데이터를 저장하고 접근할지 중심 / 스트림은 데이터에 어떤 계산을 할 것인지 묘사하는 것에 중심. 
스트림은 스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공한다는 것이 핵심이다. 
컬렉션을 필터링할 수 있는 가장 빠른 방법은 컬렉션을 스트림으로 바꾸고, 병렬로 처리한 다음에 리스트로 다시 복원하는 것이다. 

## 1.4.2 자바의 병렬성과 공유되지 않은 가변 상태

자바 8은 병렬성과 관련해 두 가지 편의를 제공한다. 우선 라이브러리에서 분할 처리 한다. 즉, 큰 스트림을 병렬로 처리할 수 있도록 작은 스트림으로 분할한다. 
함수형 프로그래밍에서 함수형이란 함수를 일급값으로 사용한다 의미에 더불어 프로그램이 실행되는 동안 컴포넌트 간에 상호작용이 일어나지 않는다는 의미도 있다.