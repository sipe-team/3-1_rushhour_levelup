### 11/5 spring-ai에 대해서 알아보았습니다.

이미 서비스에서 외부 ai 서비스를 사용하고 있는데, 나름 구조화를 하여 사용하고 있지만 제한된 점이 많이 있는데요. 그 부분에 대해서 어느정도 해결할 수 있는 spring-ai를 확인해보았습니다.

특징 몇 가지만 공유드리겠습니다.

1. 출력을 Pojo로 손쉽게 매핑할 수 있습니다.
2. 사용량, limit 등의 정보를 쉽게 확인할 수 있게 되었습니다.
3. 요청에 필요한 정보들이 구조화되어서 손쉽게 사용도 가능하고, 통일된 방식으로 요청도 가능하다는 이점도 있는 것 같아요!

참고 링크 공유드립니다.

https://spring.io/projects/spring-ai

### 11/6 리액티브 프로그래밍

리액티브 프로그래밍(Reactive Programming)은 **비동기적인 프로세싱 파이프라인**을 만들기 위한 목적으로 개발되었고, 함수형 프로그래밍과 유사한 "선언적 코드"를 사용한 새로운 패러다임입니다. 

Reactive Streams Interface 명세

Reactive Stream의 시퀀스는 Publisher가 데이터를 생산해냅니다.

하지만, 기본적으로 Subscriber가 등록되기(구독하기)전까지 아무런 일도 하지 않습니다.

즉, Publisher는 Subscriber가 등록되는 시점부터 데이터를 push 하게 됩니다.

그림을 자세히 보시면, Subscriber는 Publisher에게 피드백을 줍니다.

간단하게 설명하자면, 이 피드백은 혼잡을 제어할 수 있는 도구로 "모든 이벤트를 다 push하지 말고, N개만 보내줘"와 같은 요구에 해당한다고 합니다. 이로써 Subscriber에 데이터가 지나치게 쌓여서 감당할 수 없는 현상을 제거하기 위해 이 피드백은 굉장히 중요한 역할을 합니다.

이러한 피드백을 관장하는 요소가 바로 Back-Pressure 

Reactor는 어떤 데이터가 대해서 각각의 단계를 적용하기 위한 처리에 대한 내용을 기술하기 위해 서로 묶여있는, **오퍼레이터**라는 개념을 추가했습니다.

오퍼레이터를 적용하면, 중간 퍼블리셔(upstream에 대한 subscriber, downstream에 대한 publisher)를 리턴하게 됩니다.

```
Copy
Flux<String> flux = Flux.just("A");

flux.map(s -> "foo" + s);// 새로운 Publisher 생성, 걔를 구독해야 fooA가 찍힘.
flux.subscribe(System.out::println);// 얘는 1번 라인의 flux를 구독, 따라서 "A"가 나온다.
Flux<String> flux = Flux.just("A");

Flux<String> flux2 = flux.map(s -> "foo" + s);

flux2.subscribe(System.out::println);// fooA

```

두 번째 줄의 코드를 보면, flux.map()을 통해서 새로운 중간 publisher가 생성되게 됩니다.

하지만 셋째 줄에서 첫째 줄의 flux를 subscribe 했기 때문에 "fooA"가 아닌 "A"가 나오게 되죠.

그 아래의 코드대로 사용하면, 기존의 의도대로 코드가 나오게 됩니다. 물론 flux2를 생성하지 않고 이어서 선언할 수도 있습니다.

### 11/7 **반응형 스트림(Reactive Stream) 수행과정**

**반응형 스트림 처리 과정**

https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdT2vwe%2FbtsqBwUlPkC%2FjfmERVJtjeqmWbi125Aag0%2Fimg.png

1.**[subscribe]**

Subscriber를 Publisher에 ‘등록’하고 데이터 스트림을 ‘수신할 준비’가 되었음을 Publisher에게 알립니다.

2.**[onSubscribe]**

Publisher가 Subscriber에게 데이터 스트림을 ‘전송하기 시작하기 전에 호출’됩니다. 이 메서드를 통해 Subscriber는 Subscription 객체를 받아들여 데이터의 양을 제어할 수 있습니다.

