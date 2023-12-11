## Observer vs Listener vs Callback

### Callback
argument 형태로 전달되는 코드 (함수)
- caller 가 callee 의 return 이후에 실행 될 코드를 argument 로 전달하는 패턴
- callee 가 callback 의 실행 시점을 **지연**시킬 수 있다

### Listener
Event 의 발생을 사전에 등록된 Listener 에 통지하는 패턴

### Observer
Subject 의 상채변경(Event)을 사전에 등록된 Observer 에 통지하는 패턴

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