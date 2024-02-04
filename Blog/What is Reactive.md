## Non-Blocking I/O
### Sync vs Async
2개의 인스럭션(코드)가 있을때 실행을 대기(wait)하면 Sync, 안하면 Async. 시점(timing)의 동기화 여부를 판단. thread, process, kernel 등 모든 실행흐름을 포함.

Single thread에서, 코드의 실행(함수의 실행과 호출)은 process call stack에 쌓이고 caller는 callee의 종료(리턴)을 기다림. Spring @EventListener도 동기호출 (sync)

Multi thread에서, 각 thread는 서로의 실행을 간섭하지 않음. 독립적인 실행흐름(excution context, register + call stack)을 가짐(async)

Multi thread에서, lock/conditional variable을 사용하면 각 thread는 코드 실행 순서 기다리게 됨. kernel 인터럽트 처리순서. java의 syncronized (sync)

I/O는 각 device processor에 의해 cpu와 독립적인 실행흐름(register 상태)를 가지므로 기본적으로 Async 성격을 가짐(async)
### Blocking vs Non-Blocking (I/O)
Process가 I/O를 위한 Kernel(시스템 콜) 호출 시, I/O 대기를 위한 Schedule(Context Switching) 발생 여부(wait)

전통인 I/O는 Blocking. process call stack -> kernel call stack 흐름
- select, poll (sync blocking. multiflexing)
- epoll(sync non-blocking)

posix aio의 경우 별도의 callback 함수를 별도 thread에서 호출 함 (async non-blocking)

실제 I/O는 non-blocking으로 수행되지만 caller가 I/O를 동기 방식으로 기다릴 수 있음. WebClient를 기다리는 Feature.get()
## Observer vs Listener vs Callback

### Callback
~~argument 형태로 전달되는 코드 (함수)~~
> Spring의 `InitializingBean.afterPropertiesSet()` 인터페이스는 caller의 명시적인 호출과 인자의 형태로 코드가 전달되지 않음
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