---
title: "쿠버네티스(kubernates) 아키텍쳐"
excerpt: ""
categories: devops
tags:
    - kubernates
    - kubernates architecture
last_modified_at: 2022-02-01
---

# kubernates architecture

## 쿠버네티스 클러스터 아키텍처
- 쿠버네티스의 아키텍쳐는 서비스 검색을 위해 유연하고 느슨하게 결합된 메커니즘을 제공 
- 대부분의 분산 컴퓨팅 플랫폼과 마탄가지로 쿠버네티스 클러스터는 하나 이상의 마스터와 여러 컴퓨팅 노드로 구성됨

![kubernetes_architecture](/assets/images/posts/20201201/kubernetes_architecture.png)<br/>
출처: [https://tkssharma.com/kubernetes_architecture/](https://tkssharma.com/kubernetes_architecture/)
<br/>

### 마스터 노드 
- 마스터 노드는 전체 쿠버네티스 시스템을 제어하고 관리하는 역할을 수행
- 쿠버네티스 클러스터를 제어하는 컨트롤 플레인(Control plane)이 실행됨  
- 마스터 노드는 API 제공하고 개발자 or 관리자는 해당 API(kubelet, UI) 를 이용해 클러스터를 관리 
- 마스터 노드 내부에 API 서버는 외부 API 요청과 컨트롤 플레인 구성 요소들과 통신
  - 쿠버네티스 관리를 위한 리소스 항목과 인증, 권한 부여, 엑세스 제어, API 등록 및 검색을 위한 메커니즘을 제공 
- 스케줄러는 어플리케이션의 배포를 담당 
  - 스케줄링 정책에 따라서 워커 노드에 pod 를 스케줄링
  - => 어플리케이션의 배포 가능한 구성 요소를 워커 노드에 할당
- 컨트롤로 매니저는 구성 요소의 복제본, 워커 노드 추적, 노드 장애 처리 등과 같은 클러스터 단의 기능을 수행 
- etcd 는 클러스터 구성에 대한 정보를 저장하는 분산 데이터 저장소 
  - SSOT(Single Source of Truth) 역할을 수행 
  - CoreOS 의 오픈소스 분산 키-값 데이터 베이스 
- 마스터는 etcd 를 쿼리하여 노드, pod 의 상태의 다양한 매개변수를 검색함 

![kubernetes_label_selector](/assets/images/posts/20201201/kubernetes_label_selector.png)<br/>
출처: [https://tkssharma.com/kubernetes_architecture/](https://tkssharma.com/kubernetes_architecture/)
<br/>

### 워커 노드
- 워커 노드는 실제 배포되는 컨테이너 어플리케이션을 실행함 
- 각 워커 노드는 마스터와 통신을 위한 쿠버네티스 에이전트 (kubelet) 존재
  - kubelet 은 터케니어늬 life cycle 관리하고 볼륨(CSI) 및 네트워크(CNI) 관리 
- 컨테이너 실행을 위한 컨테이너 런타임(docker)
- 어플리케이션 구성 요소 간의 네트워크 트래픽을 로드밸런싱 하는 쿠버네티스 서비스 프록시(kube-proxy)
- 워커 노드는 로깅, 모니터링, 서비스 검색 및 추가 기능을 위한 추가 구성요소도 실행될 수 있음

### pod
- 위 그림에서 1개의 원이 1개의 pod  
- pod 는 하나 이상의 컨테이너 모음으로 쿠버네티스의 핵심 관리 단위로 동일한 컨테스트와 리소스를 공유하는 컨테이너의 논리적 경계 역할을 수행함 
- pod 그룹화 메커니즘은 여러 종속 프로세스를 함께 실행 할 수 있도록 한다 
- pod 는 service 를 통해 내부 or 외부에서 접근 될 수 있음 
- pod 는 label, selector 라는 key-value 를 통해 service 에 연결됨 
- service 는 지정된 selector 와 일치하는 label 을 가진 pod 를 자동으로 검색함

### Label and Selector
Label : 쿠버네티스 오브젝트를 식별하기 위한 메타 데이터로 key-value 쌍으로 구성됨<br/>
Selector : 쿠버네티스의 그룹화에 사용되는 핵심<br/>
- Label Selector 를 이용해서 쿠버네티스 객체들을 그룹화하여 관리 할 수 있음

label, selector, namespace 는 쿠버네티스 리소스들을 유연하고 동적으로 관리하는데 도움을 준다<br/>
- Label Selector 는 namespace 내에서 유일해야 함 

쿠버네티스 자체는 분산 아키텍처를 기반으로 하므로 MSA 구축하고 관리하는데 큰 도움을 준다<br/>

## 오케스트레이션 layer
컨테이너 오케스트레이션은 컨테이너의 배포, 관리, 확장(scale out), 네트워킹 등을 자동화 하는 도구로 [이전 글](https://kimkoungho.github.io/devops/kubernates/)에서 설명했다<br/>

오케스트레이션 layer 에서 수행하는 것들 
- 컨테이너의 스케줄링 수행 
- 컨테이너가 중지되면 restart 
- 컨테이너 네트워크 활성화 
- 필요에 따라 컨테이너 및 관련 리소스를 확장 or 축소 
- 서비스 discovery

## 컨테이너 네트워킹 
- 컨테이너 간의 네트워킹은 컨테이너 오케스트레이션에서 까다로운 문제 중 하나
- 여기서는 docker 가 컨테이너 네트워킹을 처리하는 방법과 kubernetes 가 네트워킹을 처리하는 방법을 비교해본다 

### docker 네트워킹 
- docker 컨테이너는 host-private 네트워킹을 사용
- 이를 위해서 docker 는 virtual bridge 로 불리는 ```docker0``` 인터페이스를 호스트에 프로비저닝 함 
- virtual bridge 에 연결하기 위해 docker 는 ```veth``` (가상 이더넷 장치) 를 할당하고 ```eth0``` NAT 를 통해 IP 를 매핑 
- NAT 는 패킷의 IP 헤더에 있는 네트워크 주소 정보를 수정하여 하나의 IP 를 내부 IP 로 매핑하는 방법  
- 이런 docker 네트워킹은 동일한 머신 or 동일한 virtual bridge 를 갖는 컨테이너만 통신 할 수 있는 문제가 있음 
- 많은 호스트와 시스템이 관련된 서비스에서는 위 제약사항은 문제가 됨 
- 그리고 NAT 에 대한 의존도를 무시할 수 없기 때문에 성능 저하로 이어질 수 있음 

### kubernetes 네트워킹 
- 쿠버네티스를 사용한 네트워킹은 docker 네트워킹을 사용하는 것 보다 **성능과 확장성을 높이기 위한 것** 
- 모든 컨테이너는 NAT 없이 다른 컨테이너와 통신 할 수 있음 
- 컨테이너는 다른 컨테이너가 참조하는 데 사용하는 것과 동일한 IP 로 자신을 참조


## 래퍼런스 
- [쿠버네티스_인_액션_도서](http://www.yes24.com/Product/Goods/89607047)
- [https://tkssharma.com/kubernetes_architecture/](https://tkssharma.com/kubernetes_architecture/) 


