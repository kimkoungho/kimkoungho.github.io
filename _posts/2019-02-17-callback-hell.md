---
title: "Javascript Callback Hell"
excerpt: ""
categories: javascript
tags:
  - Callback Hell
  - javascript Callback Hell
  - promise
  - generator
  - async / await
last_modified_at: 2018-03-29
---

# Javascript Callback Hell

## Javascript Callback Hell(콜백 지옥) 이란 ?
- 콜백 지옥이란 함수의 매개변수로 전달하는 콜백함수가 중첩되는 현상 
- Javascript 가 비동기 프로그래밍임으로 발생하는 문제 
- 사실 Callback Hell 이라고 부르는 콜백지옥은 비동기 프로그래밍 전통적인 문제임.
- Javascript 비동기 동작방식에 대한 내용은 [이전 포스트](https://kimkoungho.github.io/javascript/javascript-event-loop/)를 확인
- express 프레임워크 등을 개발한 node.js 의 거물 [TJ](https://github.com/tj)가 js 를 버리고 go 로 가면서 **"callback suck"** 이라는 말을 남겼을 정도 .. 

## 콜백 지옥 예제
- 게시판 요구사항 정의 
1. 해당 기능을 JQuery 의 AJAX 로 구성 .. 
2. 검색 버튼 클릭 -> 검색 조건에 해당하는 게시글의 total count 와 list 를 출력 


```javascript
// 검색 버튼 클릭 이벤트 
$("#searchBtn").click(function(e){
  e.preventDefault();

  // 검색 파라미터 
  var searchParam = getSearchParam();

  // 검색 조건에 해당하는 게시글 count 조회 
  $.getJSON("/board/count", searchParam, function(res){
      var count = res;

      if(count > 0){
        // 검색 조건에 해당하는 게시글 list 조회 
        $.getJSON("/board/list", searchParam, function(res){
            var list = res;
            // dom 변경사항 적용 
        });
      }
  });
});
```
- JQuery를 사용해 봤다면 위의 예제와 같은 콜백 형식을 많이 볼 수 있다.
- 위 예제에서는 3개의 콜백 함수만이 이용되었지만, 조건이 추가되어 더 많은 비동기 함수가 추가된다면 ... 그야 말로 콜백 지옥에 빠질 수 있다.
  * 콜백함수 1 : 검색 버튼 클릭 이벤트에 대한 콜백함수
  * 콜백함수 2 : AJAX 로 count 를 조회하는 콜백함수
  * 콜백함수 3 : AJAX 로 list 를 조회하는 콜백함수

## Callback Hell(콜백지옥) 의 문제점 
- 가독성이 떨어짐
- 예외 처리에 대한 복잡도 증가
- 확장성이 떨어짐 

### 가독성이 떨어짐
- 가독성 : indent 가 많아지면서 코드를 읽게 힘들게 되는것으로 눈으로 봐도 알 수 있다. (실제 코드에서 indent가 내 모니터의 절반 이상을 차지하게 되는것을 생각해보면 ...)
  * 개인적으로 node.js 로 된 callback hell 을 본적 있는데 ... 너무 .. 읽기 싫었다...

### 예외 처리에 대한 복잡도 증가
- 예를 들어 콜백함수에서 에러가 발생한다면 ? 
```javascript
try {
  // 검색 조건에 해당하는 게시글 count 조회 
  $.getJSON("/board/count", searchParam, function(res) {
    throw 'error'
  }
}catch(e) {
  // 에러 처리 ...
}
```
- 콜백 함수 내부에서 예외를 던졌지만, 위와 같은 방식을 이용해서는 예외를 catch 할 수 없다.
```javascript
// 검색 조건에 해당하는 게시글 count 조회 
$.getJSON("/board/count", searchParam, function(res){
    try{
      throw 'error'
    }catch(e){
        // 에러 처리 ...
    }
}
```
- 위와 같은 방식으로 콜백 함수 내부에서 예외 처리를 해야함.
  * JS 이벤트 루프와 관련이 있음. [이전 포스트](https://kimkoungho.github.io/javascript/javascript-event-loop/)를 확인
- 원본 예제에서의 예외 처리 
```javascript
// 검색 버튼 클릭 이벤트 
$("#searchBtn").click(function(e){
  try {
    e.preventDefault();
    // 검색 파라미터 
    var searchParam = getSearchParam();
    // 검색 조건에 해당하는 게시글 count 조회 
    $.getJSON("/board/count", searchParam, function(res){
        try {
          var count = res;
          if(count > 0){
            // 검색 조건에 해당하는 게시글 list 조회 
            $.getJSON("/board/list", searchParam, function(res){
                try {
                  var list = res;
                  // dom 변경사항 적용 
                } catch (e) {
                  // 에러 처리 ...
                }
            });
          }
        } catch(e) {
            // 에러 처리 ...
        }
    });
  } catch(e) {
    // 에러 처리 ...
  }
});
```
- 와우 ... 가독성도 그렇고 들여쓰기가 너무 많아졌음 ...

### 확장성이 떨어짐 
- 최초의 요구사항에서 페이지 네비게이션 기능을 추가 해보자 
- 단, 페이지 이동 이후에는 alert 을 출력하도록 하자 - 좀 억지이지만 일단 해보자 ..  

```javascript
// 검색 버튼 클릭 이벤트 
$("#searchBtn").click(function(e){
  e.preventDefault();

  // 검색 파라미터 
  var searchParam = getSearchParam();

  // 검색 조건에 해당하는 게시글 count 조회 
  $.getJSON("/board/count", searchParam, function(res){
      var count = res;

      if(count > 0){
        // 검색 조건에 해당하는 게시글 list 조회 
        $.getJSON("/board/list", searchParam, function(res){
            var list = res;
            // dom 변경사항 적용 
        });
      }
  });
});

// 페이지 이동 이벤트
$(".movePage").click(function(e){
  e.preventDefault();
  // 가정일 뿐 ..
  var page = e.target.getAttribute("page");

  // 검색 파라미터 
  var searchParam = getSearchParam();
  searchParam.page = page;

  // 검색 조건에 해당하는 게시글 list 조회 
  $.getJSON("/board/list", searchParam, function(res){
      var list = res;
      // dom 변경사항 적용 
      alert("페이지 이동후 게시글 리스트 출력");
  });  
});
```

- 게시글의 list 를 조회하는 것이 중복되어 보인다.. 중복은 언제나 나쁘니까 이를 추출해보자 

```javascript
// 검색 조건에 해당하는 게시글 list 조회 
// 조회후 callback 함수를 호출 
function getBoardList(searchParam, callback){
  $.getJSON("/board/list", searchParam, function(res){
    var list = res;
    // dom 변경사항 적용 
    callback();
  });
}

// 검색 버튼 클릭 이벤트 
$("#searchBtn").click(function(e){
  e.preventDefault();
  // 검색 파라미터 
  var searchParam = getSearchParam();

  // 검색 조건에 해당하는 게시글 count 조회 
  $.getJSON("/board/count", searchParam, function(res){
      var count = res;

      if(count > 0){
        getBoardList(searchParam, null);
        // getBoardList(searchParam); // 사실 이렇게 해도된다... 
      }
  });
});

// 페이지 이동 이벤트
$(".movePage").click(function(e){
  e.preventDefault();
  // 가정일 뿐 ..
  var page = e.target.getAttribute("page");

  // 검색 파라미터 
  var searchParam = getSearchParam();
  searchParam.page = page;

  var callback = function(){
    alert("페이지 이동후 게시글 리스트 출력");
  }

  // 검색 조건에 해당하는 게시글 list 조회 
  getBoardList(searchParam, callback);
});
```

- 좀 억지라고 볼 수도 있을 꺼 같다. 하자만 공통된 로직을 굉장히 여러 군데에서 사용하는 경우는 반드시 생기기 마련이다.
- 예를들어) 사용자 인증후에만 사용할 수 있는 사이트가 있다면, 사용자의 인증처리는 공통 로직이나 .. 그 이후 처리들은 모두 각각 다를것이기 때문에 
  * 페이지 요청시 서버에서 하면되잖아? 라고 생각할 수 있지만 .. 
  * 처음에 인증된 사용자가 일정한 시간이 흘러 인증이 풀린 후 AJAX 요청을 했을때 문제의 소지는 있다.


## 해결 방법
1. 콜백 함수의 분리
2. Promise
3. Generator 와 co 라이브러리 
4. Async / Await


### 1.콜백 함수의 분리
- 이 방법은 중첩된 익명의 콜백함수들을 추출하는 방법으로 [callbackhell.com](http://callbackhell.com/) 에서 소개되었다.

```javascript
function clickSearch(e){
  e.preventDefault();
  // 검색 파라미터 
  var searchParam = getSearchParam();

  var listCallback = function(res){
    var list = res;
    // dom 변경사항 적용 
  }
  
  getBoardCount(searchParam, function(res){
      var count = res;
      if(count > 0){
          getBoardList(searchParam, listCallback);
      }
  });
}
// count
function getBoardCount(searchParam, callback){
  $.getJSON("/board/count", searchParam, callback);
}

// list
function getBoardList(searchParam, callback){
  $.getJSON("/board/list", searchParam, callback);
}

// 검색 버튼 클릭 이벤트 
$("#searchBtn").click(clickSearch);

function clickMovePage = function(e){
  e.preventDefault();
  // 가정일 뿐 ..
  var page = e.target.getAttribute("page");

  // 검색 파라미터 
  var searchParam = getSearchParam();
  searchParam.page = page;

  var listCallback = function(res){
    var list = res;
    // dom 변경사항 적용 
    alert("페이지 이동후 게시글 리스트 출력");
  }

  // 검색 조건에 해당하는 게시글 list 조회 
  getBoardList(searchParam, listCallback);
}

// 페이지 이동 이벤트
$(".movePage").click(clickMovePage);
```

- 이 방법에 대해서는 사람마다 다르게 느낄 수 있을 것 같다. 오버 엔지니어링 아님? 이라고 생각할 수 있다.
- 무엇보다 내용도 별로 없는데, 중첩을 없애고자 모두 함수로 추출하고 나니 함수가 너무 많아졌기 때문에 ..
- 하지만 getBoardList() 는 검색 이벤트와 페이지 이동 이벤트에서 서버의 데이터를 받아온 뒤의 처리가 달라지므로 callback 함수를 파라미터로 전달 받는 것이 맞다. 

### 2.Promise
- 이러한 JS 의 콜백 지옥에 대한 대안으로 Promise.js 가 라이브러리 형태로 계속 사용되어 왔다.
- [MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise) 의 정의에 따르면, 프로미스란 비동기 작업의 성공, 실패에 대한 핸들러를 연결해주는 대리자이다.
- 프로미스는 프로미스 객체를 반환함으로써 비동기 콜백에 대한 처리를 좀 더 편리하게 할 수 있다.
- Promise.js 는 ES6(ECMA 2015+) 에서 정식으로 언어적 차원에서 지원을 시작했다.
- 초기에는 브라우저 별로 지원하지 않는 브라우저들이 있었으나 .. 최근 브라우저에서는 모두 지원한다고 함.
- 프로미스는 다른 비동기 함수들 보다 우선되어 처리되는데 ... 이는 JS 이벤트 루프와 관련이 있음 
  * 프로미스와 이벤트 루프에 대한 내용은 [이전 포스트](https://kimkoungho.github.io/javascript/javascript-event-loop/)를 확인


### 기본적인 문법 
- 프로미스 생성 : resolve 와 reject 함수를 인자로 갖는 콜백함수를 전달한다.
  * resolve(value) : 해당 프로미스가 성공했을 경우에 호출하는 함수로써 value 를 리턴함
  * reject(value) : 해당 프로미스가 실패했을 경우에 호출하는 함수로써 value 를 리턴함

```javascript
var promise = new Promise(function(resolve, reject){
  // resolve('value'); 
  // reject('value');
});
```

- 프로미스를 실행
  * then(callbck) : 프로미스가 성공했을 경우 callback 호출됨
  * catch(callback) : 프로미스가 실패했을 경우 callback 호출됨
  * finally() : 프로미스가 종료되기 전에 실행될 내용을 작성 

```javascript
var promise = new Promise(function(resolve, reject){
  // ...
});

promise
  .then(function(value){
    // 성공했을 경우 실행될 내용 
  })
  .catch(function(error){
    // 실패했을 경우 실행될 내용 
  })
  .finally(function(){
    // 종료되기 전에 실행될 내용 
  });
```

- 프로미스의 체이닝 : 프로미스는 then() 함수에서 프로미스를 반환하여 체이닝으로써 사용할 수 있음.

```javascript
var promise1 = new Promise(function(resolve, reject){
  // ...
});
var promise2 = new Promise(function(resolve, reject){
  // ...
});

promise1
  .then(function(value){
    // promise1 이후 처리내용
    return promise2;
  })
  ..then(function(value){
    // promise2 이후 처리내용
  })
```

### 프로미스의 3가지 상태 
- 프로미스는 생성된 이후 종료될 때까지 3가지 상태가 있음. 여기서 각 상태는 프로미스의 처리과정을 의미함
  * Pending(대기) : 비동기 처리가 아직 완료되지 않은 상태  
  * Fulfilled(실행) : 비동기 처리가 완료되어 프로미스가 결과를 반환한 상태 
  * Rejected(실패) : 비동기 처리가 실패하거나 오류가 발생한 상태 

- 프로미스 workflow - 출처: [MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)
![No Image](/assets/images/posts/20190217/promises_workflow.png)
  * Promise 는 Promise 객체를 생성후 비동기 처리가 완료되지 않은것을 의미함 - pending 상태
  * 비동기 처리가 완료된 경우
    + 성공시 then(onFullfillment) 가 실행됨
    + 실패시 catch(onRejection) 가 실행
    + 실패시 then(onRejection) 언제 실행되는지 모르겠다 ...
  * then(), catch() 에서 프로미스를 생성해서 반환하는 경우 위의 과정이 반복됨 

- 프로미스의 처리과정을 예제를 통해 이해해보자 

```javascript
// 프로미스 생성 : 프로미스 생성할 때 바로 비동기 처리가 실행됨 
let promise1 = new Promise(function(resolve, reject){
  setTimeout(function(){
      resolve('promise1 success');
  }, 500);
});
 
let promise2 = new Promise(function(resolve, reject){
  reject(new Error('promise2 error'));
});
 
promise1.then(function(msg){
  console.log(msg);
  return promise2;
}).then(function(msg){
  console.log(msg);
}).catch(function(error){
  console.log(error);
}).finally(function(){
  console.log('exit promise');
});
```

- 결과
![No Image](/assets/images/posts/20190217/promise.png)

- 코드를 분석해보자
1. promise1 의 정의 : setTimeout() 으로 500ms 이후에 'promise1 success' 을 성공으로 반환
2. promise2 의 정의 : reject() 함수를 이용하여 에러를 발생시킴 
3. promise1.then() 을 호출하여 'promise1 success' 을 출력하고 promise2 를 반환
4. promise2 에서는 에러를 발생 시켰으므로 then() 이 아닌 catch() 가 출력되어 에러 내용이 출력
5. finnally() 를 이용하여 프로미스 종료전 'exit promise' 를 출력 


### 콜백 함수의 분리방법에 프로미스를 적용해보기 
```javascript
function clickSearch(e){
  e.preventDefault();
  // 검색 파라미터 
  var searchParam = getSearchParam();
  
  // count 조회 프로미스 
  getBoardCount(searchParam)
    .then(function(count){
      if(count > 0){
        // list 조회 프로미스 
        return getBoardList(searchParam);
      }
    })
    .then(function(list){
        // dom 변경사항 적용 
    })
    .catch(function(err){
      // error 처리
    });
}

// count 조회 프로미스 
function getBoardCount(searchParam){
  return new Promise(function(resolve, reject){
    $.getJSON("/board/count", searchParam, function(res){
      resolve(res);
    });
  });
}

// list 조회 프로미스 
function getBoardList(searchParam){
  return new Promise(function(resolve, reject){
    $.getJSON("/board/list", searchParam, function(res){
      resolve(res);
    });
  });
}

// 검색 버튼 클릭 이벤트 
$("#searchBtn").click(clickSearch);

function clickMovePage = function(e){
  e.preventDefault();
  // 가정일 뿐 ..
  var page = e.target.getAttribute("page");

  // 검색 파라미터 
  var searchParam = getSearchParam();
  searchParam.page = page;

  // 검색 조건에 해당하는 게시글 list 조회 
  getBoardList(searchParam)
    .then(function(list){
      // dom 변경사항 적용 
      alert("페이지 이동후 게시글 리스트 출력");
    })
    .catch(function(err){
      // error 처리
    });
}

// 페이지 이동 이벤트
$(".movePage").click(clickMovePage);
```
- 우선 검색 이벤트에서 프로미스의 체이닝을 이용하여 getBoardCount()와 getBoardList() 의 콜백 함수를 연결하여 가독성이 상승된 것을 볼 수 있다.
- 프로미스를 이용하여 비동기 코드를 체이닝 기법으로 동기 처럼 보이는 효과를 얻을 수 있다.
- **실제로 JQuery 1.8 이후로 AJAX 요청에 대해서 프로미스를 지원하지만 여기서는 고려하지 않음**

### 프로미스로 병렬 처리하기 - Promise.all(promise1, ..)
- Iterable 가능한 파라미터(배열과 같은)를 인자로 받아 모든 프로미스를 병렬처리하고 그 결과를 배열 형태로 resolve 한다.

```javascript
let start = new Date();
 
Promise.all([
    new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
    new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
    new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3
  ]).then(console.log) // [ 1, 2, 3 ]
    .then(function(){
        let end = new Date();
        console.log(end - start);
    })
    .catch(console.log);
```

- 결과 
![No Image](/assets/images/posts/20190217/promise_all.png)

- 처리과정
1. Promise.all 을 통해 3개의 프로미스가 인자로 전달된다. 앞에서 설명한 것 처럼 프로미스는 생성되자 마자 비동기 처리를 실행함 
2. 각 프로미스 객체는 1초, 2초, 3초 뒤에 resolve 
3. 3개의 프로미스의 resolve 된 결과를 배열로 생성후 출력. [ 1, 2, 3 ]
4. 3개의 프로미스 종료후 걸린 시간 출력 - 대략 3초 

### 여러 프로미스 중 우선 처리하기 - Promise.race(promise1, ..)
- Iterable 가능한 파라미터(배열과 같은)를 인자로 받아 각 프로미스 중 가장 먼저 처리한 프로미스를 반환 
  * 즉, 각 프로미스는 경쟁하여 먼저 처리된 것을 우선 처리할 수 있다.

```javascript
let start = new Date();
 
Promise.race([
    new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
    new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
    new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3
  ]).then(console.log) // 3
    .then(function(){
        let end = new Date();
        console.log(end - start);
    })
    .catch(console.log);
```

- 결과 
![No Image](/assets/images/posts/20190217/promise_race.png)

- 처리과정
1. Promise.race 를 통해 3개의 프로미스가 인자로 전달된다. 앞에서 설명한 것 처럼 프로미스는 생성되자 마자 비동기 처리를 실행함 
2. 각 프로미스 객체는 1초, 2초, 3초 뒤에 resolve 
3. 3개의 프로미스가 경쟁하여 1초 뒤에 실행되는 3을 출력 
4. 3개의 프로미스 종료후 걸린 시간 출력 - 대략 1초

### 3.Generator 
- [MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/function*)의 정의에 따르면, Generator(제네레이터)는 실행 도중 일시정지가 가능한 함수이다.
- 기존에 JS 프로세스를 일시정지 할 수 있는 방법은 alert, confirm, prompt 를 사용하는 방법이 있었다. 하지만 이 방법은 사용자의 이벤트에 의존적임으로 종료 시점을 사용자에 의존적일 수 밖에 없었다.
- 하지만 제네레이터를 이용한다면, 순수 스크립트 만으로도 일시정지와 재실행이 가능하다는 장점이 있다.
- 제네레이터의 제약은 생성자로써 사용될 수 없다는 것이다. (Type Error 를 발생시킨다.)
- 제네레이터는 ES6 에서 소개되었음

### Generator의 동작방식 

![No Image](/assets/images/posts/20190217/generator_workflow.png)
- 수행 과정 (여기서는 제네레이터가 이미 선언되어 실행되는 과정만 설명함)
1. generator 호출하면 제네레이터는 iterator 객체를 반환
2. iterator 객체의 next() 함수를 호출 
3. generator 내부로 진입하여 최초의 yield 를 만날때 까지 실행한뒤 yield 에서 값을 반환 (여기서 현재 제네레이터의 상태를 저장)
4. 다시 next()가 호출되면 최초의 yield 에서 다음 yield 를 만날때 까지 실행한 뒤 또 다음 yield 에서 값을 반환 (resume 이 일어나 이전에 저장한 상태 이후의 내용을 실행)
5. 더 이상 yield 가 존재하지 않거나 return 문을 만날때까지 위의 과정을 반복

### Generator의 기본 문법
- 제네레이터 선언

```javascript
function *generator(param){
  // 제네레이터의 중단점 설정
  yield 1;
  // 제네레이터의 최종 종료 시점 
  return 3;
}
```

- 제네레이터의 호출

```javascript
// iterator 가 반환 
let gen = generator(param);

// 최초의 yield 를 만날 때까지 실행
// {value, done} 형식의 object 를 반환 
// value : 실제 반환 값
// done : 제네레이터의 종료 여부 (return 문을 만난 경우 true)
gen.next(); 
// 다음 yield 를 만날 때까지 실행
gen.next();
```

### Generator 기본 예제
- 제네레이터로 값을 주고 받으며 결과를 출력하는 예제 
```javascript
function *generator(param) { // param = 3
    const x = yield (param * 2); // x = 10
    const y = yield (x + 1); // y = 20
    const z = yield (y + 2); // z = 30
    return x + y + z;
}
 
const iter = generator(3);
console.log(iter.next());   // {value:6, done:false}
console.log(iter.next(10)); // {value:11, done:false}
console.log(iter.next(20)); // {value:22, done:false}
console.log(iter.next(30)); // {value:60, done:true}
``` 

- 결과 
![No Image](/assets/images/posts/20190217/generator.png)

- 처리과정
1. 제네레이터로 3을 전달하고 이터레이터 생성
2. next() 실행 -> (param * 2) 를 반환 
3. 콘솔에 결과를 출력 - {value:6, done:false}
4. next(10) 실행 -> 10이 x에 담기고 (x + 1) 를 반환
5. 콘솔에 결과를 출력 - {value:11, done:false}
6. next(20) 실행 -> 20이 y에 담기고 (y + 2) 를 반환
7. 콘솔에 결과를 출력 - {value:22, done:false}
6. next(30) 실행 -> 30이 z에 담기고 (x + y + z) 를 반환 
7. 콘솔에 결과를 출력 - {value:60, done:true}, 6번에서 return 문을 만났으므로 제네레이터를 종료


### Generator 의 yield*
- yield* 키워드를 이용하여 iterator 객체를 연결해서 사용가능

```javascript
// 연결될 제네레이터
function* anotherGenerator(i) {
    yield i + 1;
    yield i + 2;
    yield i + 3;
}
// 호출될 제네레이터
function* generator(i){
    yield i;
    // 제네레이터를 연결
    yield* anotherGenerator(i);
    yield i + 10;
}

// for .. of 문을 사용하여 다음 대상이 존재하는 동안 반복 
// 결과는 {value, done} 중에 value 만 반환함 
for(let val of generator(10)){
    console.log(val);
}
```

- 결과
![No Image](/assets/images/posts/20190217/generator_yield_iter.png)


### Generator 와 비동기 (+프로미스)
- 사실 제네레이터 자체는 비동기 처리와 관련이 없다. 위에서도 살펴봤지만 제네레이터는 콜 스택에 의존적이지만, 비동기 처리는 이벤트 루프와 연관이 있기 때문이다.
- 제네레이터와 프로미스를 함께 사용하면 가능하다. 예제로 확인해 보자.

```javascript
function promise1(){
  return new Promise(function(resolve, reject){
    setTimeout(()=> resolve("promise 1"), 1000);
  });
}
function promise2(param){
  return new Promise(function(resolve, reject){
    setTimeout(()=> resolve(param + ", promise 2"), 500);
  });
}
// 제네레이터 선언 
function* generator(){
  const value1 = yield promise1();
  return promise2(value1);
}

// 실행 영역 
let iterator = generator();
let ret = null;
(function runNext(val) {
    ret = iterator.next(val); 

    if (!ret.done) {
        // ret.value = 프로미스 객체 
        ret.value.then(runNext);
    } else {
        ret.value.then((result)=> console.log(result));
    }
})();
```

- 결과 
![No Image](/assets/images/posts/20190217/generator_and_promise.png)

- 각 함수들은 프로미스를 반환하고 제네레이터에서는 프로미스를 반환하도록 구현했다.
- 수행과정
1. 프로미스들을 반환하는 메소드 정의 promise1(), promise2()
2. 제네레이터 선언부에서는 yield 문에서 각 프로미스를 반환
3. 제네레이터에 대한 이터레이터를 생성 
4. 익명 즉시 실행 함수 runNext()를 이용하여 재귀적으로 제네레이터가 종료할 때까지 반복
5. 마지막 프로미스가 반환되면 console.log() 로 결과를 출력

- 비동기 처리를 하기 위해서 매번 runNext() 와 같은 제네레이터 실행함수를 작성해야 할까?
- 그에 대한 대안으로 **co** 라이브러리가 있으나 여기서는 다루지 않을 생각이다.
  * [co 라이브러리](https://www.npmjs.com/package/co)

### Generator 와 프로미스로 기존 코드 개선

```javascript
// 제네레이터의 마지막을 실행해주는 함수
// iter: 제네레이터의 이터레이터
// lastCallback: 가장 마지막 프로미스 결과 이후를 처리할 callback
function runLast(iter, lastCallback){
  var ret = null;
  (function runNext(val) {
      ret = iterator.next(val); 

      if (!ret.done) {
          // ret.value = 프로미스 객체 
          ret.value.then(runNext);
      } else {
          ret.value.then(lastCallback);
      }
  })();
}


function clickSearch(e){
  e.preventDefault();
  // 검색 파라미터 
  var searchParam = getSearchParam();

  // 클릭 이벤트에 대한 제네레이터 생성후 이터레이터 
  function* generator(){
    try{
      var count = yield getBoardCount(searchParam);
      if(count > 0){
        return getBoardList(searchParam);
      }
    }catch(err){
      // error 처리
    }
  }

  var iter = generator();
  runLast(iter, function(list){
    // dom 변경사항 적용 
  });
}

// count 조회 프로미스 
function getBoardCount(searchParam){
  return new Promise(function(resolve, reject){
    $.getJSON("/board/count", searchParam, function(res){
      resolve(res);
    });
  });
}

// list 조회 프로미스 
function getBoardList(searchParam){
  return new Promise(function(resolve, reject){
    $.getJSON("/board/list", searchParam, function(res){
      resolve(res);
    });
  });
}

// 검색 버튼 클릭 이벤트 
$("#searchBtn").click(clickSearch);

function clickMovePage = function(e){
  e.preventDefault();
  // 가정일 뿐 ..
  var page = e.target.getAttribute("page");

  // 검색 파라미터 
  var searchParam = getSearchParam();
  searchParam.page = page;

  // 비동기 요청이 1번인 경우 -> 제네레이터 x 
  getBoardList(searchParam)
    .then(function(list){
      // dom 변경사항 적용 
      alert("페이지 이동후 게시글 리스트 출력");
    })
    .catch(function(err){
      // error 처리
    });
}

// 페이지 이동 이벤트
$(".movePage").click(clickMovePage);
```

### 4.Async / Await
- [MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function)의 정의에 따르면, async 함수는 여러 프로미스의 동작을 동기 스럽게 사용하도록 돕는다.
- async function 의 선언은 [AsyncFunction](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction) 객체를 반환하는 하나의 비동기 함수를 정의하는 것을 의미함
- 제네레이터와 프로미스를 묶어서 사용하는 것으로 볼 수 있음
- async/await 은 ECMAScript 2017 에서 발표되었다. 

### 기본적인 문법
- async function 의 선언

```javascript
async function asyncCall() {
  // ...
  // var value = await 표현식
};
```

- async function 내부에서는 await 식이 포함될 수 있음 
  * await 은 일반 function 에서는 사용할 수 없음 - Syntax Error
- await 은 generator yield 와 마찬가지로 함수의 실행을 일시중지 시키고 프로미스가 resovle, reject 되기까지 대기하다가 다음 코드 블럭을 실행 시킴 
- await 의 반환 값은 프로미스에서 resolve 된 값 
- 해당 프로미스가 reject 되면 await 은 에러를 throw 
- await 뒤에 나오는 표현식이 프로미스가 아니면, 해당 표현식을 프로미스의 resovle 로 변환시킴 

### async function - 프로미스 resolve 예제

```javascript
// 1초뒤 프로미스를 반환하는 함수 
function resolveAfter1Seconds() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve("resolved");
        }, 1000);
    });
}
  
async function asyncCall() {
    let start = new Date();
    console.log("start");

    let promise = resolveAfter1Seconds();
    console.log(promise);
    let result = await promise;
    console.log(result);

    console.log("end");
    let end = new Date();
    console.log(end - start);
}
 
asyncCall();
```

- 결과  
![No Image](/assets/images/posts/20190217/async_function_01.png)

- 처리과정
1. "start" 출력 
2. resolveAfter1Seconds() 호출 : setTimeout() 이 즉시 실행되고 프로미스가 반환됨 
3. 프로미스를 출력
4. let result = await promise : 프로미스가 resovle 되기 전까지 대기하고 resolve 되면 result 변수에 값이 저장
5. "resolved" 출력
6. "end" 출력
7. 실행시간 출력 (약 1초)

### async function - 프로미스 아닌경우 예제 
```javascript
async function asyncCall() {
    setTimeout(()=> {console.log("timeout")}, 0);

    let result = await "await promise";
    console.log(result);
}
 
asyncCall();
```

- 결과  
![No Image](/assets/images/posts/20190217/async_function_02.png)

- 처리과정 
1. setTimeout() 을 시작
2. "await promise" 가 프로미스로 변환되어 resolve 됨
3. "await promise" 출력 
4. "timeout" 출력
  * 프로미스가 setTimeout 보다 우선순위가 높아 먼저 출력됨 - 참고: [JS 이벤트루프](https://kimkoungho.github.io/javascript/javascript-event-loop/)

### async function - 프로미스 reject 예제
```javascript
function rejectAfter1Seconds() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            reject(new Error("reject"));
        }, 1000);
    });
}
  
async function asyncCall() {
    let start = new Date();
    console.log("start");

    let promise = rejectAfter1Seconds();
    console.log(promise);
    try {
      let result = await promise;
    } catch(err) {
      console.log(err);
    }

    console.log("end");
    let end = new Date();
    console.log(end - start);
}
 
asyncCall();
```

- 결과  
![No Image](/assets/images/posts/20190217/async_function_03.png)

- 처리과정
1. "start" 출력 
2. rejectAfter1Seconds() 호출 : setTimeout() 이 즉시 실행되고 프로미스가 반환됨 
3. 프로미스를 출력
4. 1 초 뒤 프로미스가 reject 되어 error throw 
5. catch 문에서 "reject" 출력
6. "end" 출력
7. 실행시간 출력 (약 1초)

### async function - 프로미스 순서대로 처리하기

```javascript
function resolveAfter2Seconds() {
    return new Promise(resolve => {
        setTimeout(function() {
            resolve(2);
        }, 2000);
    });
};
  
function resolveAfter1Second() {
    return new Promise(resolve => {
        setTimeout(function() {
            resolve(1);
        }, 1000);
    });
};
// 순서대로 실행
async function sequentialStart() {
    let start = new Date();
    
    const slow = await resolveAfter2Seconds();
    console.log(slow);
    const fast = await resolveAfter1Second();
    console.log(fast);
 
    let end = new Date();
    console.log(end - start);
}
 
sequentialStart();
```

- 결과   
![No Image](/assets/images/posts/20190217/async_function_04.png)

- 처리과정
1. await resolveAfter2Seconds() : 2초 뒤 프로미스를 resolve 하는 함수 호출하고 resolve 되기까지 대기
2. 2초 뒤 2 출력 
3. await resolveAfter1Second() : 1초 뒤 프로미스를 resolve 하는 함수 호출하고 resolve 되기까지 대기
4. 1초 뒤 1 출력
5. 실행시간 출력 (약 3초)

### async function - 프로미스 병렬 처리하기

```javascript
function resolveAfter2Seconds() {
    return new Promise(resolve => {
        setTimeout(function() {
            resolve(2);
        }, 2000);
    });
};
  
function resolveAfter1Second() {
    return new Promise(resolve => {
        setTimeout(function() {
            resolve(1);
        }, 1000);
    });
};

// 동시 실행, 순서대로 return
async function concurrentStart() {
    var start = new Date();
 
    // 2개의 프로미스가 동시에 실행됨 
    const slow = resolveAfter2Seconds(); 
    const fast = resolveAfter1Second();
  
    // 이미 실행되었으나 프로미스가 resolve 되기 전까지 대기 
    console.log(await slow);
    console.log(await fast); 
    var end = new Date();
    console.log(end - start);
}
 
concurrentStart();
```

- 결과   
![No Image](/assets/images/posts/20190217/async_function_05.png)

- 처리과정
1. const slow = resolveAfter2Seconds()  : 2초 뒤 프로미스를 resolve 하는 함수 호출 
2. const fast = resolveAfter1Second() : 1초 뒤 프로미스를 resolve 하는 함수 호출
3. console.log(await slow) : slow 프로미스가 resolve 될 때까지 대기한 후 2를 출력 
4. console.log(await fast) : fast 프로미스가 resolve 될 때까지 대기한 후 1을 출력 
5. 실행시간 출력 (약 2초)


### async function 으로 기존 코드 개선하기 

```javascript
async function clickSearch(e){
  e.preventDefault();
  // 검색 파라미터 
  var searchParam = getSearchParam();
  try {
    var count = await getBoardCount(searchParam);
    if(count > 0){
      var list = getBoardList(searchParam);
      // dom 변경사항 적용 
    }
  } catch (err) {
    // error 처리
  }
}

// count 조회 프로미스 
function getBoardCount(searchParam){
  return new Promise(function(resolve, reject){
    $.getJSON("/board/count", searchParam, function(res){
      resolve(res);
    });
  });
}

// list 조회 프로미스 
function getBoardList(searchParam){
  return new Promise(function(resolve, reject){
    $.getJSON("/board/list", searchParam, function(res){
      resolve(res);
    });
  });
}

// 검색 버튼 클릭 이벤트 
$("#searchBtn").click(clickSearch);

function clickMovePage = function(e){
  e.preventDefault();
  // 가정일 뿐 ..
  var page = e.target.getAttribute("page");

  // 검색 파라미터 
  var searchParam = getSearchParam();
  searchParam.page = page;
  
  try {
    var list = await getBoardList(searchParam);
    // dom 변경사항 적용 
    alert("페이지 이동후 게시글 리스트 출력");
  } catch (error) {
    // error 처리 
  }
}

// 페이지 이동 이벤트
$(".movePage").click(clickMovePage);
```

- async, await 을 사용하니 코드의 가독성이 훨씬 좋아졌다.
- async, await 이 ES8(ECMA 2017) 에 추가되어 브라우저별 제약사항이 있을 것 같지만, 잘만 활용한다면 콜백 지옥에도 벗어날 수 있을 것 같다.


## Reference
- <https://librewiki.net/wiki/%EC%BD%9C%EB%B0%B1_%EC%A7%80%EC%98%A5>
- <http://callbackhell.com/>
- <https://medium.com/@ejpbruel/the-drawbacks-of-callbacks-or-why-promises-are-great-5dedf2c98c67>
- <https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise>
- <https://stackoverflow.com/questions/42118900/when-is-the-body-of-a-promise-executed>
- <https://poiemaweb.com/es6-promise>
- <https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/function*>
- <https://meetup.toast.com/posts/93>
- <https://codeburst.io/understanding-generators-in-es6-javascript-with-examples-6728834016d5>
- <https://meetup.toast.com/posts/73>
- <https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function>
- <https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction>
- <https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/await>
- <https://javascript.info/async-await>