3.**[request(n)/cancel]**

request(n) 메서드는 Publisher에게 n개의 데이터를 요청하고, cancel 메서드는 데이터 스트림을 취소합니다.

4.**[onNext(data)]**

Publisher가 ‘생성한 데이터’를 Subscriber에게 전달합니다. 이 메서드는 데이터가 전송될 때마다 호출됩니다.

5.**[onComplete/onError]**

Publisher가 모든 데이터를 전송하고, 더 이상 데이터가 없을 때 호출됩니다.

- onComplete 메서드는 모든 데이터가 성공적으로 전송되었음을 나타냅니다.
- onError 메서드는 데이터 전송 중 오류가 발생했음을 나타냅니다.

### 11/8  **Mono/Flux**

Mono

- **Reactor 라이브러리에서 제공하는 Reactive Streams의 Publisher 중 하나로 오직 ‘0개 또는 하나의 데이터항목 생성’하고 이 결과가 생성되고 나면 스트림이 종료되면 결과 생성을 종료합니다.**
- Mono를 사용하여 비동기적으로 결과를 반환하면 해당 결과를 구독하는 클라이언트는 결과가 생성될 때까지 블로킹하지 않고 다른 작업을 수행할 수 있습니다.

| **메소드** | **설명** |
| --- | --- |
| **just()** | 주어진 데이터를 포함하는 Mono를 생성합니다. |
| **empty()** | 데이터가 없는 Mono를 생성합니다. |
| **error()** | 에러 상황을 나타내는 Mono를 생성합니다. |
| **fromCallable()** | Callable 객체를 이용해 Mono를 생성합니다. |
| **fromFuture()** | Future 객체를 이용해 Mono를 생성합니다. |
| **fromRunnable()** | Runnable 객체를 이용해 Mono를 생성합니다. |

```java
Mono.just("Hello, world!")
    .map(String::toUpperCase)
    .flatMap(s -> Mono.just("Mono: " + s))
    .subscribe(System.out::println);

```

Flux

- **Reactor 라이브러리에서 제공하는 Reactive Streams의 Publisher 중 하나로 Mono와 달리 ‘여러 개의 데이터 항목’를 생성하고 스트림이 종료되면 결과 생성을 종료합니다.**
- 비동기 작업을 수행하면 작업이 완료될 때까지 블로킹하지 않고 다른 작업을 수행할 수 있습니다.
- Spring WebFlux에서 Flux를 사용하여 HTTP 요청을 처리하는 경우, 요청을 수신한 즉시 해당 요청을 처리하고 결과를 생성하는 대신 결과 생성이 완료될 때까지 다른 요청을 처리할 수 있습니다.

| **메소드** | **설명** |
| --- | --- |
| **just()** | 주어진 데이터를 포함하는 Flux를 생성합니다. |
| **fromIterable()** | Iterable 객체를 이용해 Flux를 생성합니다. |
| **fromArray()** | 배열을 이용해 Flux를 생성합니다. |
| **fromStream()** | Stream을 이용해 Flux를 생성합니다. |
| **empty()** | 데이터가 없는 Flux를 생성합니다. |
| **error()** | 에러 상황을 나타내는 Flux를 생성합니다. |
| **range()** | 지정된 범위의 정수를 포함하는 Flux를 생성합니다. |
| **interval()** | 일정 시간 간격으로 값을 생성하는 Flux를 생성합니다. |
| **merge()** | 여러 개의 Flux를 병합하여 하나의 Flux로 만듭니다. |
| **concat()** | 여러 개의 Flux를 순차적으로 이어붙여 하나의 Flux로 만듭니다. |
| **zip()** | 여러 개의 Flux를 조합하여 튜플 형태로 만듭니다. |
| **collectList()** | Flux에 포함된 모든 데이터를 리스트로 수집합니다. |
| **collectMap()** | Flux에 포함된 모든 데이터를 맵으로 수집합니다. |

```java
Flux.just("apple", "banana", "cherry")
    .map(String::toUpperCase)
    .flatMap(s -> Mono.just("Flux: " + s))
    .subscribe(System.out::println);

```
