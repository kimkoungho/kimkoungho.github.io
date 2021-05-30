# kubernates

## 컨테이너 오케스트레이션 이란? 
쿠버네티스는 컨테이너 오케스트레이션 도구 중 하나인데, 먼저 컨테이너 오케스트레이션에 대해서 알아보자 
   
컨테이너 오케스트레이션은 컨테이너의 배포, 관리, 확장(scale out), 네트워킹 등을 자동화 하는 도구이다 
도커의 등장으로 서버 관리를 컨테이너를 이용하여 관리가 쉬어졌음   
  
하지만 Containerization 만으로는 한계가 존재했음 
- 배포 문제
도커 컨테이너 위에 돌아가는 어플리케이션을 실행하기 위해서 각 서버에 ssh 로 접속하여 docker stop, run 등을 실행 해주어야 함  
> 온프로미스 환경과 동일하다고 보면 될 것 같다  

배포된 어플리케이션을 롤백하려면 이전 버전의 도커 이미지를 수동으로 실행해야함 -> 빠른 롤백의 어려움 

- 서비스 검색(Service Discovery) 문제 
일반적으로 운영환경에서는 로드 밸런서를 이용하여 서비스를 구축하는데, 스케일 아웃이 필요하면 로드 밸런서의 설정 업데이트가 필요   
ㄴ 새로 등록된 서버 IP 를 추가 해주어야 함 

- 서비스 장애,부하 모니터링의 문제
예상치 못한 부하에 대한 대응을 일일히 사람이 해주어야 함 
서비스 장애도 사람이 수동으로 모니터링하여 대응해야 함   

이러한 문제점들을 해결하기 위해서 컨테이너 환경을 효율적으로 관리하기 위해서 등장한 기술이 컨테이너 오케스트레이션이다 

## 쿠버네티스의 등장   
쿠버네티스는 다른 오케스트레이션 도구들에 비해서 비교적 늦게 등장했는데, 기존에 도커 진영에서 개발된 [도커 스웜](https://docs.docker.com/engine/swarm/) 이 쉽고 간단한 사용법으로 세력을 넓히고 있었고 
AWS 에서는 ECS, Hashicorp 에서는 [Nomad](https://www.nomadproject.io/), Mesos 에서는 [Marathon](https://mesosphere.github.io/marathon/) 을 발표했었으나 
구글의 컨테이너 관리 노하우가 녹아있는 쿠버네티스가 등장하고 나서는 쿠버네티스가 압도적인 점유율을 자랑하게 되었다.
클라우드의 대명사인 AWS, Google Cloud, Azure 등 클라우드 환경에서도 쿠버네티스 관리형 서비스를 내놓은 것만 봐도 그 위상을 알 수 있다

> 쿠버네티스 vs 도커 스웜  
> 쿠버네티스는 스웜 보다 더 많은 기능을 갖춘 컨테이너 **오케스트레이션 시스템** 으로 컨테이너 런타임을 다룰 수 있다 

## 쿠버네티스 란 ? 
**쿠버네티스는 컨테이너를 쉽고 빠르게 배포/확장(scale out) 하고 관리를 자동화 해주는 오픈소스 플랫폼이다**
- 쿠버네티스라는 명칭은 키잡이(helmsman) / 파일럿(pilot)을 뜻하는 그리스어에서 유래했다고 함  
- 주로 컨테이너 오케스트레이션 도구라고 불리며 구글 주도로 개발되었음  
- 컨테이너들을 다루기 위한 API 및 CLI 환경을 지원함  
- 단순히 컨테이너 관리 플랫폼을 넘어서 MSA, 클라우드 플랫폼을 지향하고 손쉽게 관리할 수 있는 그릇역할을 수행 

## 쿠버네티스의 특징 
- 서비스 디스커버리와 로드 밸런싱 
- 스토리지 오케스트레이션 
- 자동화된 롤아웃과 롤백 
- 자동화된 빈 패킹(bin packing)
- 자동화된 복구(self-healing)
- 시크릿과 구성 관리 
 


- 쿠버네티스를 이용하면 컨테이너를 이용한 어플리케이션 배포와 클라우드 상의 컨테이너의 배치, 스케일링, 로드 밸런싱 등의 여러가지 기능을 사용할 수 있음 

  
    





## 래퍼런스 
- [공식 도큐 - 쿠버네티스란 무엇인가?](https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/)
- [위키북스 도커/쿠버네티스를 활용한 컨테이너 개발입문](https://wikibook.co.kr/docker-kubernetes/)
- [subicura - 쿠버네티스 기본](https://subicura.com/2019/05/19/kubernetes-basic-1.html)
- [Redhat container orchestration](https://www.redhat.com/ko/topics/containers/what-is-container-orchestration)
- [modolee 블로그 - kubernetes-newbie-guide-01](https://velog.io/@modolee/kubernetes-newbie-guide-01)
