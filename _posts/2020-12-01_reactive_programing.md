---
title: "리액티브 프로그래밍 기본 개념"
excerpt: ""
categories: spring
tags:
    - 
last_modified_at: 2021-01-02
---
이 글은 [실전! 스프링5 리액티브 프로그래밍](http://www.yes24.com/Product/Goods/74394497?OzSrank=1) 스터디를 진행하면서 작성한 글입니다.

# 리액티브 프로그래밍이란?
[위키 Reactive_programming](https://en.wikipedia.org/wiki/Reactive_programming) 에 따르면 리액티브 프로그래밍은 data stream 와 변경된 사항을 전파(propagation) 와 관련이 있는 프로그래밍 패러다임이다  
이런 패러다임으로 정적(arrays), 동적(event emitters) 데이터 스트림을 쉽게 표현 할 수 있으며, 연관된 실행 모델 내에서 변경된 데이터 흐름의 자동 전파를 용이하게 한다  
  
말이 어려운데 쉽게 말해서 **리액티브 프로그래밍은 비동기 데이터 스트림으로 프로그래밍 하는것**이다  


리액티브 프로그래밍 예제

리액티브 프로그래밍 vs 시스템 
ㄴ https://atin.tistory.com/575
ㄴ 여기에 전체적인 내용이 녹아져 있다 ..


## 리액티브 시스템의 기본 원칙 
[리액티브 선언문](https://www.reactivemanifesto.org/en)는 리액티브 애플리케이션과 시스템 개발의 핵심 원칙을 공식적으로 정의한다  
![reactive](images/reactive-traits.svg)  
- Responsive (응답성) : 시스템이 가능한 한 사용자에게 즉시 응답하고 문제를 신속하게 탐지하고 대처할 수 있음을 의마한다. 응답성이 있는 시스템은 **일관성 있는 응답 시간을 제공**한다    
- Resilient (탄력성) : 시스템에 장애가 발생하더라도 응답성을 유지하는 것을 의미한다. 탄력성은 복제(replication), 격리(isolation), 위임(delegation) 으로 구현된다 
- Elastic (유연성) : 시스템의 작업 량이 변화하더라도 응답성을 유지하는 것을 유연성이라고 한다. 즉 요청의 증가, 감소에 따라서 자원을 증가시커나 감소 시킬 수 있음을 의미한다  
- Message Driven (메시지 구동) : 리액티브 시스템은 비동기 메시지 전달을 이용해서 구성 요소 간의 느슨한 결합, 격리, 위치 투명성을 지원할 수 있도록 경계를 명확히 정의할 수 있다.
    * 경계는 장애를 메시지로 전송하는 수단을 제공 -> 탄력성  
    * 명시적인 메시지 전달은 시스템에 메시지 큐를 생성하고 모니터링하여 배압(backpressure)을 제공 -> 유연성 


### 용어 ?



## 왜 리액티브 인가 ??

## 동기, 비동기 + blocking, non-blocking


## future , promise
https://en.wikipedia.org/wiki/Futures_and_promises

## 



## 간단한 역사 

## Reference
- [위키 Reactive_programming](https://en.wikipedia.org/wiki/Reactive_programming)
- [staltz-리액티브 프로그래밍에 대한 소개 gist](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)

- [리액티브 메니페스토](https://www.reactivemanifesto.org/ko)


