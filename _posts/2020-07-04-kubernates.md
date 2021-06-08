---
title: "쿠버네티스(kubernates) 개념"
excerpt: ""
categories: devops
tags:
    - kubernates
    - kubernates 개념 
    - kubernates object 
last_modified_at: 2021-06-06
---

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
- 서비스 디스커버리와 로드 밸런싱 : DNS 이름을 사용하거나 자체 IP 주소를 이용하여 컨테이너를 노출 가능함     
- 스토리지 오케스트레이션 : 로컬 저장소, 클라우드 저장소 등을 연결하여 사용 가능함 
- 자동화된 롤아웃과 롤백 : 쿠버네티스로 배포된 컨테이너는 원하는 상태 (desired state) 를 서술할 수 있으며 현재 상태를 원하는 상태가 되도록 한다  
ex) 배포시 마다 매번 새로운 컨테이너를 띄우도록 자동화하면 기존 컨테이너를 제거하고 새로운 컨테이너에 모든 리소스를 적용할 수 있다  
- 자동화된 빈 패킹(bin packing) : 컨테이너화 작업을 실행하는데 사용할 수 있는 쿠버네티스 클러스터 노드를 제공함  
각 컨테이너가 필요로 하는 CPU 와 메모리를 쿠버네티스에게 지시하면 쿠버네티스는 컨테이너를 노드에 맞추어서 리소스를 잘 사용할 수 있도록 해줌 
- 자동화된 복구(self-healing) : 쿠버네티스는 실패한 컨테이너를 다시 시작하고, 컨테이너를 교체하며, 응답하지 않는 컨테이너를 죽이는 등 원하는 상태가 되도록 노력한다  
서비스가 준비가 되기 전까지 클라이언트에게 알려주지 않음  
- 시크릿과 구성 관리 : Oauth 토큰 및 SSH 키와 같은 중요한 정보를 저장하고 관리할 수 있음  
컨테이너 이미지를 재구성하지 않고 스크릿 및 애플리케이션 구성을 배포 및 업데이트 가능함 


## 쿠버네티스의 기본 개념  

### Desired State 
![Desired_State](/assets/images/posts/20200704/desired-state.png)  
- desired state (원하는 상태) 는 쿠버네티스의 가장 중요한 개념중 하나로 관리자가 바라는 환경을 의마함 
- 관리자는 원하는 서버의 대수, 포트 등의 상태를 정의할 수 있음 
- 쿠버네티스는 현재 상태 (current state) 를 모니터링 하면서 관리자가 정의한 원하는 상태(desired state) 로 유지하려고 내부적으로 이러저런 처리를 수행함 
- 이런 개념때문에 실제 배포시에도 어떤 동작을 명령하는 방식이 아닌 상태를 선언하는 방식을 사용한다 
- ex) nginx 를 실행하라는 명령(imperative) 대신에 nginx 컨테이너 1개를 유지해달라는 상태를 선언(declarative) 한다 
- 이런 차이는 명령에서도 알 수 있음  
 
```shell script
$ docker run # 명령   
$ kubectl create # 원하는 상태 생성 
```

**쿠버네티스의 핵심은 상태이며 쿠버네티스를 사용하려면 어떤 상태들이 있고 어떻게 상태를 선언해야 하는지 알아야 함** 

### 쿠버네티스 Object
[공식_쿠버네티스 Object](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/kubernetes-objects/)  
쿠버네티스는 상태를 나타내기 위해서 오브젝트를 이용하며 구체적으로는 아래와 같음   
- 어떤 컨테이너화된 애플리케이션이 동작 중인지 (+어느 노드인지)
- 그 애플리케이션이 사용가능한 리소스 
- 그 애플리케이션의 재구동 정책, 업그레이드, 문제 발생시 정책 등   
쿠버네티스에는 여러가지 상태를 나타내기 위한 여러 오브젝트를 제공하고 있으며 새로운 오브젝트 추가가 쉬워 확장성이 좋음  

쿠버네티스 오브젝트는 상태 외에도 명세(spec) 가 있는데 이것은 오브젝트에 대한 정보를 yaml 로 정의 하고 원하는 상태를 기술하는데 사용함
- 오브젝트의 명세는 생성, 조회, 삭제로 관리할 수 있어서 REST API 로 쉽게 노출이 가능함    

> 쿠버네티스 오브젝트는 다양한 종류가 있는데, 여기서는 자주 사용되는 몇 가지만 소개함 
 
