---
title: "Test Double"
excerpt: ""
categories: testing
tags:
  - test double
  - mock vs stub
last_modified_at: 2019-07-07
---

## Test Double (테스트 더블)

**테스트 더블 이란?**
- 테스트를 진행할 때, 실제 객체를 대신에 사용하는 객체를 **테스트 더블** 이라고 함
- 테스트 더블은 실제 객체와 같은 인터페이스를 사용하여 구현 
- 테스트 더블을 사용하는 클라이언트는 해당 객체가 테스트 더블인지 실제 객체인지 알 수 없음 
- _"Meszaros, Gerard"_ 가 _"xUnit Test Patterns"_ 책에서 소개한 용어로 "_메스제로스_" 는 테스트 더블을 **스턴트 맨** 에 비유
- "_메스제로스_" 의 [xunitpatterns 포스팅](http://xunitpatterns.com/Test%20Double.html)

   
### Test Double 사용시기
1. 테스트 대상을 의존 객체와 관계를 끊고 격리시킬 때 
2. 예측 불가능한 요소를 통제하여 테스트하고 싶을 때
3. 느린 테스트를 더 빠르게 만들고 싶을 때
4. 통합 환경 부재로 실제와 같은 대용품 객체를 만들고 싶을 때
5. 실제 객체를 사용하기 불편할 때

   
### Test Double 종류
![No Image](/assets/images/posts/20190622/TypesOfTestDoubles.gif#center)

> 
! 테스트 더블의 분류는 _메스제로스_ 가 최초에 정의했지만, 이 후 각각의 의미가 다른 느낌으로도 쓰임 ...
...


### Dummy Object
- 일반적으로 호출 되었을 경우 어떤 동작도 하지 않는 빈 객체
- 단순히 파라미터 리스트 채우기 위해 사용됨 
- 엄밀히 테스트 더블로 보기는 어려움 
- ex) Java 의 라이브러리의 HashSet 클래스 : 내부적으로 HashMap 을 이용하며 value 에는 dummy object 를 이용
  * [HashSet 구현](https://github.com/openjdk-mirror/jdk7u-jdk/blob/master/src/share/classes/java/util/HashSet.java)


### Fake Object
- 실제 구현이 존재하는 객체로 실 환경에서는 사용할 수 없지만, 테스트를 위해서 사용하는 객체
- 수행하고자 하는 테스트가 환경에 의존적인 경우에 사용
  * 통합 테스트에서 DB 접속으로 인해서 테스트 수행이 느린 경우
- ex) Fake DB, In-memory DB, Fake Web Service
- ex) 실제 DB 에 접속하는 DAO 가 아닌 테스트 환경에서 Fake DAO 를 이용
![No Image](/assets/images/posts/20190622/fake_object.png#center)


### Test Stub
- 일반적인 테스트 스텁은 주어진 입력에 대한 의도된(하드 코딩된 값)을 반환하도록 구현
- 하드 코딩된 값 : 테스트 작성자가 의도한 값 (예외를 포함)
- 일반적으로 테스트를 위해서 작성된 내용을 제외하고 아무런 동작을 하지 않음 (stubbing 된 것에만 응답)
- 테스트 대상 객체를 독립적으로 단위 테스트를 수행할 때 사용 
- ex) 학생의 평균 등급을 구하는 기능을 테스트 하려고 하는 경우 
![No Image](/assets/images/posts/20190622/test_stub.png#center)
- 실제 DB 값을 이용하는 대신에 하드 코딩된 값을 이용해서 평균 등급을 구하는 기능에 대한 테스트를 작성할 수 있음 
- 즉, 테스트 스텁은 현재 테스트 하고자 하는 1가지 상태를 표현

- Mockito 프레임워크를 이용한 예제  
```java
@Test
public void averageServiceStubTest() {
    /** set up */
    // dependency object
    StudentGradeDAO studentGradeDAO = mock(StudentGradeDAO.class);
    // test target
    AverageService averageService = new AverageServiceImpl();
    // inject
    averageService.setStudentGradeDAO(studentGradeDAO);
  
    /** given */
    String studentId = "testId";
    StudentGrade studentGrade = new StudentGrade(8d, 6d, 10d);
  
    // stubbing
    when(studentGradeDAO.getStudentGrade(studentId))
        .thenReturn(studentGrade);
    
    /** then */
    // exercise
    Double average = averageService.getAverage(studentId);
    
    // verify
    Double expect = (8 + 6 + 10) / 3.0;
    Assert.assertEquals(expect, average);
}
```


### Mock Object
- 목 객체는 행위를 검증 하기 위해서 사용하는 하나의 스텁 
  * 하나의 스텁 이므로 주어진 입력에 대해서 의도된 값을 반환하는 부분은 같음 
- 목 객체를 이용해서 해당 메소드가 수행되었는지 확인 할 수 있음 
- ex) 메일링 서비스 처럼 테스트 환경에서는 수행하고 싶지 않은 서비스가 있을 수 있음 
  * 하지만 해당 목 객체가 호출되었는지를 검증하기를 원하는 경우 사용
