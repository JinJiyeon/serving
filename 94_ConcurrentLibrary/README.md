# 94_ConcurrentLibrary

#### 기본정보

- 목적 : 멀티스레드 프로그램을 구현하는 데 필요한 구현체와 내부 구조를 분석해보자.

- 기간 : 2021/11/14 ~ 
- Ref : 
  - [Java 8 Concurrency Tutorial](https://winterbe.com/posts/2015/04/07/java8-concurrency-tutorial-thread-executor-examples/)
  - [Concurrent Hash map 내부구조](https://devlog-wjdrbs96.tistory.com/269)



## 개념

- synchronized 
  다른 스레드가 접근하지 못하도록 lock을 건다
- Atomic (compare and swap)
  기대값과 같을 때에만 조작을 해라.
- volatile
  변수를 캐시가 아니라 메모리에서 읽어오라. 



## Concurrent Hash Map

- hashmap이 모든 메서드에 대해 synchronized가 걸려있는 반면,
  concurrent hash map은 put 메서드의 일부에 한해서만 synchronized가 걸려 있다.
- lock이 최소한으로 걸려 있기 때문에 속도가 빠른 반면,
  get하는 사이 put이 되어 데이터가 달라질 위험이 있다.



## Atomic Long

- cnt = cnt + 1 처럼 일반적인 연산은 data race가 발생할 수 있다.
- thread safe한 연산을 하기 위해 lock을 걸 수도 있겠으나. lock은 cpu에 부담이 된다. 
- cpu 차원의 lock이 아닌 논리로 연산을 할 수 있다. 기댓값과 같을 때에만 조작을 하는 것이다.