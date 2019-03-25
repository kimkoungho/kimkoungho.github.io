---
title: "Javascript Callback hell"
excerpt: ""
categories: javascript
tags:
  - javascript event loop
  - javascript Callback hell
last_modified_at: 2018-02-24
---

# Javascript Callback Hell

## Javascript 콜백 지옥 ??
Javascript 는 비동기로 동작한다. 사실 Callback Hell 이라고 부르는 콜백지옥은 비동기 프로그래밍 전통적인 문제임.
- Javascript 비동기 동작방식에 대한 내용은 [이전 포스트](https://kimkoungho.github.io/javascript/javascript-event-loop/)를 확인


```javascript

```



## Callback Hell 의 문제점 
- 가독성 
- 실수 하기 쉬움 (상위 변수에 접근하는 문제)
- 콜백 함수의 return 값 ... (기본적으로 return 값이 없으며, )


## 해결 방법


### callback 함수 추출 


### Promise





### Async / Await
async/await 은 Javascript 비동기 처리에 대한 callback 중첩과 같은 문제들을 해결하기 위해서 ECMAScript 2017 에서 발표되었다. 

async function 의 선언은 비동기 함수를 반환하는 함수를 정의하는 것으로 암시적으로 프로미스를 사용하여 결과를 반환한다. 
async function 내부에서는 await 식이 포함될 수 있는데, (await 은 일반 function 에서는 사용할 수 없음 - Syntax Error)
await 은 async function 의 실행을 일시 중지하고 전달된 프로미스의 결과를 기다린 다음 
await 이후의 코드 블럭을 실행한다. 


```Javascript

```

async/await 의 장/단점
- 코드의 대한 가독성
- 디버깅이 훨씬 쉬움 (.then 블록 내에서 중단점을 설정하면 동기 코드를 단계별로 수행하기 때문)
- await 은 프로미스가 return 되기까지 해당 코드를 blocking 함으로써 무분별한 사용으로 인해서 성능저하가 생길 수 있음 
- 서로 종속성을 갖지 않는 것들은 Promise.all() 등을 이용하여 비동기 병렬처리를 하는 것이 좋음


### Generator 


### 


TODO: 
- 프로미스 정의, 여러 메소드 

## Reference
<http://callbackhell.com/>
<https://blog.hellojs.org/asynchronous-javascript-from-callback-hell-to-async-and-await-9b9ceb63c8e8>
<https://meetup.toast.com/posts/89>
<https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/>
<https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context>
<https://html.spec.whatwg.org/multipage/webappapis.html#event-loop>
<https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/await>
<https://javascript.info/async-await>
