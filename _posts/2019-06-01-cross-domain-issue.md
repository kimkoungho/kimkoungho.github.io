---
title: "크로스 도메인 이슈(Cross-Domain Issue)"
excerpt: ""
categories: javascript
tags:
  - Cross-Domain Issue
  - Same-origin Policy
  - CROS
  - JSONP
last_modified_at: 2019-06-01
---

# 크로스 도메인 이슈

## 동일 출처 원칙(Same-origin Policy)
동일 출처 원칙 이란 ?
- 동일 출처 원칙은 웹 어플리케이션 보안 모델에서 중요한 개념으로써 대부분의 브라우저들이 해당 정책을 따르고 있습니다.
- 동일 출처(same-origin)에서만 스크립트를 이용해서 데이터의 접근을 허용하는 정책입니다.
- 동일 출처 원칙은 image 태그, css, javasript 등을 내장시키는 것에는 적용되지 않습니다.
ex) a.com, b.com 2개의 페이지가 있다고 가정합시다.
    a.com 에서 event 가 발생하면 b.com 에 dom 요소를 변경하고 싶습니다.
    하지만 a.com 에서는 b.com 과 출처가 다르므로 동일 출처 원칙에 따라 요청이 거부됩니다.


## 출처 (origin)
출처 (origin) 은 URI 스키마, Host, Port 번호 등으로 구성된 정보를 의미합니다.

ex) http://www.example.com 과 출처 비교
> 
|비교 대상 URL|결과|이유|
|-----------|----|----|
|http://www.example.com/main|성공|같은 호스트, 포트, 프로토콜
|-----------|----|----|
|http://www.example.com:81|실패|다른 포트
|-----------|----|----|
|https://www.example.com|실패|다른 프로토콜
|-----------|----|----|
|http://mail.example.com|실패|다른 호스트 (정확한 일치 필요)
|-----------|----|----|
|http://www.example.com:80|의존|명시적 포트 (브라우저 구현에 따라 다름)


## 동일 출처 원칙의 필요성
동일 출처 원칙이 왜 필요할 까요 ?
- XSS 와 같은 스크립트 삽입 공격을 통해서 출처가 다른 웹 어플리케이션에 접근을 방지할 수 있습니다.
- 최근 웹 어플리케이션 들은 인증된 사용자 세션정보를 HTTP 쿠키에 담아서 광범위하게 사용하곤 하는데, 출처가 다른 페이지에서 스크립트를 이용해 해당 쿠키정보를 추출할 수 있기 때문입니다.
- 즉, 서로 관계 없는 사이트에서 제공하는 콘텐츠를 엄격히 분리하여 데이터 기밀성과 무결성을 보장할 수 있습니다.


## 해결 방법

### 1. document.domain 속성
- 이 방법은 부분적으로 도메인이 다른경우에 적용할 수 있는 방법입니다.
- orders.example.com 과 catelog.example.com 처럼 부분적으로 도메인이 다른 경우, document.domain = "example.com" 으로 설정하여 두 도메인의 출처를 같도록 할 수 있습니다.
- 이 방법을 사용하는 경우 암시적으로 port 번호는 null 로 설정
- 브라우저에서 해당 속성을 지정하기 위해서 2개 페이지 모두 해당 속성을 지정해야함

### 2. Cross-Orgin Resource Sharing
- 이 방법은 request header 와 response header 정보를 설정하여 동작
- 


### 3. Cross-document messaging
- 이 방법은 브라우저에서 제공하는 메시징 API 를 이용하는 방법으로 각 스크립트의 출처와 관계없이 텍스트 메시지를 전송하는 것을 제공합니다.
- 메시징 API 는 비동기로 호출
- 이 방법은 출처에 대한 확인이 충분하지 않을 경우 보안 이슈가 발생할 수 있습니다.

