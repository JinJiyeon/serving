#  [92_Throughput](./92_Throughput)

#### 기본 정보

- 목적 : http 요청에 응답하는 throughput을 늘리는 방안 (WAS, Actor, proxy)에 대해 알아보자

- 기간 : 2021/11 ~ 2021/12

- Ref :

  - [블로그](https://velog.io/@dspf3527/JavaScript에서의-비동기)

  - [우테코](https://youtu.be/YxwYhenZ3BE)



## 빠르게 응답하기

- process 보다 thread를 활용한다 (WAS)
- thread를 미리 띄워놓는다 (WAS)
- 비동기로 응답하도록 한다
- cache를 활용한다



## WAS

- WAS와 웹서버를 비교하자면
  응답을 할 때에 html, css와 같이 정적인 컨텐츠가 있고
  동적으로 계산해줘야 하는 것들이 있다.
  Apache 등의 웹서버는 정적인 컨텐츠를 제공하는 데에,
  Tomcat 등의 WAS는 동적인 컨텐츠를 제공하는 데에 특화되어 있다.



#### Tomcat

- Tomcat은 thread를 미리 띄워 놓고, 요청이 들어오면 응답해준다.
  - 동적인 컨텐츠를 제공하려면 process나 thread를 띄워서 연산하는 작업이 필요하다.
  - process보다 thread를 띄우는 비용이 더 저렴하고 빠르다.
  - thread를 미리 띄워 놓으면, 그 비용마저 아낄 수 있다.

- Tomcat은 가용한 thread 개수 이상의 요청이 들어오면, 큐에 쌓아둔다.



#### Actor

- Actor 모델도 thread를 미리 띄워 놓고, 요청이 들어오면 응답해준다.
- Actor 모델은 가용한 thread 개수 이상의 요청이 들어오면, 각 액터의 메시지 큐에 쌓아둔다.
- Actor는 비동기로 연산하기 때문에, 많은 요청을 더 빠르게 처리할 수 있다.



## Async

- [블로그](https://velog.io/@dspf3527/JavaScript에서의-비동기)

- 비동기란 특정 로직의 실행이 끝날 때까지 기다려주지 않고 나머지 코드를 먼저 실행하는 것이다.
- js나 actor가 비동기 프로그래밍으로 유명하다. 하지만 모든 코드가 비동기로 돌아가는 것은 아니고. 특정 코드는 비동기로 실행된다. 예를 들어 js에는 setTimeOut, axios 등의 함수가 비동기로 실행된다.



## Cache

- [우테코](https://youtu.be/YxwYhenZ3BE)
- proxy 서버에 자주 사용되는 정보를 캐시해둠으로써 속도를 향상시킨다.



#### proxy

- 클라이언트와 백 사이에 있는 서버이다.
- 클라이언트 앞에 있을 수도, 백 앞에 있을 수도 있다.
- 캐시를 사용해서 응답 속도를 높이고, 
  IP를 숨겨서 개인정보를 보호하고 서버의 보안성을 높일 수 있으며,
  서버의 트래픽을 분산하는 데에 활용할 수 있다.