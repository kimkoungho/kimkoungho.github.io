---
title: "동일 출처 원칙(Same-origin Policy)"
excerpt: ""
categories: security
tags:
  - Same origin Policy
  - Cross domain issue 
  - document.domain
  - CORS
  - Cross-document messaging
  - JSONP
last_modified_at: 2019-07-27
---


## 동일 출처 원칙(Same-origin Policy) 이란?
[위키 Same-origin_policy](https://en.wikipedia.org/wiki/Same-origin_policy) 의 정의에 따르면,  
- **동일 출처(same-origin)** 에서만 스크립트를 이용하여 데이터(DOM)를 접근을 허용하는 정책
    * 출처(Origin)은 URI 스키마, 호스트, Port 로 구성된 정보를 말함
- 동일 출처 원칙은 웹 어플리케이션에서 중요한 보안 개념으로써 대부분의 브라우저들이 해당 정책을 따르고 있음
    * 동일 출처 원칙은 브라우저에서 동작하는 제약이기 때문에, 브라우저 별로 다를 수 있음
    * [MDN Same-origin_policy](https://developer.mozilla.org/ko/docs/Web/Security/Same-origin_policy) 를 보면 IE 브라우저에 대한 예외사항이 기술되어 있음 
- 동일 출처 원칙 사용 이유 
    * XSS 와 같은 스크립트 삽입 공격을 통해서 출처가 다른 웹 어플리케이션에 접근을 방지
    * 현대의 웹 어플리케이션은 인가된(authenticated) 사용자의 세션을 유지하기 위해서 HTTP Cookie 에 의존하기 때문
    * 최근 웹 어플리케이션 들은 인증된 사용자 세션정보를 HTTP 쿠키에 담아서 광범위하게 사용하곤 하는데, 출처가 다른 페이지에서 스크립트를 이용해 해당 쿠키정보를 추출할 수 있기 때문
    * 데이터의 기밀성 또는 일관성을 유지하기 위해서 클라이언트 측에서 관계 없는 사이트에서 제공된 컨탠츠를 분리해야함

### 출처 (origin) 결정 원칙 
URI 를 결정하는 알고리즘은 [RFC 6454](https://tools.ietf.org/html/rfc6454) 에 명시되어 있음  
출처 (origin) 은 URI 스키마, Host, Port 번호 등으로 구성된 정보를 의미  


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

- IE 는 출처를 결정할 때 포트를 사용하지 않고 Security Zone 을 사용


### 동일 출처 원칙의 적용 범위
- **1.** 동일 출처 원칙은 오직 스크립트 에만 적용됨
    * Microsoft Silverlight, Adobe Flash, or Adobe Acrobat 와 같은 다른 웹 스크립트에도 적용되어 있음 
    * DOM 제어와는 관련 없는 XMLHttpRequest 에도 적용되어 있음  

ex) iframe(or frame) 에서 출처가 다른 부모 frame 에 접근 

```html
<!-- parent url: www.example.com -->

<!-- current url: www.example-sub.com -->
<script>
    window.onload = function(){
        // error
        window.parent.resizeIFrame(document.body.scrollHeight);
    }
</script>
```

ex) Ajax 를 이용하여 출처가 다른 Rest API 호출

```javascript
// current url: www.example.com
// error 
$.getJson("www.openapi.com/image?appKey=~", function(){

}).fail(fucntion(){

});
```

- **2.** 동일 출처 원칙은 Html 문서 안에 포함된 image, css, 동적으로 로드된 스크립트 에는 적용되지 않음 
    * CSRF(Cross-site_request_forgery) 공격은 이러한 점을 이용한 공격 

```html
<!-- current url: www.example.com -->

<!-- bootstrap css -->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">

<!-- bootstrap js -->
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

<!-- pixabay image -->
<img src="https://pixabay.com/ko/photos/%EC%97%B0%EB%91%90-%EB%85%B9%EC%83%89-%EC%9E%8E-%EC%9E%90%EC%97%B0-%EB%82%98%EB%AD%87%EC%9E%8E-4291098/"/>
```

## 동일 출처 원칙 회피 방법

### 1. document.domain 속성
- 이 방법은 부분적으로 도메인이 다른경우에 적용할 수 있는 방법
- orders.example.com 과 catelog.example.com 처럼 부분적으로 도메인이 다른 경우, document.domain = "example.com" 으로 설정하여 두 도메인의 출처를 같도록 할 수 있음

```javascript
// current url : orders.example.com
// call url : catelog.example.com
document.domain = "example.com"
```
- 이 방법을 사용하는 경우 암시적으로 port 번호는 null 로 설정
- 브라우저에서 해당 속성을 지정하기 위해서 2개 페이지 모두 해당 속성을 지정해야함


### 2. CORS(Cross-Orgin Resource Sharing)
- 동일 출처 원칙을 회피하기 위한 방법으로 출처 자원 공유(CORS) 라는 방법이 표준으로 자리 잡음 
    * W3C 에서 권장하는 메커니즘 : [W3C CORS](https://www.w3.org/TR/cors/#cross-origin-request-with-preflight0)
- 이 방법은 HTTP 에 새로운 Origin 요청 헤더와 새로운 Access-Control-Allow-Origin 응답 헤더를 추가하는 방법
- 서버에서는 파일 요청을 허용한 명시적인 출처 리스트에 헤더를 사용하여 해당 사이트가 요청을 하도록 허용하는 방법
- 파이어폭스, 사파리, IE 10 에서는 이런 방법으로 XMLHttpRequest 로 원본 요청을 허용함

**CORS 종류**  

- Simple Request
- Preflight Request
- Credential Request
- Request without Credential

**Simple Request**
- Simple Request는 아래의 조건을 만족해야 함 (클라이언트 쪽 제약)
    * HTTP METHOD: GET, HEAD, POST 만 
    * 수동으로 설정 가능한 HTTP 헤더: Accept, Accept-Language, Content-Language, Content-Type
    * 가능한 Content-Type 종류: application/x-www-form-urlencoded, mulitpart/form-data, text/plain

- 동작 방식  
http://foo.example 에서 http://bar.other 에 요청하는 예시  

![No Image](/assets/images/posts/20190721/cors_simple_request.png)  

```javascript
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/post-here/';
var body = '<?xml version="1.0"?><person><name>Arun</name></person>';
    
function callOtherDomain(){
  if(invocation)
    {
      invocation.open('POST', url, true);
      invocation.setRequestHeader('X-PINGOTHER', 'pingpong');
      invocation.setRequestHeader('Content-Type', 'application/xml');
      invocation.onreadystatechange = handler;
      invocation.send(body); 
    }
}
```

**요청 헤더**

```sh
GET /resources/public-data/ HTTP/1.1
Host: bar.other
Referer: http://foo.example/examples/access-control/simpleXSInvocation.html
Origin: http://foo.example
# ... 생략 ...
```

**응답 헤더**

```sh
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
# *: 모든 호스트에 대하여 허용을 의미 
# ... 생략 ...
```

- 서버에서 응답 헤더에 Access-Control-Allow-Origin 속성을 추가하여 클라이언트가 컨탠츠를 사용하는 것을 허용

```sh
Access-Control-Allow-Origin: http://foo.example
```
- 특정 호스트에 대해서만 허용하는 방법 


**Preflight Request**
- Simple Request 가 아닌 경우 Preflight Request 에 해당한다
- Preflight Request 는 클라이언트에서 서버에 요청을 보내기전에 OPTIONS 메소드 방식으로 요청하는 것을 말함
- 동작 방식 

![No Image](/assets/images/posts/20190721/cors_preflight_request.png)

- 먼저 서버에 Preflight Request 를 보냄 

```sh
OPTIONS /resources/post-here/ HTTP/1.1
Host: bar.other
Origin: http://foo.example
Access-Control-Request-Method: POST # 실제 보낼 요청을 전달 
Access-Control-Request-Headers: X-PINGOTHER # 실제 요청의 커스텀 헤더 
# ... 생략 ...
```

- 서버에서 Preflight Request 에 대한 응답 

```sh
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://foo.example # 가능한 origin
Access-Control-Allow-Methods: POST, GET, OPTIONS # 가능한 메소드
Access-Control-Allow-Headers: X-PINGOTHER # 가능한 헤더 
Access-Control-Max-Age: 1728000 # Preflight Request 의 캐시시간 
Vary: Accept-Encoding, Origin
# ... 생략 ...
```

- 서버에 실제 요청을 전송

```sh
POST /resources/post-here/ HTTP/1.1
Host: bar.other
X-PINGOTHER: pingpong # 커스텀 헤더 
Referer: http://foo.example/examples/preflightInvocation.html
Origin: http://foo.example
# ... 생략 ...
```

- 서버에서 실제 요청에 대해 응답 

```sh
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://foo.example
# ... 생략 ...
```

**Credential Request**
- Http Cookie, Http Authentication 등의 인증된 정보를 인식할 수 있게 하는 요청

```javascript
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/credentialed-content/';
    
function callOtherDomain(){
  if(invocation) {
    invocation.open('GET', url, true);
    invocation.withCredentials = true;
    invocation.onreadystatechange = handler;
    invocation.send(); 
  }
}
```
- withCredentials 설정을 true 함으로써 Credential Request 요청 
- Credential Request 는 조건에 따라서 Simple Reqeust / Preflight Request 이다.
- 쿠키나 인증정보를 서버에서 응답하는 경우 해당 정보들에 접근하려는 상황이 Credential Request 임 
    * 실제 Ajax 등을 이용한 서버의 응답 쿠키는 자동으로 갱신되지 않음 

- 요청 헤더

```sh
GET /resources/access-control-with-credentials/ HTTP/1.1
Host: bar.other
Referer: http://foo.example/examples/credential.html
Origin: http://foo.example
Cookie: pageAccess=2
# ... 생략 ...
```

- 응답 헤더 

```sh
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Credentials: true # 쿠키 접근 가능함
# 응답 쿠키 
Set-Cookie: pageAccess=3; expires=Wed, 31-Dec-2008 01:34:53 GMT
# ... 생략 ...
```

- 서버에서 응답 헤더에 Access-Control-Allow-Credentials: true 를 반환함으로써 클라이언트에서 쿠키 접근이 가능해짐
- Access-Control-Allow-Origin: * 과 같은 응답은 불가능함 

**Request without Credential**
- CORS 요청은 기본적으로 Non-Credential 요청
- withCredentials 설정을 한 경우에만 Credential 요청



### 3. Cross-document messaging
- 이 방법은 브라우저에서 제공하는 메시징 API 를 이용하는 방법으로 각 스크립트의 출처와 관계없이 텍스트 메시지를 전송 가능 
- 메시징 API 는 비동기로 호출
- 이 방법은 출처에 대한 확인이 충분하지 않을 경우 보안 이슈가 발생할 수 있다.

```javascript
/** targetWindow.postMessage(message, targetOrigin, [transfer])
  * targetWindow: 메시지를 전송할 윈도우 객체 (iframe, opener, parent, etc)
  * message: 전송할 테스트 메시지
  * targetOrigin: targetWindow 의 origin 을 명시적으로 작성
  *     targetWindow 의 실제 URL 과 다를 경우 메시지는 전달되지 않음
  *     특정 도메인이 아닌 경우 * 로 지정 (이 방법은 보안에 취약함)
  * transfer: 메시지와 함께 전송되는 일련의 개체 
 **/

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
- 이 방법은 script 태그의 src 속성을 이용하여 데이터를 주고 받는 방법
- 위에서 언급했듯이 Same-origin Policy 는 서로 다른 origin 에 대해서 script 를 Html 내부에 임베디드 하는 것은 허용한다는 점을 이용함 
- 데이터를 응답 해주는 origin 에서 데이터를 요청한 orgin 에서 전달한 callback 함수를 호출하여 데이터를 주고 받는 방식
- JSONP 는 script 태그를 DOM 임베드함으로써 동작하기 때문에 GET 방식으로만 구현됨 

- 동작 원리   

> 
데이터를 요청하는 Origin : www.example.com   
데이터를 응답하는 Origin : www.sample.com

**1.** www.example.com 에서는 데이터 요청을 위해 script 태그를 작성하고 해당 요청에 대한 응답 함수를 정의

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

**2.** www.sample.com 에서는 응답 정보를 작성

```java
@RequestMapping(value = "/test", method = RequestMethod.GET)
public String jsonpTest(@RequestParam(value = "callback") String callback){
    // DB 조회등 서버 데이터 세팅
    Map<String, String> serverMap = new HashMap<String, String>();
    serverMap.put("serverCode", "jsonpGood");

    // 화면에서 전달 받은 호출할 콜백 함수에 data 를 넣어서 반환 
    return callback + "(" + serverMap + ")";
}
```

**JSONP vs Ajax**

[이미지 출처: http://dev.epiloum.net/1311](http://dev.epiloum.net/1311)  
![No Image](/assets/images/posts/20190721/comparison_between_ajax_and_jsonp.png)
- JSONP는 기본적인 동작 방식에는 Ajax와 큰 차이가 없지만 사용법의 차이가 존재
- Ajax는 응답을 responseText 속성으로 가져와서 XMLHttpRequest.onreadystatechange 에 있는 콜백 함수를 실행
- JSONP는 콜백함수 호출 코드를 서버에서 완성하여 반환 


### 5. WebSockets
- 현대의 브라우저는 웹 소켓 API 를 제공함 
- 요청 헤더에 Origin 속성에서 클라이언트의 호스트 정보를 알 수 있는 것을 이용하는 방식으로 서버 측에서 Origin 헤더를 검사하여 컨텐츠 접근을 허용하는 방식




### Reference
- <https://en.wikipedia.org/wiki/Same-origin_policy>
- <https://iamawebdeveloper.tistory.com/38>
- <https://developer.mozilla.org/ko/docs/Web/Security/Same-origin_policy>
- <https://www.internetmap.kr/entry/Sameorigin-policy>
- <https://developer.mozilla.org/ko/docs/Web/HTTP/Access_control_CORS>
- <https://zamezzz.tistory.com/137>
- <https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage>
- <https://en.wikipedia.org/wiki/JSONP>
- <http://dev.epiloum.net/1311>


