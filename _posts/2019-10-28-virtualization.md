---
title: "가상화(Virtualization)"
excerpt: ""
categories: 
tags:
  - 가상화
  - Virtualization
  - VM
  - Virtual Machine
  - Hypervisor
  - native hypervisor
  - hosted hypervisor
  - Full Virtualization
  - Para-Virtualization
last_modified_at: 2019-10-30
---


# 가상화(Virtualization) 란?
[위키의 정의](https://en.wikipedia.org/wiki/Virtualization)에서 가상화는 하드웨어 리소스를 추상화하여 가상버전을 만드는 행위라고 정의하고 있다.   

좀 말이 어려운데, 소프트웨어를 사용해 물리적인 하드웨어를 추상화하여 가상머신(VM)이라고 하는 여러 **가상 컴퓨터**를 만드는 것이다.  
가상화는 물리적 인프라에서 독립된 환경 만들 수 있도록 해준다.  
  
가상화는 클라우드 컴퓨팅의 근간이 된 기술인데, 클라우드 컴퓨팅에서 가상화를 통해 물리적 하드웨어를 추상화한 리소스를 사용자에게 제공하며 사용자는 필요한 컴퓨팅 리소스만 구매하여 사용가능 하다.


# 가상화 동작 방식 

가상화에서는 크게 3가지 요소로 나눌수 있는데 가상머신(VM), 하이퍼바이저(hypervisor), 하드웨어로 나누어진다. 

![No Image](/assets/images/posts/20191028/virtualization.png)   
출처: [redhat virtualization](https://www.redhat.com/ko/topics/virtualization)

## 가상머신(VM) 
- 가상머신은 내부에 OS를 갖는 가상 컴퓨터 시스템(하나의 독립된 운영환경)
- 하나의 물리적인 서버에 여러 VM 을 생성하여 여러 OS or 애플리케이션을 실행 가능 
- 여러 VM 은 하이퍼바이저에게 물리 리소스를 요청하고 하이퍼바이저는 해당 리소스를 할당하여 하드웨어 리소스를 공유함
- 각 VM 은 완벽히 독립되어 하이퍼바이저에 의해서 실행됨
- 가상머신은 하나의 데이터 파일로 한 컴퓨터에서 다른 컴퓨터로 이동하더라도 동일하게 동작 


## 하드웨어
- 하드웨어는 일반적으로 서버, 데스크탑과 같은 컴퓨터 자원을 말함 
- 하드웨어의 리소스는 하이퍼바이저에 의해서 특정 VM 할당되거나 해제된다


## 하이퍼바이저(Hypervisor)
- VMMM(Virtual Machine Monitor or Virtual Machine Manager) 이라고도 함
- 하이퍼바이저는 가상머신을 생성하고 실행하기 위한 소프트웨어로 하드웨어와 VM 중간 계층에 위치하여 중간 관리자 역할을 수행
- 하이퍼바이저는 일반적으로 서버와 같은 하드웨어에 직접 설치되어 실행됨 
- 하이퍼 바이저는 2가지 유형으로 분류된다. 

### 네이티브 하이퍼바이저(native or bare-metal hypervisor)

![No Image](/assets/images/posts/20191028/native_hypervisor.png)   
출처: [위키 Hypervisor](https://en.wikipedia.org/wiki/Hypervisor)  

- 하이퍼바이저가 하드웨어 바로 위에서 실행되는 방식으로 CPU, 메모리, 스토리지 등을 직접제어함
- 하이퍼바이저가 호스트 OS를 대신함 
- 장점: 하드웨어를 직접 제어하기 때문에 매우 효율적이고 별도의 호스트 OS가 없으므로 보안이 향상됨
- 단점: 하드웨어에 직접 설치하기 때문에 설치가 어렵고 상황에 따라 별도의 관리 시스템이 필요할 수도 있음 
- 대표적으로 Xen, 마이크로소프트 Hyper-V, KVM(Kernel-based Virtual Machine) 등이 있음 

### 호스트된 하이퍼바이저(hosted hypervisor)

![No Image](/assets/images/posts/20191028/hosted_hypervisor.png)   
출처: [위키 Hypervisor](https://en.wikipedia.org/wiki/Hypervisor)  

- 하이퍼바이저가 호스트 OS 위에 애플리케이션으로 설치되는 방식
- 게스트 OS(VM)는 하드웨어 자원을 하이퍼바이저를 통해서 호스트 OS에서 할당받음 
- 일반적으로 서버기반 환경이 아닌 개인 PC 나 노트북 환경에서 수행되는 방식 
- MAC OS 위에 VM Ware 를 설치하여 윈도우나 리눅스를 구동시키는 경우
- 장점: 호스트 OS에서 게스트 OS에 빠르고 쉽게 접근이 가능
- 단점 
    * 호스트 OS를 통해 하드웨어 자원을 액세스하므로 네이티브 방식에 비해서 성능이 나쁨 
    * 해커에 의해 호스트 OS가 손상된 경우 게스트 OS를 조작할 수 있어 보안 위험이 있음
- 대표적으로 VMware server, VMware Workstation, Virtual box 등이 있음 


# 가상화 방법
가상화 방법은 전 가상화(Full Virtualization)와 반 가상화(Para-Virtualization) 방법이 있다.  

## 전 가상화(Full Virtualization)
- 전 가상화는 하드웨어를 모두 가상화하는 방법  
- 게스트 OS(VM)는 하드웨어 자원을 사용하기 위해서 반드시 하이퍼바이저가 중재해야 됨

![No Image](/assets/images/posts/20191028/full_virtualization.png)   
출처: [가비아 서버 가상화 기술의 진화](https://library.gabia.com/contents/infrahosting/7426) 

- 게스트 OS는 'DOM 0' 이라는 관리 머신을 거쳐서 하이퍼바이저와 통신 
- 'DOM 0' 는 여러 게스트 OS의 요청을 모두 받아서 처리함으로 CPU, RAM, I/O 처리가 잦으면 성능이 저하됨

## 반 가상화(Para-Virtualization) 
- 반 가상화는 하드웨어를 완전히 가상화하지 않음
- 게스트 OS(VM)의 커널을 일부 수정하여 'DOM 0' 없이 하이퍼바이저를 통해서 하드웨어와 직접 통신
- 게스트 OS를 수정해야 하므로 윈도우 같은 경우 별도의 Tool 이 필요할 수 있음

![No Image](/assets/images/posts/20191028/para_virtulalization.png)   
출처: [가비아 서버 가상화 기술의 진화](https://library.gabia.com/contents/infrahosting/7426) 

- 수정된 게스트 OS는 Hyper Call 이라는 명령어로 하이퍼바이저를 호출
- 하이퍼바이저는 하드웨어 자원을 수정된 게스트 OS에 할당 

# 가상화 사용 사례 

## 데스크탑 가상화
- 데스크탑 가상화는 데스크탑 환경과 실제 클라이언트 장치를 분리하는 기술 
- 데스크탑 환경이 데이터 센터에 저장되고 사용자는 로그인을 통해 데스크탑 환경을 이용하는 것
- 대표적으로 VDI(Virtual Desktop Infrastructure), DaaS 가 있음 
- VDI: 사용자는 VDI 접속 프로그램에 로그인하여 원격 서버의 데스크탑 환경을 이용하는 방법
    * 중앙 집중식 데스크탑 관리가 가능함으로써 보안이 향상되어 많은 기업에서 사용하고 있는 솔루션 

## 서버 가상화 
- 단일 물리적인 서버를 여러 개의 가상 서버로 분리하여 사용하는 기술
- 서버의 사용률을 극대화하기 위해서 사용


## 네트워크 가상화 
- 네트워크 가상화는 물리적인 네트워크를 논리적인 단위의 네트워크로 나누어 사용하는 기술
- 네트워크 가상화는 네크워크에 있는 두 도메인을 물리적으로 연결하지 않고 터널을 만들어 두 도메인을 연결
- 네트워크 관리자는 물리적인 요소(스위치, 라우터, 링크 등)들을 건드리지 않고 요소들을 수정하고 제어할 수 있음
- 대표적으로 SDN(Software Define Network), NFV(Network Functional Virtualization) 가 있음 
- NFV: 네트워크 가상화로 만들어진 터널에 서비스를 배치하는 것으로 방화벽, IDS, IPS 와 같은 4~7 계층의 로드밸런싱을 가상화 하는 것
- SDN: 제어 플레인을 데이터 플레인과 분리하는 방법으로 네트워크를 프로그래밍 가능하도록 함
    * 제어 플레인: 네트워크에 무엇이 어디에 이동할지 알려주는 역할
    * 데이터 플레인: 특정 목적지로 패킷을 보내는 역할

## 스토리지 가상화
- 네트워크 상에 있는 모든 스토리지를 단일 스토리지 처럼 액세스하고 관리할 수 있도록 함
- 모든 스토리지 블록을 단일 공유 풀로 통합하여 네트워크의 모든 VM에 할당할 수 있음
- 스토리지 가상화를 통해서 네트워크 안에 스토리지를 최대한 활용 가능

## 데이터 가상화
- 데이터 가상화는 데이터 소스, 형식, 위치에 관계없이 모든 애플리케이션들이 데이터에 접근가능 하도록 함

## 애플리케이션 가상화
- 애플리케이션을 사용자의 OS에 직접 설치하지 않고 가상환경의 애플리케이션을 실행
- 가상환경에는 오직 애플리케이션만을 설치하므로 위의 데스크탑 가상화와는 다름 
- 애플리케이션 가상화의 3가지 유형
    * 로컬 애플리케이션 가상화
    * 애플리케이션 스트리밍
    * 서버기반 애플리케이션 가상화

## CPU 가상화
- CPU 가상화는 하이퍼바이저, virtual system, OS를 가능하게 하는 기본 기술
- 단일 VM 을 여러 VM 에서 사용할 수 있도록 여러 가상 CPU 로 나눌수 있음
- 최근 프로세서들은 CPU 가상화를 지원하는 명령어 세트들이 포함되어 있음 

## 리눅스 가상화
- 리눅스에는 Intel 및 AMD의 가상화 프로세서 확장을 지원하는 KVM(Kernel Virtualization Machine)이 자체적으로 포함되어 있음
- 리눅스는 호스트 OS 내에서 x86기반 VM을 생성가능 

# 컨테이너형 가상화
- 최근 가상화 기술은 하이퍼바이저 기반 가상화 방식에서 컨테이너 기반 가상화로 변하고 있음 (도커) 
- 기존의 하이퍼바이저 기반의 가상화 방식에서는 VM 에 추가적으로 게스트 OS를 설치하기 때문에 성능 이슈가 존재했음  
- 이를 개선하기 위해 LXC(Linux Container) 에서 컨테이너라는 개념을 처음 도입
- LXC 에서는 게스트 OS를 설치하지 않고 **프로세스를 격리** 하여 VM과 유사한 독립적인 공간을 컨테이너라고 표현함
- LXC는 호스트 OS(리눅스 커널) 위에서 네임 스페이스와 cgroup 을 이용하여 구현됨
    * 네임 스페이스: 리눅스 시스템 자원을 묶어 프로세스를 할당하는 방식
    * cgroup: 프로세스 그룹의 시스템 자원을 사용량을 관리하여 특정 애플리케이션이 자원을 과다 사용하는 것을 제한 

![No Image](/assets/images/posts/20191028/vm_vs_container.png)   
출처: [가비아 서버 가상화 기술의 진화](https://library.gabia.com/contents/infrahosting/7426) 

- 컨테이너 엔진에는 실행에 필요한 환경이 모두 갖춰져 있으므로 어떤 환경에서도 실행이 가능
- 컨테이너에는 게스트 OS는 존재하지 않아서 VM 보다 가볍고 OS를 부팅하는 과정이 없어서 빠르게 시작이 가능


## Reference
- [위키 Virtualization](https://en.wikipedia.org/wiki/Virtualization)
- [IBM Virtualization](https://www.ibm.com/cloud/learn/virtualization-a-complete-guide)
- [VM Ware Virtualization](https://www.vmware.com/kr/solutions/virtualization.html)
- [redhat virtualization](https://www.redhat.com/ko/topics/virtualization)
- [redhat what-is-virtualization](https://www.redhat.com/ko/topics/virtualization/what-is-virtualization)
- [opensource.com virtualization](https://opensource.com/resources/virtualization)
- [위키 Hypervisor](https://en.wikipedia.org/wiki/Hypervisor) 
- [가비아 서버 가상화 기술의 진화](https://library.gabia.com/contents/infrahosting/7426) 
- [ibm hypervisors](https://www.ibm.com/cloud/learn/hypervisors) 
- [ibm virtualization-a-complete-guide](https://www.ibm.com/cloud/learn/virtualization-a-complete-guide) 
- [위키 Desktop_virtualization](https://en.wikipedia.org/wiki/Desktop_virtualization)  
- [TechTarget Differences-between-desktop-and-server-virtualization](https://searchvirtualdesktop.techtarget.com/feature/Differences-between-desktop-and-server-virtualization)
- [불곰님 블로그 네트워크 가상화](https://brownbears.tistory.com/60)