- ex) 조건에 따라서 Window.close() 또는 Door.close() 를 수행한다고 가정하자 
![No Image](/assets/images/posts/20190622/mock_object.png#center)
- 실제 Window 와 Door 를 사용하는 대신 WindowMock 과 DoorMock 을 이용 
- 우리가 작성한 테스트는 DoorMock.close() 를 호출하고 해당 메소드가 호출되었는지 검증함

- Mockito 프레임워크를 이용한 예제  
```java
@Test
public void securityServiceMockTest() {
    /** set up */
    // dependency object
    Window windowMock = mock(Window.class);
    Door doorMock = mock(Door.class);
    // test target
    SecurityService securityService = new SecurityService(windowMock, doorMock);

    // exercise
    securityService.securityOn();

    // verify : door.close()
    verify(doorMock).close();
    verify(windowMock, never()).close();
}
```


### Test Spy
> 
테스트 스파이는 "메스제로스" 가 [xunitparttern Test Spy](http://xunitpatterns.com/Test%20Spy.html) 포스팅에서 정의한 것과 최근 Mock 프레임워크에서 다른 의미로 사용되고 있음 ..  
"메스제로스" 는 테스트 스파이는 약간의 정보를 기록하는 하나의 테스트 스텁 이라고 정의함 
ex) 테스트 스텁의 호출 횟수 정보를 기록 


최근 Mock 프레임 워크
- 테스트 스파이는 임의의 메소드를 stubbing 할 수 있는 실제 객체를 의미
- [Mockito 프레임워크 spy](https://static.javadoc.io/org.mockito/mockito-core/2.28.2/org/mockito/Mockito.html#13)

- Mockito 프레임워크를 이용한 예제  
```java
@Test
public void listSpyTest() {
    // spy
    List list = spy(List.class);
    
    // stubing
    when(list.size())
      .thenReturn(-1);
    
    list.add("123");
    
    // size 에 대해서 stubing 해서 size 가 음수 
    Assert.assertEquals(-1, list.size());
}
```


### Mock VS Stub
"마틴 파울러"의 ["Mocks Aren’t Stubs"](https://martinfowler.com/articles/mocksArentStubs.html) 에서 목과 스텁에 대한 차이점을 설명함 
- [번역](http://testing.jabberstory.net/)


**목과 스텁의 차이점**
- 목은 행위 검증을 목적으로 하고 스텁은 상태 검증을 목적으로 함 
  * 상태 검증 : 주어진 입력에 대한 출력 값을 예상 값과 비교 
  * 행위 검증 : 주어진 입력에 따라서 어떤 행위를 수행했는지 검증 (A, B 메소드 중 어느것을 수행했는지 검사)

>
"마틴 파울러" 의 포스팅에서는 목과 스텁의 차이점을 **객체 생성 방법의 차이점** 과 **검증 방법의 차이점** 을 예로 들어 설명했다.   
하지만 [Spock 프레임워크](http://spockframework.org/spock/docs/1.1/all_in_one.html#Stubs) 에 따르면 **객체 생성 방법의 차이점**은 거의 차이가 없는 듯 하다.   
다만, **검증 방법의 차이점** 에서는 목에 대해서는 상태 검증과 행위 검증을 모두 제공하는 반면, 스텁은 상태 검증만을 제공한다.  
[참고] Mockito 프레임워크에서는 stub 을 따로 제공하지 않는다.

- 사실 목과 스텁에 대한 차이점에 대해서 stackoverflow 에 아직도 질문이 정말 많음 ...
- 그 중 [Spock 프레임워크 문서를 참조해 작성한 stackoverflow 답변](https://stackoverflow.com/questions/24413184/difference-between-mock-stub-spy-in-spock-test-framework)
>
해당 답변에서 강조 하는 것은 스텁이 목보다 간단하며, 행위 검증이 필요한 경우에만 목을 사용하는 것을 권하고 있음   
또한 스파이에 대해서는 실제 객체를 사용하는 것이므로 주의를 기울여 사용해야 한다고 함


> 
사내에서 코드 커버리지를 높이기 위해서 테스트 코드를 작성하던 중에 개발환경에서 성공하던 테스트 코드가 실 환경에서 실패하는 경험을 하고  
사수 분께서 Mockito 프레임워크 이용해서 작성한 테스트 코드를 참조하여 테스트를 작성했다.   
항상 성공하는 테스트 코드를 작성하기에 Mockito 프레임워크는 매우 매력적으로 느껴졌음.   
물론 Mock 을 이용한 단위 테스트 뿐만 아니라, 실환경을 이용한 테스트가 필요한 경우도 있는것 같음 (DB 데이터를 조회하는 DAO 같은 경우)   


[소스](https://github.com/kimkoungho/TDDProject)

TODO:
- Junit 정리
- mockito 정리 

### Reference
- [xUnit Test Double](http://xunitpatterns.com/Test%20Double.html)
- [xUnit Test Object](http://xunitpatterns.com/Fake%20Object.html)
- [xUnit Test Stud](http://xunitpatterns.com/Test%20Stub.html)
- [xUnit Test Mock](http://xunitpatterns.com/Mock%20Object.html)
- [xUnit Test Spy](http://xunitpatterns.com/Test%20Spy.html)
- [xUnit Mocks, Fakes, Stubs](http://xunitpatterns.com/Mocks,%20Fakes,%20Stubs%20and%20Dummies.html)
- [slideshare 자바 테스트 자동화](https://www.slideshare.net/gyumee/ss-90206560)
- [위키 mock object](https://en.wikipedia.org/wiki/Mock_object)
- [Michal Lipski test-double-fakse-mocks](https://blog.pragmatists.com/test-doubles-fakes-mocks-and-stubs-1a7491dfa3da)
- [jabberstory 마틴 파울러 모의객체는 스텁이 아니다 번역](http://testing.jabberstory.net/)
- [martinfowler 모의객체는 스텁이 아니다](https://martinfowler.com/articles/mocksArentStubs.html)
- [spockframework stub](http://spockframework.org/spock/docs/1.1/all_in_one.html#Stubs)
- [stackflow spy 프레임워크 mock, stub, spy](https://stackoverflow.com/questions/24413184/difference-between-mock-stub-spy-in-spock-test-framework)
- [solutionsiq mock or not mock](https://www.solutionsiq.com/resource/blog-post/to-mock-or-not-to-mock-is-that-even-a-question/)
- [openjdk 해시셋 git](https://github.com/openjdk-mirror/jdk7u-jdk/blob/master/src/share/classes/java/util/HashSet.java)


