---
title: "스프링 5 리액티브 프로그래밍"
excerpt: ""
categories: spring
tags:
    - docker 설치 
last_modified_at: 2020-12-06
---
이 글은 [실전! 스프링5 리액티브 프로그래밍](http://www.yes24.com/Product/Goods/74394497?OzSrank=1) 스터디를 진행하면서 작성한 글입니다.

# 1장

# 2장 

다룰 주제 
- 관찰자 패턴
- 발행-구독(publish-subscribe) 패턴 
- RxJava 의 역사 및 기본 개념 
- Marble(마블) 다이어그램 
- 리액티브 프로그래밍을 적용한 비지니스 사례 
- 리액티브 라이브러리 현황 

* 1장에서 Future 와 CompletableFuture 에 대해서 다룸 
ㄴ 자세히 안나옴 ... 모던 자바 스터디에 있는데 .. 추가할까 ?? 

## 리액티브 프로그래밍 기본 개념 

### Observer(관찰자) 패턴 
리액티브 프로그래밍의 기초라고 할 수 있는 **GoF(Gang of Four) 디자인 패턴** 중 하나인 Observer 패턴에 대해서 공부할 필요가 있다  
- 디자인 패턴에 대한 자세한 내용은 [GoF 디자인 패턴 위키](https://en.wikipedia.org/wiki/Design_Patterns) 에서 확인할 수 있다   
  
Observer 패턴에서는 관찰자라고 불리는 Subject(주제) 를 필요로 함 
주체는 일반적으로 자신의 메서드 중 하나를 호출해서 관찰자에게 상태 변경을 알림
ㄴ 거의 모든 UI 라이브러리가 이러한 패턴을 사용함   
  
Observer 패턴 class 다이어 그램 
[observer pattern class diagram](/assets/images/posts/20201201/observer_pattern_class_diagram.png)  
  
위 클래스 다이어그램에서 볼 수 있듯이 관찰자 패턴에서는 Subject 와 Observer 2개의 인터페이스로 구성된다  
Observer 는 Subject 에 등록되고 Subject 의 notifyObserver() 메소드가 호출을 통해서 Subject 에 등록된 Observer 들이 호출되는 형식이다  
Java 코드로 나타내면 아래와 같다  

```java
public interface Subject<T> {
    void registerObserver(Observer<T> observer);
    void unregisterObserver(Observer<T> observer);
    void notifyObservers(T event);
}

public interface Observer<T> {
    void observe(T event);
}
```
=======


관찰자 패턴을 사용하면 런타임에 객체 사이에 일대다 의존성을 등록할 수 있음 
이를 통해 각 부분이 활발히 상호작용하게 하면서도 응용 프로그램 사이의 결합도를 낮출 수 있음 
이런 유형의 통신은 단방향으로 이루어지며 , 다이어그램에서 나오는 것처럼 시스템을 통해 이벤트를 배포하는데 도움이 된다 




## 리액티브 프레임 워크 RxJava
[RxJava](https://github.com/ReactiveX/RxJava) 1.x 버전은 Java 진영에서 리액티브 프로그래밍을 위한 표준 라이브러리 였음  
ㄴ RxJava 를 제외하고 리액티브 라이브러리는 AKKa Streams, Reactor(스프링 webflux 의 기반) 라이브러리가 있음  
RxJava 라이브러리는 **Reactive Extensions(ReactiveX)** 의 자바 구현체
[ReactiveX](http://reactivex.io/) 는 명령형 언어를 이용하여 동기식 or 비동기식 데이터 스트림을 조작할 수 있는 도구로 Observer 패턴, Iterator 패턴 및 함수형 프로그래밍의 조합으로 정의된다  
ㄴ 뒤에서 추가 설명이 나오지만 ReactiveX 는 MS 에서 만들었다고 한다  

### 관찰자 + 반복자 = 리액티브 스트림 
관찰자 패턴 요약
```java
public interface Subject<T> {
    void registerObserver(Observer<T> observer);
    void unregisterObserver(Observer<T> observer);
    void notifyObservers(T event);
}

public interface Observer<T> {
    void observe(T event);
}
```
관찰자 패턴을 이용한 접근법은 무한한 데이터 스트림에 대해서는 매력적이지만, 데이터 스트림의 끝을 알리는 기능이 있으면 더욱 매력적일것임  
또한 컨슈머가 준비되기 이전에 프로듀서가 이벤트를 생성하는 것은 우리가 바라는 상황이 아니기 때문에 이를 방지해야한다  

이럴때 동기식 셰계에서는 반복자 패턴을 이용할 수 있음  
```java
public interface Iterator<T> {
    T next();
    boolean hasNext();
}
```
hasNext() 를 이용하여 결과의 끝을 알 수 있음  
  
비동기식 관찰자 패턴에 이런 반복자 패턴을 혼합해보자  
```java
public interface RxObserver<T> {
    void onNext(T next);
    // 이벤트의 끝을 알리는 메소드 
    void onComplete(); 
    // 오류 전파를 위한 메소드 
    void onError(Exception e);
}
```
우리는 위 클래서를 이용하여 리액티브 스트림의 모든 컴포넌트 사이에 데이터가 흐르는 방법을 정의할 수 있다  
리액티브 스트림 Observable 클래스 = 관찰자 패턴의 Subject 클래스 이며, 이벤트를 발생 시킬 때 이벤트 소스 역할을 수행  
실제로 Observable 클래스에는 수십 가지의 팩토리 메소드와 수백 가지의 스트림 변환 메소드가 포함되어 있다  
ㄴ 

===
Subscribe 추상 클래스는 Observable 인터페이스를 구현하고 이벤트를 소비하는 역할을 한다  
Observable 과 Subscriber 의 관계는 메시지 구독 상태를 확인하고 필요한 경우 이를 취소할 수도 있는 구독에 의해 제어된다 
===

[RxJava observer and observable](/assets/images/posts/20201201/reactive_observable_observer.png)  
RxJava 에서 Observable 은 0개 이상의 이벤트를 보낼수 있다  
그런 다음 성공을 알리거나 오류를 발생시켜 실행 종료를 알린다  
따라서 각 구독자에 대한 Observable 은 onNext() 를 여러번 호출한 다음 onComplete() 또는 onError() 를 호출한다  
  
### 스트림의 생산과 소비 
RxJava 의 Observable 로 표현되는 스트림 정의  
```java
Observable<String> observable = Observable.create(
        new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> sub) {
                sub.onNext("Hello, reactive world!");
                sub.onCompleted();
            }
        }
);
```

위에 작성한 코드는 구독자가 나타나자 마자 이벤트 1회 전파후 바로 종료함  

> 
 RxJava 1.27 부터는 Observable 을 이용하는 방식은 더 이상 사용되지 않음  
 이 방식은 생성하는 것들이 너무 많고 구독자에게 과도한 부하를 줄 수 있어 안전하지 않다  
 즉, backpressure 를 지원하지 않는다 
 자세한 내용은 나중에 다시 다룰 것임. 다만 기본 개념을 익히기 위해서 위의 코드를 작성함  

구독자 코드를 작성해보자 
```java
Subscriber<String> subscriber = new Subscriber<String>() {
    @Override
    public void onNext(String s) {
        System.out.println(s);
    }
    @Override
    public void onCompleted() {
        System.out.println("Done!");
    }
    @Override
    public void onError(Throwable e) {
        System.out.println(e);
    }
};
```
ㄴ Subscriber 에서 onError 메서드를 구현하지 않으면 오류가 발생하면 rx.exceptions.OnErrorNotImplementedException 을 발생시킨다 
  
실행 결과 
```
Hello, reactive world!
Done!
```

람다식으로 표현해본 코드 
```java
Observable.create(
        sub -> {
            sub.onNext("Hello, reactive world!");
            sub.onCompleted();
        }
).subscribe(
        System.out::println, // onNext
        System.out::println, // onError
        () -> System.out.println("Done!") // onCompleted
);
```

RxJava 라이브러리는 Observable 및 Subscriber 인스턴스를 생성하기 위해서 많은 유연성을 제공함  
```java
Observable.just("1", "2", "3", "4")
        .subscribe(
                System.out::println, // onNext
                System.out::println, // onError
                () -> System.out.println("Done!") // onCompleted
        );
Observable.from(new String[]{"A", "B", "C"})
        .subscribe(
                System.out::println, // onNext
                System.out::println, // onError
                () -> System.out.println("Done!") // onCompleted
        );
Observable.from(Collections.emptyList())
        .subscribe(
            System.out::println, // onNext
            System.out::println, // onError
            () -> System.out.println("Done!") // onCompleted
        );
```
  
또한 Callable, Future 를 이용해서 생성 가능하며, concat 메소드를 이용해서 입력 스트림을 다운 스트림 Observable 로 다시 보낼수 있다  
입력 스트림은 종료 신호(onCompleted(), onError()) 가 발생할 때까지 처리되며 처리 순서는 concat 의 인수의 순서대로 처리된다  
```java
Observable<String> hello = Observable.fromCallable(() -> "Hello ");
Future<String> future = Executors.newCachedThreadPool().submit(() -> "World");
Observable<String> world = Observable.from(future);

// concat 으로 스트림을 연결하여 forEach 로 문자열을 출력 
Observable.concat(hello, world, Observable.just("!"))
        .forEach(System.out::print);
```
실행 결과
```
Hello World!
```

### 비동기 시퀀스 생성하기  
interval 메소드를 이용해서 주기적으로 비동기 이벤트 시퀀스를 생성할 수 있다  
```java
Observable.interval(1, TimeUnit.SECONDS)
        .subscribe(e -> System.out.println("Received: " + e));

// 이 코드를 제거하면 아무것도 일어나지 않는다
Thread.sleep(5000);
```
Thread.sleep 메소드를 제거하면 아무것도 출력하지 않고 종료되는데, 이것은 이벤트가 생성되는 스레드는 현재 코드가 실행되는 스레드와 다른 스레드이기 때문  
Thread.sleep 을 이용해서 메인 스레드 종료를 지연시킨 것으로 메인 스레드가 종료되지 않는다면 subscribe 는 무한히 호출된다  
  
구독을 취소하기위해서 Subscription 이라는 인터페이스가 존재한다  
```java
public interface Subscription {
    void unsubscribe();
    boolean isUnsubscribed();
}
```
구독자는 unsubscribe() 메소드를 이용해서 구독을 취소할 수 있으며,  
Observable 은 isUnsubscribed() 메소드를 이용해서 구독자가 여전히 이벤트를 기다리는지 확인한다  
  
구독 취소하는 예제  
```java
// countDown() 3 번 호출되야 종료됨 
CountDownLatch externalSignal = new CountDownLatch(3);

Subscription subscription = Observable
        .interval(100, TimeUnit.MILLISECONDS)
        .subscribe(System.out::println);

// 3초간 대기하도록 ..
Thread thread = new Thread(() -> {
    try {
        while(externalSignal.getCount() > 0) {
            Thread.sleep(1000);
            externalSignal.countDown();
        }
    } catch(InterruptedException e){}
});
thread.start();

// 대기한다 count = 0 일때까지 
externalSignal.await();
subscription.unsubscribe();

// unsubscribe 이후에는 몇 초를 대기하던 출력되지 않음
Thread.sleep(2000);
```
CountDownLatch 는 await() 이 호출되어 count = 0 이 되기전까지 대기하고 count = 0 이 되면 unsubscribe() 가 호출되어 구독을 취소한다  
구독 취소 이후에는 몇 초를 기다리던 아무것도 출력되지 않는다  
ㄴ 책 예제가 불완전하여 코드를 추가함  


### 스트림 변환과 마블 다이어그램 
RxJava 에는 거의 모든 시나리오에서 사용할 수 있는 엄청난 양의 연산자가 있지만, 여기서는 기본 연산자들을 살펴보자  
대부분의 다른 연산자들은 기본 연산자의 조합일 뿐임  
  
마블 다이어그램(marble diagram) 은 리액티브 커뮤니티에서 리액티브 스트림을 시각적으로 표현할 때 사용하는 다이어그램  
- 수평선으로 표시된 리액티브 스트림에 임의의 순서로 구성요소가 기하학적 모형 으로 표시된다  
- 특수 기호는 에러나 종료 신호를 나타냄 
- box 는 해당 요소가 어떻게 변화하는지를 나타낸다 
  
#### Map 연산자 
```java
public final <R> Observable<R> map(Func1<? super T, ? extends R> func)
```
- T 를 R 로 변환하는 func 을 파라미터로 받아서 Observable<T> 를 Observable<R> 로 변환할 수 있음을 나타낸다  
- RxJava 에서 가장 많이 사용하는 연산자 
[marble diagram map](/assets/images/posts/20201201/marble_diagram_map.png)  
위 다이어그램을 보면 map 을 통해서 원소가 하나씩 변환되는 것을 알 수 있다  
ㄴ 사실 자바의 함수형 인터페이스인 Function 을 Observable 로 wrapping 한 것 처럼 보인다  

```java
Observable.interval(100, TimeUnit.MILLISECONDS)
        .map(e -> e * 2)
        .subscribe(System.out::println);

Thread.sleep(500);  
```
  
#### Filter 연산자 
```java
public final Observable<T> filter(Func1<? super T, Boolean> predicate)
```
- 파라미터로 predicate 를 받아서 조건이 만족하는 요소만 filtering 할 수 있는 연산자 
[marble diagram map](/assets/images/posts/20201201/marble_diagram_filter.png)  
위 다이어그램을 보면 조건을 만족한 원소들만 추출된다  
  
### Count 연산자 
```java
public final Observable<Integer> count()
```
- 입력 스트림의 개수를 발행
- 스트림이 완료될때에만 카운트가 발행되기 때문에 스트림이 무한대일 때는 count 연산자가 완료되지 않거나 아무것도 반환하지 않는다  
[marble diagram map](/assets/images/posts/20201201/marble_diagram_count.png)  
```java
Observable.just("1", "2", "3", "4")
        .count()
        .subscribe(System.out::println);

// 무한 스트림은 결과가 없다 
Observable.interval(100, TimeUnit.MILLISECONDS)
        .count()
        .subscribe(System.out::println);

Thread.sleep(500);
```

#### Zip 연산자 





