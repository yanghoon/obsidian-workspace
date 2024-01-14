## Observer vs Listener vs Callback

### Callback
~~argument 형태로 전달되는 코드 (함수)~~
Spring의 `InitializingBean.afterPropertiesSet()` 
- caller 가 callee 의 return 이후에 실행 될 코드를 argument 로 전달하는 패턴
- callee 가 callback 의 실행 시점을 **지연**시킬 수 있다
- 동기 (Comparator), 비동기 (Runnalble)

### Observer
Subject 의 변경을 사전에 등록된 Observer 에 통지하는 패턴
- Open Close 원칙

### Listener
Event 의 발생을 사전에 등록된 Listener 에 통지하는 패턴
- Event-Driven 구조에서 Callback + Observer 패턴을 사용한 구현
- Emitter(발생)과 Handler(처리)를 분리
- Callback 의 Argument Type 에 대한 설계  

## Reactive vs Reactor Pattern
### Reactor
Event Loop with Non-Blocking I/O (Event Driven)
- Avoid complex concurrency handling in multithreading
- Avoid context switchings and memory cache unefficient
- Use small threads, Pull Mode
### Reactive
Scalable Asyncronous Event Streams
- Pub/Sub(Event Driven), Streams (not a single)
- Use funtional style, backpressure
- More readable asyncronous codes
- Use small threads, Push Mode

https://colevelup.tistory.com/37