```javascript
/** targetWindow.postMessage(message, targetOrigin, [transfer])
  * targetWindow: 메시지를 전송할 윈도우 객체 (iframe, opener, parent, etc)
  * message: 전송할 테스트 메시지
  * targetOrigin: targetWindow 의 origin 을 명시적으로 작성
  *     targetWindow 의 실제 URL 과 다를 경우 메시지는 전달되지 않음
  *     특정 도메인이 아닌 경우 * 로 지정 (이 방법은 보안에 취약함)
  * transfer: 메시지와 함께 전송되는 일련의 개체 
  */



function receiveMessage(event){
    // 예상하는 origin 일때만 기능 수행
    if(event.origin == "www.example.com"){
        var data = event.data; // 전송된 문자열 메시지
        //event.source : 이벤트를 보낸 윈도우 객체 
    }
}

// message 이벤트 등록 
window.addEventListener("message", receiveMessage, false);

```

### 4. JSONP (JSON with Padding)
- 이 방법은 script 태그의 src 속성을 이용하여 데이터를 주고 받는 방법입니다. 위에서 언급했듯이 Same-origin Policy 는 서로 다른 origin 에 대해서 script 를 임베디드 하는 것은 허용한다는 점을 이용하는 것입니다.
- script 태그의 src 속성을 이용하면 src 에 적용된 자바스크립트 파일이 해당 페이지에 임베디드 되는것은 알고 계실겁니다.
- 데이터를 응답 해주는 origin 에서 데이터를 요청한 orgin 에서 전달한 callback 함수를 호출하여 데이터를 주고 받는 방식입니다.
- 동작 원리   
데이터를 요청하는 Origin : www.example.com   
데이터를 응답하는 Origin : www.sample.com
* 1.www.example.com 에서는 데이터 요청을 위해 script 태그를 작성하고 해당 요청에 대한 응답 함수를 정의 합시다.

```html
<!-- www.example.com 페이지의 head -->
<head>
    <!-- 응답시 동작할 call back 함수 작성 -->
    <script>
        function callbackLog(otherData){
            console.log(otherData);
        }
    </script>
    <!-- 요청을 위한 script 태그 작성 -->
    <script type="application/javascript" src="www.sample.com?callback=callbackLog"></script>
</head>
```

* 2.www.sample.com 에서는 응답 정보를 작성합니다.

```java
@RequestMapping(value = "/test", method = RequestMethod.GET)
public String jsonpTest(@RequestParam(value = "callback") String callback){
    // DB 조회등 서버 데이터 세팅
    Map<String, String> serverMap = new HashMap<String, String>();
    serverMap.put("serverCode", "jsonpGood");

    return callback + "(" + serverMap + ")";
}
```

... 스크린샷 ...
- JSONP 는 script 태그를 DOM 임베드함으로써 동작하기 때문에 GET 방식으로만 구현됩니다.
- 보안문제
* 1.신뢰할 수 없는 외부 코드
공격자는 JSONP 를 이용하여 원본 웹 페이지에 스크립트를 삽입하여 공격할 수 있음.
즉, XSS 공격이 가능함

* 2.콜백 메소드 이름 조작과 반사 파일다운로드 공격 (RFD : Refelcted File Download attack)
RFD: 파일 다운로드 요청에 대한 헤더나 본문을 조작하여 파일의 내용에 명령어나 악성코드를 삽입하는 취약점
JSONP 방식은 RFD 를 위한 방법으로 이용될 수 있음

* 3.교차 사이트 요청 변조 (CSRF)
HTML 내부에서 scr

* 4.Rosetta Flash
이 공격은 데이터를 응답하는 서버를 공격하는 방법입니다. 
플래시 파일을 callback 함수의 값으로 이용해서 서버가 해당 플래시 파일을 실행하게 만듬으로써 응답하는 서버를 제어할 수 있습니다.

### 5. WebSockets
- 





reference
<https://en.wikipedia.org/wiki/Same-origin_policy>
<https://en.wikipedia.org/wiki/Web_Messaging>
<http://m.mkexdev.net/75>
<https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage>
<https://en.wikipedia.org/wiki/JSONP>
<http://dev.epiloum.net/1311>
<https://blog.avira.com/understanding-rosetta-flash-vulnerability/>