### Pod 
[공식_쿠버네티스 Pod](https://kubernetes.io/ko/docs/concepts/workloads/pods)  

![node_pod](/assets/images/posts/20200704/node_pod.png)    
- 파드는 쿠버네티스 애플리케이션의 배포가능한 가장 작은 단위로 한 개 이상의 컨테이너와 스토리지, 네트워크 속성이 있음
- 파드는 노드에 배치되는 컨테이너 그룹의 논리적인 개념이라고 보면됨 
- Pod 에 속한 컨테이너들은 Pod 의 네트워크와 스토리지를 공유하고 localhost 로 서로 접근이 가능함  
- Pod 의 네트워크 인터페이스에는 Node 의 IP 와 연결되기 위한 IP 가 존재함  
- 실제 어플리케이션은 Pod 내부의 container 로 구동되기 때문에 외부에서 접근 할 수 있는 방법을 정의해야 한다     
 
### ReplicaSet 
[공식_쿠버네티스 ReplicaSet](https://kubernetes.io/ko/docs/concepts/workloads/controllers/replicaset/)  
Pod 를 여러 개 복제하여 생성하고 관리하는 오브젝트  
레플리카 세트를 이용하면 지정된 수의 레플리카 파드가 유지되도록 보장해줌  
일반적으로 레플리카 세트를 직접 사용하기 보다는 아래 나오는 디플로이먼트(Deployment) 를 사용함     
> 공식 도큐에서도 디플로이먼트 사용을 권장하고 있다  

### Deployment
[공식_쿠버네티스_Deployment](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)    
디플로이먼트는 레플리카 셋의 상위 개념으로 추가적인 유용한 기능들을 제공하는 오브젝트  
ㄴ 파드 개수를 유지 해줄 뿐만 아니라 배포 작업을 좀 더 세분화 하여 관리가 가능함  
ㄴ ex) 롤링 업데이트, 블루/그린 배포 등      
디플로이먼트에서 의도하는 상태를 기술하면 디플로이먼트 컨트롤러는 현재 상태를 의도한 상태가 되도록 조정한다  
ㄴ 일반적으로 애플리케이션 배포에 기본이 되는 단위라고 볼 수 있음    

디플로이먼트를 사용해서 애플리케이션을 배포하게 되면 새로운 레플리카 세트가 생성되며 기존에 있던 레플리카 세트는 교체된다    
- pod 의 갯수만 수정한 경우(scale out)에는 기존 레플리카 세트를 그대로 유지한다  
  
디플로이먼트는 레플리카와 가장 큰 차이점으로 **리비전** 을 갖는데, 이는 배포에 대한 버전이라고 생각하면 된다   
디플로이먼트가 업데이트 되면 리비전도 업데이트 된다   
ㄴ ex) REVISION=1 -> REVISION=2  
롤백을 수행한 경우에 쿠버네티스는 이 리비전을 이용하여 롤백을 수행한다    

