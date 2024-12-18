# 스프링으로 시작하는 리액티브 프로그래밍

## 리액티브 시스템과 리액티브 프로그래밍
- 리액티브 시스템
  - 반응을 잘하는 시스템
  - 빠른 응답성을 바탕으로 유지보수와 확장이 용이한 시스템
- 리액티브 프로그래밍
  - 리액티브 시스템을 구축하는데 필요한 프로그래밍 모델
  - 비동기 메시지 통신을 기반으로 함(Non-Blocking I/O)
  - 코드가 간결해지며 가독성이 올라감
- 리액티브 프로그래밍 코드 구성
  - Publisher: 데이터를 제공하는 역할(발행자, 생산자)
  - Subscriber: 제공한 데이터를 전달받아서 사용하는 주체(구독자, 소비자)
  - Data Source: Publisher의 입력으로 들어오는 데이터를 대표하는 용어, Data Stream이라고도 표현함
  - Operator: Publisher에서 Subscriber로 데이터가 전달될 때 애플리케이션의 요구사항에 맞게 적절한 가공 처리를 하는 역할, 리액티브 프로그래밍은 Operator로 시작해서 Operator로 끝난다고 볼 수 있음

## 리액티브 스트림즈
- 리액티브 스트림즈
  - 데이터 스트림을 Non-Blocking이면서 비동기적인 방식으로 처리하기 위한 리액티브 라이브러리 표준사양
  - RxJava, Reactor, Akka Streams, Java 9 Flow API 등이 있음
- 리액티브 스트림즈 구성요소
  - Publisher: 데이터를 생성하고 통지(발행, 게시, 방출)하는 역할
  - Subscriber: 구독한 Publisher로부터 통지(발행, 게시, 방출)된 데이터를 전달받아서 처리하는 역할
  - Subscription: Publisher에 요청할 데이터의 개수를 지정하고, 데이터의 구독을 취소하는 역할
  - Processor: Publisher와 Subscriber의 기능을 모두 가지고있음, Subscriber로서 다른 Publisher를 구독할 수 있고, Publisher로서 다른 Subscriber가 구독할 수 있음
  - Publisher와 Subscriber의 동작 과정
    - 1 먼저 Subscriber는 전달받을 데이터를 구독함(subscribe)
    - 2 다음으로 Publisher는 데이터를 통지할 준비가 되었음을 Subscriber에 알림)onSubscribe
    - 3 Publisher가 데이터를 통지할 준비가 되었다는 알림을 받은 Subscriber는 전달받기를 원하는 데이터의 개수를 Publisher에게 요청함(Subscription.request)
      - Publisher가 통지하는 속도보다 Subscriber가 처리하는 속도보다 빠르면 처리를 기다리는 데이터는 쌓이게되므로 시스템에 부하가 커지게되어 Subscription.request를 통해 데이터 개수를 제어함
    - 4 다음으로 Publisher는 Subscriber로부터 요청받은 만큼의 데이터를 통지함(onNext)
    - 5 Publisher와 Subscriber 간에 데이터 통지, 데이터 수신, 데이터 요청의 과정을 반복하다가 Publisher가 모든 데이터를 통지하게 되면 마지막으로 데이터 전송이 완료되었음을 Subscriber에게 알림(onComplete)
    - 5-1 Publisher가 데이터를 처리하는 과정에서 에러가 발생하면 에러가 발생했음을 Subscriber에게 알림(onError)
  - 리액트 스트림즈 컴포넌트의 인터페이스 코드는 Publisher가 subscribe 메서드를 통해 Susbscriber를 구독하게 되어있는데 이는 Kafka의 Pub/Sub 개념과 조금 다르다. Kafka는 중간에 메시지 브로커를 통해 토픽이라는 것이 존재하고 Publisher와 Subscriber가 해당 토픽을 바라보며 느슨한 결합이 되어있다. 리액트 스트림즈 또한 개념상으로는 Subscriber가 구독하는 것이 맞는데 실제 코드는 위와 같이 된다.
  - Signal: Publisher와 Subscriber간에 주고받는 상호작용(onSubscribe, onNext, onComplete, onError, request, cancel)
  - Demand: Subscriber가 Publisher에게 요청하는 데이터, Subscriber가 요청한 아직 전송되지 않은 데이터
  - Emit: Publisher가 Subscriber에 데이터를 전달하는 것(통지, 발행, 게시, 방출)
  - Upstream/Downstream: 데이터 흐름에서 상위에 있냐 하위에 있냐에 따라 상대적으로 달라짐
  - Sequence: Publisher가 emit하는 데이터의 연속적인 흐름을 정의해 놓는 것, Operator 체인 형태로 정의
  - Operator: just, filter, map 같은 메서드
  - Source, Original: 최초의 생성된 무언가(Data Source, Source Publisher, Source Flux)

## Blocking I/O와 Non-Blocking I/O
- Blocking I/O
  - 요청 스레드가 차단되어 작업 스레드가 처리를 끝내고 응답을 반환하기 전까지 대기함
  - 하나의 스레드가 I/O에 의해서 차단되어 대기하는 것
  - 멀티 스레딩 기법을 사용하여 보완할 수 있지만 컨텍스트 스위칭으로 인한 스레드 전환 비용이 발생됨
  - 과다한 메모리 사용으로 오버헤드가 발생할 수 있음
  - 스레드 풀에서 응답 지연이 발생할 수 있음
  - Spring MVC
- Non-Blocking I/O
  - 작업 스레드의 종료 여부와 관계없이 요청한 스레드는 차단되지 않음
  - Blocking I/O 방식보다 더 적은 스레드를 사용하기 때문에 멀티스레딩 기법을 사용할 때 발생한 문제점들이 생기지 않음
  - CPU 대기 시간 및 사용량에 있어서 효율적임
  - CPU를 많이 사용하는 작업이 포함된 경우에는 성능에 악영향을 줌
  - 사용자의 요청에서 응답까지의 전체 과정에서 Blocking I/O 요소가 포함된 경우에 이점을 발휘하기 힘듬
  - Spring Webflux