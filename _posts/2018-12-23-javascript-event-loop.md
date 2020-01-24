---
title: "Javascript 이벤트 루프"
excerpt: ""
categories: javascript
tags:
  - javascript event loop
  - javascript asynchronous 
last_modified_at: 2019-03-25
---

# Javascript 이벤트 루프
- 우선 Javscript 이벤트 루프를 이해하기 위해서 Javascript 런타임 환경 구성 요소들에 대한 이해가 필요
- **여기서 설명하는 이벤트루프는 웹 브라우저 상에서의 이벤트 루프에 대해서 설명함**
- **Node JS 환경에서의 이벤트 루프는 다름..**

## Javascript 런타임 환경
- Javascript 런타임 환경은 일반적으로 브라우저를 일컫는 말
![No Image](/assets/images/posts/20181223/javascript_runtime_enviornment.jpeg)
출처: [Rod Machen의 JS Event Loop](https://codenotcode.com/my-event-loop-beebef81cd46)
- 위 그림은 브라우저의 JS 런타임 환경에서 이벤트 루프를 설명하기 위해 필요한 것을 묘사


### Javascript Runtime Engine
- JS 엔진은 여러 종류가 있으며, 각 엔진은 인터프린터 or Just-in-time 컴파일러를 이용해 코드를 실행

- 엔진 내부에는 코드를 실행하기 위한 힙과 스택 영역이 존재
  * 힙 영역 : 다른 언어들과 마찬가지로 Object 들이 할당되는 영역으로 코드 실행중에 동적으로 할당되는 영역
  * 스택 영역 : 코드의 실행 블럭 쌓이는 영역

- JS가 단일 스레드라고 불리는데 이는 엔진에서 단일 호출 스택을 이용한다는 관점에서 말하는 것 
  * 실제로 WEB 브라우저 환경은 100% 단일 스레드가 아니라는 [stackoverflow 글](https://stackoverflow.com/questions/38705444/javascript-event-loop-where-do-web-apis-get-executed)
  * 엔진이 단일 호출 스택을 이용하므로 엔진이 수행할 수 있는 코드 블럭은 1번에 1개만을 수행가능
  * 멀티 스레드 환경에서는 각각의 스레드가 고유의 스택영역을 갖기 때문에 코드를 수행하므로 여러 코드를 수행가능
  * JS의 실행 환경에서는 실제로 멀티 스레드로 구동되며, 각각의 스레드들과 엔진이 소통하기 위해서 이벤트 루프를 이용

### Web APIs
- 해당 영역은 런타임 환경(브라우저)에서 제공해주는 라이브러리로 이는 순수 Javascript 에서 제공하는 것이 아니라 런타임 환경에서 제공됨
- 런타임 환경이 브라우저라면 브라우저 이벤트에 대한 Web API 들이 제공됨 
- 이 영역은 라이브러리 영역으로써 개발자가 제어할 수 없는 영역으로 Web API 에서 제공하는 인터페이스 만을 이용해서 프로그램을 수행할 수 있음
- 여기서 말하는 WEB API 는 하나의 규약이자 JS 언어로 만든 인터페이스 
- [MDN 의 WEB API 도큐](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Introduction)


### 이벤트 루프와 콜백 큐
- JS 런타임 환경에서 엔진과 Web API들을 포함한 여러 스레드들과 소통하기 위해서 이벤트 루프라는 방식을 채택함.
- JS 비동기에 대한 이해는 이벤트 루프에 대한 이해가 필요함 

- 예제를 통해서 이벤트 루프가 갖는 의미를 살펴보자 
```javascript
console.log('Hi');
setTimeout(function cb1() { 
    console.log('cb1');
}, 0);
console.log('Bye');
```
- 크롬에서 개발자 도구로 실행해본 결과 
![No Image](/assets/images/posts/20181223/javascript_event_loop_ex_ret.jpeg)

- cb1() 함수를 분명 0ms 로 지정하고 실행했음에도 'cb1' 보다 'Bye' 가 먼저 출력된다. 
  * 이것은 이벤트 루프의 수행과정과 관련이 있다.

- 위 코드의 실행 순서를 살펴보자!
1. 우선 console.log('Hi')가 콜 스택에 추가
2. console.log('Hi')가 실행 (콘솔에 Hi 가 출력되는 시점이다)
3. console.log('Hi')가 제거
4. setTimeout(function cb1(){ ... })이 콜 스택에 추가 
5. setTimeout(function cb1(){ ... })이 실행 
6. 브라우저 Web API에 cb1() 함수가 추가됨 (Web API 는 실행시간과 관련한 일종의 타이머를 생성함)
7. setTimeout(function cb1(){ ... })이 제거
8. console.log('Bye')가 콜 스택에 추가
9. console.log('Bye')가 실행 (콘솔에 Bye 가 출력되는 시점이다)
10. console.log('Bye')가 제거 
11. 그런 다음 Web API의 타이머가 완료되면(setTimeout 에 지정한 시간이 지나면) cb1()함수를 콜백큐에 추가
12. 이벤트 루프는 cb1()함수를 콜백 큐에서 콜 스택으로 이동
13. console.log('cb1') 함수가 추가
14. console.log('cb1') 함수가 실행 (콘솔에 cb1이 출력되는 시점이다)
15. console.log('cb1') 함수가 제거 


![No Image](/assets/images/posts/20181223/javascript_event_loop_ex.gif)

- 위에서 실행 순서를 보았듯이 이벤트 루프는 Web API 에 등록된 함수를 콜백 큐에 넣고 
- 콜 스택이 비었을 경우 콜백 큐에서 콜 스택으로 해당 함수를 이동시킴

- 여기서 추가적으로 setTimeout 함수에 대해 알아보자.
```javascript
setTimeout(function time(){
  console.log('timeout');
}, 1000);
```
- setTimeout 함수를 실행하면 Web API는 일종의 타이머를 생성하고 실행하여 해당 함수를 1초 뒤에 수행시켜준다.
- 하지만 이것은 정확히 1초 뒤에 실행한다는 의미가 아니라, 1초 뒤에는 해당 함수를 콜백 큐에 넣어주겠다는 의미이다. 
- 따라서 콜 백큐에 해당 다른 함수들이 있거나 콜 스택이 비어있지 않으면 time 함수는 2~3초 뒤에 수행될 수도 있음을 의미한다.


## 프로미스와 이벤트 루프

### 프로미스란 ? 
- 프로미스는 자바스크립트의 비동기 처리에서 발생하는 콜백 지옥(Callback Hell)을 개선하기 위해 시작된 라이브러리
- 프로미스는 promise.js 란 라이브러리 형태로 존재하다가 ES6(ES2015+) 에서 표준으로 채택되었음 
  * 기존에는 브라우저 별로 promise 의 대한 동작방식이 다른 문제가 있었으나 최근 브라우저에서는 모두 해결되었다고 함

**여기서는 프로미스의 사용법에 대해서 다루지는 않음**

### 프로미스와 이벤트 루프의 관계 
- 위에서 이벤트 루프에 대해 예제를 살펴 보았으나 실제로 이벤트 루프는 [HTML 도큐](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)에 정의되어 있음 
- HTML 도큐에는 여러 내용이 기술 되어 있으나 간략하게 정리한 [stackoverflow](https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)에 따르면
> 
if) call stack 이 비었을 때 (이 부분은 위에서 설명했음)  
- 1\. task 큐에서 가장 오래된 task 를 선택
- 2\. 해당 task 가 null 이면 (task 큐가 비었으면) 6번으로 jump 
- 3\. 현재 실행중인 task를 "task A"로 설정
- 4\. "task A" 를 실행 (콜백 함수를 실행하는 것을 의미함)
- 5\. 현재 실행중인 task 를 null 로 설정하고 "task A"를 제거 
- 6\. micro task queue 를 실행 
  * 6.1 micro task 큐에서 가장 오래된 task 를 선택
  * 6.2 해당 task 가 null 이면 (micro task 큐가 비었으면) 6.7로 jump
  * 6.3 현재 실행중인 task를 "task x"로 설정
  * 6.4 "task x"를 실행 (콜백 함수를 실행하는 것을 의미함)
  * 6.5 현재 실행중인 task 를 null 로 설정하고 "task x"를 제거 
  * 6.6 micro task 큐에 다음 실행될 task 가 존재하면 6.1로 jump
  * 6.7 micro task 큐 종료 
- 7\. 1번으로 jump

- 이벤트 루프의 처리 모델을 보면 task 큐에 task 를 실행하고 나면 micro task 큐를 검사해서 micro task 큐에 존재하는 모든 작업을 처리하는 것을 볼 수 있다.

- 예제를 보자 

```javascript
console.log('script start');

setTimeout(function timeoutCallback() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function promiseCallback1() {
  console.log('promise1');
}).then(function promiseCallback2() {
  console.log('promise2');
});
```
[소스 출처](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

- 위 코드의 출력 순서는 어떻게 될까?
![No Image](/assets/images/posts/20181223/promise1.jpg)

- 위 결과를 보면 setTimeout 보다 promise1 이 먼저 출력된다.
- 이것은 바로 이벤트 루프 에서 task (macro task)과 micro task 로 인해 발생하는 현상이다.
- [Jake 의 JS microtasks queue](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)에서 설명하듯이 프로미스는 micro task 로써 항상 일반 GUI 작업 or AJAX 와 같은 macro task 보다 먼저 수행함을 보장한다.

**코드의 실행순서**
> 
1. 스크립트가 실행되어 task queue 에 현재 실행 task 가 추가 
2. console.log('script start'); 코드가 실행되에 콘솔에 'script start' 가 출력됨
3. setTimeout(..) 이 실행되어 0초 뒤에 timer 에 의해서 timeoutCallback 이 task queue 에 등록
4. Promise 가 실행되어 최초의 Promise then 함수가 micro task 에 등록
5. 현재 실행 중인 task 를 제거하고 micro task queue 를 실행
6. micro task queue 에서 promiseCallback1 을 선택하여 수행하여 콘솔에 'promise1'이 출력
7. promiseCallback1 의 return 값이 없으므로 'undefined' 를 반환하고 다음 promise 콜백을 micro task queue 에 추가한다. (promiseCallback2 이 추가됨)
8. promiseCallback1 가 micro task queue 에서 제거
9. micro task queue 작업이 남아 있으므로 다시 promiseCallback2 를 선택
10. promiseCallback2 를 수행하여 콘솔에 'promise2'가 출력
11. promiseCallback2 의 return 값이 없으므로 'undefined' 를 반환하고 다음 promise 콜백이 없으므로 micro task queue 에 추가하지 않음
12. promiseCallback2 가 micro task queue 에서 제거
13. micro task queue 가 비었으므로 micro task queue 는 종료됨
14. task queue 에서 timeoutCallback 을 선택하여 수행되어 콘솔에 'setTimeout' 가 출력됨 

- 태스크의 종류들
  * task (macro task) : setTimeout, setInterval, setImmediate, requestAnimationFrame, I/O, UI 랜더링, etc
  * micro task : process.nextTick(nodejs custom micro task), Promises, Object.observe, MutationObserve

## TODO 리스트
- Javascript 엔진에 대한 조사
- Node JS 환경에서의 이벤트 루프
- [MDN WEB API 리스트](https://developer.mozilla.org/ko/docs/WebAPI)


## Reference 
- [TOAST Meetup 자바스크립트와 이벤트 루프](https://meetup.toast.com/posts/89)
- [Rod Machen의 JS Event Loop](https://codenotcode.com/my-event-loop-beebef81cd46)
- [Sunki Baek 의 JS 이벤트 루프와 async, await](https://engineering.huiseoul.com/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94%EA%B0%80-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84%EC%99%80-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%EC%9D%98-%EB%B6%80%EC%83%81-async-await%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%BD%94%EB%94%A9-%ED%8C%81-%EB%8B%A4%EC%84%AF-%EA%B0%80%EC%A7%80-df65ffb4e7e)
- [WEB API 리스트](https://developer.mozilla.org/ko/docs/WebAPI)
- [Jake 의 JS microtasks queue](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
- [HTML 도큐 이벤트루프](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)
- [stackoverflow JS promise 와 resolve](https://stackoverflow.com/questions/28921127/how-to-wait-for-a-javascript-promise-to-resolve-before-resuming-function)
- [stackoverflow JS 이벤트 루프에서 microtask vs macrotask](https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)
- [stackoverflow JS 이벤트 루프는 어떻게 실행 되는가](https://stackoverflow.com/questions/38705444/javascript-event-loop-where-do-web-apis-get-executed)
- [MDN 의 WEB API 도큐](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Introduction)
- [MDN WEB API 리스트](https://developer.mozilla.org/ko/docs/WebAPI)