### Service
[공식_쿠버네티스_Service](https://kubernetes.io/ko/docs/concepts/services-networking/service/)  
네트워크와 관련된 오브젝트로써 파드를 외부 네트워크와 연결해주고 여러 개의 Pod 를 바라보는 내부 로드 밸런서를 생성할 때 사용한다  

말이 좀 어려운데, 왜 필요한지 먼저 살펴보자   
- 쿠버네티스의 파드는 비영구적인 리소스로 디플로이먼트를 이용하여 배포를 수행하면 동적으로 파드가 생성되거나 제거 될 수 있음  
- 각 파드는 고유한 IP 주소를 갖지만, 디플로이먼트 업데이트를 수행하면 기존의 파드는 교체 된다
- 이것은 파드의 집합을 IP 만으로 찾을 수 없음을 의미한다 > 서비스 디스커버리 불가능     
    
서비스 오브젝트를 이용하면 클러스터 내부/외부에서 클러스터 상에 존재하는 파드 집합을 찾을 수 있도록 서비스 디스커버리 역할을 수행함    
ㄴ 서비스 Object 는 일반적으로 파드 집합을 식별하기 위한 **셀렉터** 가 필요함  
ㄴ 헤드리스 서비스(headless service) 라는 셀렉터를 지정하지 않는 서비스도 존재함  

서비스 오브젝트의 Type 
- ClusterIP   
서비스 생성시 type 을 ClusterIP 로 지정하여 클러스터의 고유한 IP 를 지정 가능  
ㄴ type 의 기본값  
ClusterIP 는 클러스터 내부에 IP 를 노출시키며 외부로 부터는 접근 할 수 없다   
클러스터 내부 노드나 파드에서 ClusterIP 를 이용해서 서비스에 연결되어 파드에 접근이 가능하다   

- NodePort  
고정 포트로 각 노드의 IP 에 서비스를 노출 시킬 수 있으며 외부에서 접근이 가능하다
ㄴ 외부에서 <노드 IP>:<NodePort> 로 접근 가능      
NodePort 타입은 자동으로 ClusterIP 가 생성됨 
ㄴ 실제로 노드 1 에만 파드가 떠있고 노드 2 에는 파드가 없어도 노드2 IP 를 이용해서 접근해도 노드 1의 파드가 응답이 가능  

- LoadBalancer  
클라우드 공급자가 제공하는 로드 밸런서를 사용하여 서비스를 외부에 노출 시킴  
외부 로드 밸런서가 라우팅되는 NodePort 와 ClusterIP 가 자동으로 생성됨  
해당 로드 밸런서의 IP 를 이용해서 외부에서 접근이 가능함 

- ExternalName  
서비스를 ExternalName 값이랑 매칭하는 역할을 수행  
클러스터 내부에서 외부로 접근할 때 주로 사용됨  
이 서비스로 접근하면 설정한 CNAME 값으로 연결되어 클러스터 외부로 접근이 가능함  
내부에서 외부로 접근할 때 사용하는 값이기 때문에 **셀렉터가 필요 없음**
 
> Ingress 를 이용하여 서비스를 노출시킬 수 있는데, 서비스 유형은 아니지만 클러스터의 진입점 역할을 수행함 
  
### Ingress
[공식_쿠버네티스_Ingress](https://kubernetes.io/ko/docs/concepts/services-networking/ingress/)  
인그레스는 클러스터 내의 서비스에 대한 외부 접근을 관리하는 API 오브젝트로 일반적으로 HTTP 를 관리    

인그레스의 간단한 예시  
![kubernetes_ingress](/assets/images/posts/20200704/kubernetes_ingress.png)    
- 인그레스는 외부에서 서비스로 접속 가능한 URL, 로드밸런스 트래픽, SSL/TLS 종료, 이름 기반의 가상 호스팅을 제공하도록 구성 가능함  
- 인그레스가 위의 기능들을 정의 해둔 자원이라면 인그레스 컨트롤러는 정의한 대로 동작하게 해주는 관리자  
- 클라우드 상에서는 일반적으로 로드 밸런서 서비스들과 인그레스를 연동해 주지만, 직접 인그레스를 사용하기 위해서는 인그레스 컨트롤러를 인그레스와 직접 연동해주어야 함  
  
### Volume
[공식_쿠버네티스_Volume](https://kubernetes.io/ko/docs/concepts/storage/volumes/)  
Volume 은 저장소와 관련된 오브젝트다  
  
볼륨이 필요한 이유   
- 컨테이너의 특성 상 어떤 문제가 발생하여 컨테이너가 삭제되면 데이터도 같이 삭제된다
- ex) 컨테이너 런타임에 생성된 로그 파일 
- 컨테이너가 종료되더라도 유지해야 하는 파일을 저장하기 위해서 볼륨을 이용해야 함  
  
볼륨은 호스트 디렉토리를 그대로 사용할 수 있고 EBS 같은 스토리지를 동적으로 생성하여 사용 가능함  
- 임시 볼륨인 Pod 의 볼륨과 영구 저장용 볼륨인 퍼시스턴트 볼륨 등이 있음  


## 래퍼런스 
- [공식_쿠버네티스란 무엇인가?](https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/)
- [위키북스 도커/쿠버네티스를 활용한 컨테이너 개발입문](https://wikibook.co.kr/docker-kubernetes/)
- [subicura - 쿠버네티스 기본](https://subicura.com/2019/05/19/kubernetes-basic-1.html)
- [Redhat container orchestration](https://www.redhat.com/ko/topics/containers/what-is-container-orchestration)
- [modolee 블로그 - kubernetes-newbie-guide-01](https://velog.io/@modolee/kubernetes-newbie-guide-01)
- [공식_쿠버네티스 Object](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/kubernetes-objects/)
- [공식_쿠버네티스 Pod](https://kubernetes.io/ko/docs/concepts/workloads/pods) 
- [공식_쿠버네티스 ReplicaSet](https://kubernetes.io/ko/docs/concepts/workloads/controllers/replicaset/)
- [공식_쿠버네티스_Deployment](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)
- [공식_쿠버네티스_Service_doc](https://kubernetes.io/ko/docs/concepts/services-networking/service/)
- [https://arisu1000.tistory.com/27838](https://arisu1000.tistory.com/27838)
- [공식_쿠버네티스_Ingress](https://kubernetes.io/ko/docs/concepts/services-networking/ingress/)
- [공식_쿠버네티스_Volume](https://kubernetes.io/ko/docs/concepts/storage/volumes/) 