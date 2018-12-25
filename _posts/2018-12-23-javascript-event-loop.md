---
title: "Javascript 이벤트 루프"
excerpt: ""
categories: javascript
tags:
  - javascript event loop
  - javascript asynchronous 
last_modified_at: 2018-12-23
---

# Javascript 이벤트 루프

우선 Javscript 이벤트 루프를 이해하기 위해서 Javascript 런타임 환경 구성 요소들에 대한 이해가 필요함.


## Javascript 런타임 환경

Javascript 런타임 환경은 일반적으로 브라우저를 일컫는 말이며, node.js 에서는 별도의 LIBUV 라는 런타임 환경이 존재.

![No Image](/assets/images/posts/20181223/javascript_runtime_enviornment.jpeg)
<https://codenotcode.com/my-event-loop-beebef81cd46>
- 위 그림은 Javascript 런타임 환경에서 이벤트 루프를 설명하기 위해 필요한 것을 묘사


### Javascript Runtime Engine
실제로 Javscript 코드를 실행하는 역할을 수행
- Javscript 엔진은 여러 종류가 있으며, 각 엔진은 인터프린터 or Just-in-time 컴파일러를 이용해 코드를 실행

엔진 내부에는 코드를 실행하기 위한 힙과 스택 영역이 존재함
- 힙 영역 : 다른 언어들과 마찬가지로 Object 들이 할당되는 영역으로 코드 실행중에 동적으로 할당되는 영역
- 스택 영역 : 코드의 실행 블럭 쌓이는 영역

Javscript가 단일 스레드라고 불리는데 이는 엔진에서 단일 호출 스택을 이용한다는 관점에서 말하는 것
- 엔진이 단일 호출 스택을 이용하므로 엔진이 수행할 수 있는 코드 블럭은 1번에 1개만을 수행가능
- 멀티 스레드 환경에서는 각각의 스레드가 고유의 스택영역을 갖기 때문에 코드를 수행하므로 여러 코드를 수행가능
- Javscript의 실행 환경에서는 실제로 멀티 스레드로 구동되며, 각각의 스레드들과 엔진이 소통하기 위해서 이벤트 루프를 이용

### Web APIs
해당 영역은 런타임 환경(브라우저)에서 제공해주는 라이브러리로 이는 순수 Javascript 에서 제공하는 것이 아니라 런타임 환경에서 제공됨
- 런타임 환경이 브라우저라면 브라우저 이벤트에 대한 Web API 들이 제공됨 

이 영역은 라이브러리 영역으로써 개발자가 제어할 수 없는 영역으로 Web API 에서 제공하는 인터페이스 만을 이용해서 프로그램을 수행할 수 있음

Web APIs 에 대한 자세한 설명 - <https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Introduction>


### 이벤트 루프와 콜백 큐 
Javascript 런타임 환경에서 엔진과 Web API들을 포함한 여러 스레드들과 소통하기 위해서 이벤트 루프라는 방식을 채택함.


ex)
```Javascript
console.log('Hi');
setTimeout(function cb1() { 
    console.log('cb1');
}, 0);
console.log('Bye');
```
크롬에서 개발자 도구로 실행해보자
![No Image](/assets/images/posts/20181223/javascript_event_loop_ex_ret.jpeg)

cb1() 함수를 분명 0ms 로 지정하고 실행했음에도 'cb1' 보다 'Bye' 가 먼저 출력된다. 
- 이것은 이벤트 루프의 수행과정과 관련이 있다.

위 코드의 실행 순서를 살펴보자!
- 우선 console.log('Hi')가 콜 스택에 추가
- console.log('Hi')가 실행 (콘솔에 Hi 가 출력되는 시점이다)
- console.log('Hi')가 제거
- setTimeout(function cb1(){ ... })이 콜 스택에 추가 
- setTimeout(function cb1(){ ... })이 실행 
    브라우저 Web API에 cb1() 함수가 추가됨 (Web API 는 실행시간과 관련한 일종의 타이머를 생성함)
- setTimeout(function cb1(){ ... })이 제거
- console.log('Bye')가 콜 스택에 추가
- console.log('Bye')가 실행 (콘솔에 Bye 가 출력되는 시점이다)
- console.log('Bye')가 제거 
- 그런 다음 Web API의 타이머가 완료되면(setTimeout 에 지정한 시간이 지나면) cb1()함수를 콜백큐에 추가
- 이벤트 루프는 cb1()함수를 콜백 큐에서 콜 스택으로 이동
- console.log('cb1') 함수가 추가
- console.log('cb1') 함수가 실행 (콘솔에 cb1이 출력되는 시점이다)
- console.log('cb1') 함수가 제거 
![No Image](/assets/images/posts/20181223/javascript_event_loop_ex.gif)

위에서 실행 순서를 보았듯이 이벤트 루프는 Web API 에 등록된 함수를 콜백 큐에 넣고 
콜 스택이 비었을 경우 콜백 큐에서 콜 스택으로 해당 함수를 이동시킴

여기서 추가적으로 setTimeout 함수에 대해 알아보자.
```Javascript
setTimeout(function time(){
  console.log('timeout');
}, 1000);
```
setTimeout 함수를 실행하면 Web API는 일종의 타이머를 생성하고 실행하여 해당 함수를 1초 뒤에 수행시켜준다.
하지만 이것은 정확히 1초 뒤에 실행한다는 의미가 아니라, 
1초 뒤에는 해당 함수를 콜백 큐에 넣어주겠다는 의미이다. 
따라서 콜 백큐에 해당 다른 함수들이 있거나 콜 스택이 비어있지 않으면 time 함수는 2~3초 뒤에 수행될 수도 있음을 의미한다.

예제를 통해 살펴보았지만 MDN 의 이벤트 루프에 대한 가상코드를 보면 좀 더 명확히 이해할 수 있다.
```Javascript
while(queue.waitForMessage()){
  queue.processNextMessage();
}
```
위의 코드를 보면 waitForMessage() 메소드의 결과가 true인 동안 무한반복하면서 processNextMessage() 를 실행하고 있다.
waitForMessage() 메소드는 현재 실행할 대상이 없는 경우 동기적으로 무한 대기를 하는 코드이고
processNextMessage() 메소드는 다음 실행할 대상을 실행하는 코드이다.
- MDN 의 이벤트 루프 설명 : <https://html.spec.whatwg.org/multipage/webappapis.html#event-loops>


TODO 리스트
- Javascript 엔진에 대한 조사
- 프로미스, await 등 ES6 에서의 비동기 메소드들에 대한 내용 
- WEB API 리스트 : <https://developer.mozilla.org/ko/docs/WebAPI>

## Reference 
<https://meetup.toast.com/posts/89>
<https://codenotcode.com/my-event-loop-beebef81cd46>
<https://engineering.huiseoul.om/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94%EA%B0%80-%EC%97%94%EC%A7%84-%EB%9F%B0%ED%83%80%EC%9E%84-%EC%BD%9C%EC%8A%A4%ED%83%9D-%EA%B0%9C%EA%B4%80-ea47917c8442>
<https://engineering.huiseoul.com/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94%EA%B0%80-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84%EC%99%80-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%EC%9D%98-%EB%B6%80%EC%83%81-async-await%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%BD%94%EB%94%A9-%ED%8C%81-%EB%8B%A4%EC%84%AF-%EA%B0%80%EC%A7%80-df65ffb4e7e>
<https://developer.mozilla.org/ko/docs/WebAPI>