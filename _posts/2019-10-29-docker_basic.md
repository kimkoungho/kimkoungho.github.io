---
title: "Docker 란?"
excerpt: ""
categories: devops
tags:
    - docker 
    - docker 개념
    - docker container
    - docker image
last_modified_at: 2019-10-29
---

# 도커란? 
[위키 도커](https://ko.wikipedia.org/wiki/%EB%8F%84%EC%BB%A4_(%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4)) 정의에 따르면, 도커는 **컨테이너 가상화 기술을 사용하는 플랫폼** 으로 리눅스의 응용 프로그램들을 컨테이너라는 독립된 공간에 배치시키는 작업을 자동화하는 오픈소스 프로젝트이다. 


# 도커의 역사 
dotCloud 라는 PaaS 기업의 창업자인 Solomon Hykes 가 사내 프로젝트였던 도커를 2013년 3월 산타클라라에서 열린 Pycon Conference 에서 [The future of Linux Containers](https://www.youtube.com/watch?v=wW9CAH9nSLs&feature=youtu.be) 세션을 발표하면서 알려졌음  
발표 이후 도커가 엄청난 인기를 끌면서 회사 이름을 dotCloud 에서 Docker Inc 로 바꾸고 2014년 6월 docker 1.0 을 발표함  
  
현재 Solomon Hykes 는 Docker 를 퇴사 했는데, 창립자이면서 도커를 퇴사하는 Solomon Hykes 심정을 도커 블로그에 남겼다. 
- [Solomon Hykes 가 퇴사하면서 남긴 포스팅](docker.com/blog/au-revoir/)


# 컨테이너란?
[도커 레퍼런스 what-container](https://www.docker.com/resources/what-container) 에 따르면, 컨테이너는 애플리케이션의 코드와 구동 환경을 추상화한 논리적인 패키지 라고 설명하고 있는데, 쉽게 말하면 **실행가능한 격리된 공간** 이다.  

## VM vs 도커 Container
도커와 VM은 모두 가상화 기술을 이용한 소프트웨어라는 공통점이 있지만 **가상화를 위해 어떤 기술을 사용하는지, 어떤 시스템 레벨을 가상화하는지**에 대한 차이점이 존재한다.  
VM 에서 사용하는 하이퍼바이저 기반 가상화는 추가적으로 게스트 OS를 설치해야 하는 오버헤드가 존재했다. LXC(Linux Containers) 에서는 이런 문제를 해결하기 위해서 **프로세스를 격리** 하는 기법을 이용하여 VM과 유사한 역할을 하는 격리된 공간을 컨테이너로 정의한다.  
- LXC(Linux Containers) 는 도커 이전에 컨테이너 기반 가상화를 지원하는 리눅스의 프로젝트 
- 도커도 LXC 기반으로 개발되었다가 도커 0.9 버전에서 [libcontainer](https://github.com/docker/libcontainer) 라는 자체적인 컨테이너 기술을 사용하면서 LXC 없이도 동작할 수 있게 되었다. 도커 1.11 버전에서는 libcontainer 가 [runc](https://github.com/opencontainers/runc) 프로젝트에 합쳐졌다. 
- runc 는 도컨 엔진의 런타임 모듈로 호스트 OS 커널과 통신한다.


![No Image](/assets/images/posts/20191029/docker_vs_vm.png)   
출처: [도커 레퍼런스 what-container](https://www.docker.com/resources/what-container)  

- VM 의 하이퍼바이저는 하드웨어를 에뮬레이트 하는 방법을 사용하기 때문에 추가적인 게스트 OS 가 필요
- VM 의 호스트 OS 에 관련없이 독립적인 게스트 OS 를 구축할 수 있음 
- 도커 엔진은 호스트 OS 위에서 동작하며 추가적인 게스트 OS 없이 프로세스를 격리하여 컨테이너의 독립성을 보장
- 도커의 컨테이너는 호스트 OS 커널(리눅스 커널)을 공유함으로써 기존의 VM 방식에 비해서 빠르고 가볍게 동작 가능
- 하지만 도커는 호스트 OS 커널을 공유하기 때문에 별도의 OS 환경을 구축할 수 없음 

> 
가상화에 대해서 추가적인 내용은 이전 포스트인 [가상화 포스팅](https://kimkoungho.github.io/virtualization/)을 참고하자.


# 이미지란?

이미지는 **컨테이너를 구성하는 파일과 실행하는 애플리케이션 설정을 포함된 것**으로 컨테이너를 생성하는 하나의 **템플릿**이다. 이미지는 하나의 템플릿으로 같은 이미지를 이용해서 1개 or 여러개의 컨테이너를 생성하고 실행할 수 있다.  
  
이미지의 가장 큰 특징은 **Immutable(불변)** 이다. 이미지는 Immutable 이므로 실행 중인 컨테이너를 이용하여 이미지를 생성하는 경우 매번 새로운 이미지가 생성되고 컨테이너 생성후 추가되거나 변경된 사항은 컨테이너 자체에서 저장된다. 컨테이너가 삭제되어도 이미지에는 영향을 주지 않는다.  
  
도커 이미지는 [docker hub](https://hub.docker.com/)에 등록이 가능하며 ubuntu 이미지, nginx, mysql 등 자주 사용되는 이미지들은 docker hub 에서 다운받아서 사용가능하다. 또한 자체적인 [docker registry](https://docs.docker.com/registry/)를 구축하여 사용할 수도 있다. 


## 이미지 활용 예제 
1개의 컨테이너에는 mysql 을 설치하고 다른 컨테이너에는 mysql 와 wordpress 를 설치해야 한다고 가정해보자  
  
![No Image](/assets/images/posts/20191029/docker-image.png)   
출처: [subicura 도커란 무엇인가](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)  
위 그림에서 처럼 docker hub 에서 mysql image를 pull 받은 후, mysql 을 base image 로 하는 wordpress 이미지를 생성할 수 있다. 2개의 이미지를 이용하여 원하는 컨테이너를 생성하고 실행하는 것이 가능하다.  

## 레이어 저장 방식
도커 이미지는 컨테이너를 실행하기 위한 모든 정보를 가지고 있기 때문에 기존 이미지에 파일 1개를 추가했다고 이미지를 다시 다운받는 것은 비효율적이다. 도커는 이런 문제를 해결하기 위해서 **레이어** 라는 개념을 사용한다. 이미지를 각 레이어로 나누어서 이미지 전체를 다운받지 않아도 특정 레이어만 다운받을 수 있다.  
이것이 가능한 이유는 도커가 [유니온 파일 시스템](https://en.wikipedia.org/wiki/UnionFS)을 이용하여 여러 개의 레이어를 하나의 파일 시스템으로 사용할 수 있도록 지원하기 때문이다.

![No Image](/assets/images/posts/20191029/image-layer.png)   
출처: [subicura 도커란 무엇인가](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)

예를들어 ubuntu 이미지가 A, B, C 레이어의 집합이라면, ubuntu 를 base image 로 하여 만든 nginx 이미지는 A, B, C, nginx 레이어를 갖게 된다. webapp 이미지를 nginx 를 기반으로 만들면 A, B, C, nginx, web app source 레이어로 생성된다. webapp 을 수정한 경우 매번 A, B, C, nginx 레이어를 다시 설치하지 않고 web app source 레이어만 다시 다운받아 사용하는 것이 가능하다.  
컨테이너를 생성하면 기존 이미지 레이어 위에 Read-Write 레이어가 생성된다. Read-Write 레이어는 컨테이너를 실행중에 생성되거나 변경된 내용이 저장되는 영역이다.  
  
배포시에 수정된 내용만을 다시 받으면 되기 때문에 효과적이고 빠르게 배포가 가능해진다.

# 도커를 사용하는 이유  
- 코드를 통한 실행 환경 구축 및 애플리케이션 구성
- 변하지 않는 실행환경으로 멱등성(Indempotency) 확보
- 실행 환경과 애플리케이션 일체화로 이식성 향상
- 시스템을 구성하는 애플리케이션 및 미들웨어 관리 용이성 


## 코드를 통한 실행 환경 구축 및 애플리케이션 구성 - 코드로 관리하는 인프라
여러 서버에 어플리케이션을 배포하기 위해서는 애플리케이션에 의존하는 수 많은 라이브러리를 설치해야 한다. 이러한 라이브러리들을 수동으로 설치할 수 없기 때문에 일반적으로 쉘 스크립트를 작성하여 환경을 설정해 왔다. 하지만 쉘 스크립트로 클러스터 환경에 존재하는 수많은 서버에 동일한 환경을 구축하는것은 한계가 있다.  
  
이러한 한계점을 극복하기 위해서 나온 개념이 코드로 관리하는 인프라 개념이다. Chef, Puppet, Ansible 등은 프로비저닝 툴로써 인프라를 코드로 선언하고 모든 서버에 배포함으로써 환경을 동일하게 만들수 있다.  
그 중 Ansible 은 많이 사랑받고 있는 환경 자동화 도구인데, playbook 에 yaml 형식으로 환경을 코드로 선언하여 사용한다.  
  
도커도 Dockfile 에 환경을 코드로 작성함으로써 인프라를 코드로 관리한다.  

## 변하지 않는 실행환경으로 멱등성(Indempotency) 확보 - 불변 인프라 
하지만 코드로 관리하는 인프라에도 문제점이 존재하는데 인프라는 시간이 지나면서 변한다는 것이다. 도커는 이런 문제를 해결하기 위해서 불변 인프라 환경을 제공한다. 아래 예제를 보자    
  
애플리케이션의 트래픽이 높아져 이미 A 서버를 사용하고 있는데 새로 B 서버를 투입한다고 가정해보자. 
B 서버에 A 서버와 같은 환경을 구축했다고 하더라도 원하는 대로 동작하지 않는 경우가 흔하게 등장한다. A 서버와 B 서버를 구축한 시점이 다르기 때문에 두 서버에 설치된 OS, 컴파일러, 의존 패키지들까지 완벽하게 같기는 어렵다. 서로 모양이 다른 서버가 존재하는 것을 Snowflake Server(눈송이 서버) 라고 한다. 눈송이가 모두 모양이 다르듯 서버들도 서로 다른 모습이라는 것이다.  

> 
예를들어 개발환경인 mac os 업데이트로 이후 기존에 잘 동작하던 것들이 제대로 동작하지 않는 경험은 수 없이 해봤다. OS 뿐만 아니라 npm install 로 stable 버전을 설치해서 많이 사용하곤 하는데, stable 버전은 안정화된 버전이지 변경이 없는 버전은 아니라는 것이다. 

이러한 문제의 근본적인 원인은 인프라의 가변성(mutable infrastructure)을 허용하고 있기 때문이다.  
  
Ansible vs Docker  
![No Image](/assets/images/posts/20191029/ansible_vs_docker.png)   
출처: [44bits why-should-i-use-docker-container](https://www.44bits.io/ko/post/why-should-i-use-docker-container#%EC%9A%B4%EC%98%81%ED%95%98%EB%A9%B4%EC%84%9C-%EB%A7%8C%EB%93%A4%EC%96%B4%EC%A7%80%EB%8A%94-%EB%88%88%EC%86%A1%EC%9D%B4-%EC%84%9C%EB%B2%84%EB%93%A4snowflake-servers)

- Ansible(앤서블)은 provisioning 도구로써 인프라 환경을 코드로 관리하는 유명한 오픈소스
- 앤서블과 같은 provisioning 도구들은 실행 시점에 서버의 상태가 결정되지만 도커는 이미지를 이용해 구성 시점에 서버의 상태가 결정됨
- 따라서 도커는 항상 같은 실행 결과를 보장하는 불변 인프라(immutable infrastructure) 를 구축 가능
- 도커의 이미지는 도커의 운영 시점에 대한 snapshot 을 파일로 변환 한 것으로써 불변 인프라의 기반

## 실행 환경과 애플리케이션 일체화로 이식성 향상
기존에 서버에 애플리케이션을 배포하는 방식은 이미 구축된 서버에 배포되는 방식으로 장비가 추가되면 인프라를 복제하는 작업과 애플리케이션을 배포하는 작업이 분리되어 있었다.   
하지만 도커는 컨테이너라는 개념을 이용하여 인프라와 애플리케이션을 함께 묶는 것이 가능하다. 따라서 도커를 이용하면 장비 추가되더라도 도커 이미지만 빌드하여 얼마든지 배포가 가능하다.

## 시스템을 구성하는 애플리케이션 및 미들웨어 관리 용이성 
일정 규모를 넘는 시스템들은 주로 여러 개의 애플리케이션과 미들웨어를 조합하는 형태로 구성된다. 각 애플리케이션과 미들웨어를 조합하지 않으면 시스템을 구성할 수 없는 문제가 발생한다.  
도커는 여러 컨테이너를 한 덩어리 처럼 동작시키기 위해서 도커 컨테이너 오케스트레이션 시스템들이 존재한다. 도커는 여러 컨테이를 사용하는 애플리케이션을 쉽게 관리할 수 있도록 Docker Compose 라는 도구 제공한다. 도커 컴포즈를 이용하면 컨테이너를 정의하고 컨테이너 간의 의존 관계를 정의해서 시작 순서 등을 조작할 수 있다.  
도커는 여러 서버에 걸쳐 있는 여러 컨테이너를 관리할 수 있는 기능을 가진 컨테이너 오케스트레이션 도구인 
도커 스웜, 쿠버네티스 라는 도구들도 존재한다. 컨테이너 오케스트레이션 도구를 활용하면 컨테이너 증가/감소와 노드의 리소스를 효율적으로 사용하기 위해서 컨테이너 배치 및 로드 밸런싱 기능 등이 포함되어 있다. 

## Reference
- [위키북스 도커/쿠버네티스를 활용한 컨테이너 개발입문](https://wikibook.co.kr/docker-kubernetes/)
- [subicura 도커란 무엇인가](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
- [위키 도커](https://ko.wikipedia.org/wiki/%EB%8F%84%EC%BB%A4_(%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4))
- [Pycon Conference 에서 The future of Linux Containers 세션](https://www.youtube.com/watch?v=wW9CAH9nSLs&feature=youtu.be)
- [도커의 창업자가 도커를 퇴사하면서 남긴 포스팅](docker.com/blog/au-revoir/)
- [도커 레퍼런스 what-container](https://www.docker.com/resources/what-container)
- [구글 클라우드 containers](https://cloud.google.com/containers/?hl=ko)
- [redhat whats-a-linux-container](https://www.redhat.com/ko/topics/containers/whats-a-linux-container)
- [libcontainer](https://github.com/docker/libcontainer)
- [runc](https://github.com/opencontainers/runc)
- [위키 유니온 파일 시스템](https://en.wikipedia.org/wiki/UnionFS)
- [docker image](https://rampart81.github.io/post/docker_image/)
- [raccoony why-should-i-use-docker-container](https://www.44bits.io/ko/post/why-should-i-use-docker-container#%EC%9A%B4%EC%98%81%ED%95%98%EB%A9%B4%EC%84%9C-%EB%A7%8C%EB%93%A4%EC%96%B4%EC%A7%80%EB%8A%94-%EB%88%88%EC%86%A1%EC%9D%B4-%EC%84%9C%EB%B2%84%EB%93%A4snowflake-servers)