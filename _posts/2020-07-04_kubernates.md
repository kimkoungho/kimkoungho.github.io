# kubernates

## 쿠버네티스 란 ? 
[쿠버네티스 공식 도큐](https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/) 에 따르면, 쿠버네티스는 컨테이너화된 워크로드와 서비스를 관리하기 위한 오픈소스 플랫폼  
- 쿠버네티스라는 명칭은 키잡이(helmsman) / 파일럿(pilot)을 뜻하는 그리스어에서 유래했다고 함  
- 주로 컨테이너 오케스트레이션 도구라고 불리며 구글 주도로 개발되었음  
- 컨테이너들을 다루기 위한 API 및 CLI 환경을 지원함  
- 쿠버네티스를 이용하면 컨테이너를 이용한 어플리케이션 배포와 클라우드 상의 컨테이너의 배치, 스케일링, 로드 밸런싱 등의 여러가지 기능을 사용할 수 있음  
  
오케스트레이션은 기존에 사용하던 app01, db01, cache01 과 같이 목적에 맞게 서버를 분리하는 개념과는 달리 server1, 2, 3 등에 각 목적에 맞는 app01, db01 컨테이너를 적절히 배치하도록 배포하는 방식으로 
특정 어플리케이션에 부하가 생기거나 장애가 발생하면 해당 어플리케이션에 해당하는 컨테이너를 늘림으로써 장애를 방지할 수 있는 기술  
쿠버네티스는 다른 오케스트레이션 도구들에 비해서 비교적 늦게 등장했는데, 기존에 도커 진영에서 개발된 [도커 스웜](https://docs.docker.com/engine/swarm/) 이 쉽고 간단한 사용법으로 세력을 넓히고 있었고 
AWS 에서는 ECS, Hashicorp 에서는 Nomad, Mesos 에서는 Marathon 을 발표했었으나 쿠버네티스가 등장하고 나서는 쿠버네티스가 압도적인 점유율을 자랑하게 되었다.  



### 컨테이너 배포 

도커 컨테이너를 얼마나 많은 머신의 배포할 것인가?
배포된 도커 컨테이너 중 얼마나 많은 컨테이너에 실행을 유지할 것인가 ?
컨테이너의 일부가 실행 도중 실패하는 경우 어떻게 처리할까?
컨테이너와 컨테이너 사이의 연결 및 컨테이너와 스토리지의 연결 ?





## 쿠버네티스 용어
- 노드 : 컨테이너가 배치되는 서버
- 네임스페이스 : 쿠버네티스 클러스터 안의 가상 클러스터
- 파드 : 컨테이너의 집합 중 가장 작은 단위로 컨테이너의 실행 방법을 정의
- 레플리카 세트 : 같은 스펙을 갖는 파드의 복제품 세트
- 디플로이먼트 : 레플리카 세트의 리비전 관리
- 서비스 : 파드의 집합에 접근하기 위한 경로를 정의
- 인그레스 : 서비스를 쿠버네티스 클러스터 외부로 노출
- 컨피그맵 : 설정 정보를 정의하고 파드에 전달
- 퍼시스턴트 볼륨 : 파드가 사용할 스토리지의 크기 및 종류 정의
- 퍼시스턴트 볼륨 클레임 : 퍼시스턴트 봄률을 동적으로 확보
- 스토리지 클래스 : 퍼시스턴트 볼륨이 확보하는 스토리지의 종류를 정의 
- 스테이트 풀 세트 : 같은 스펙으로 모두 동일한 파드를 여러 개 생성하고 관리
- 잡 : 상주 실행을 목적으로 하지 않는 파드를 여러 개 생성하고 정상적인 종료를 보장
- 크론잡 : 크론 문법으로 스케줄링되는 잡
- 시크릿 : 인증 벙보 같은 기밀 데이터를 정의
- 롤 : 네임스페이스 안에서 조작 가능한 쿠버네티스 리소스의 규칙을 정의
- 롤 바인딩 : 쿠버네티스 리소스 사용자와 롤을 연결
- 클러스터 롤 : 클러스터 전체적으로 조작 가능한 쿠버네티스 리소스의 규칙을 정의
- 클러스터 롤 바인딩 : 쿠버네티스 리소스 사용자와 클러스터 롤을 연결
- 서비스 계정 : 파드가 쿠버네티스 리소스를 조작할 때 사용하는 계정 

## 노드 
- 마스터 노드는 클러스터 상에서 모든 작업을 제어 
작업의 스케줄링도 제어  
분산된 key-value 안정성 있는 제어/관리 (config map)
API 서버 제어 / 관리 

- 워커 노드에는 kubelet 이라는 쿠버네티스 에이전트가 설치되어 컨테이너가 배치됨 


## pod 
[쿠버네티스 도큐 pod](https://kubernetes.io/ko/docs/concepts/workloads/pods/pod-overview/)
파드는 쿠버네티스 애플리케이션의 기본 실행 단위  
ㄴ 하나의 가상머신이라고 볼 수 있음 
클러스터 에서 running 프로세스를 의미함  
일반적으로 파드는 단일 컨테이너 이거나 리소스를 공유하는 다중 컨테이너 이며, 쿠버네티스에서의 어플리케이션 단일 인스턴스를 의미  
pod 의 네트워크, 컨테이너, 스토리지는 노드와 분리되어 있음 

파드가 아닌 순수 노드 영역
- OS 계층
- 하드웨어 하드디스크
- 하드웨어 NIC (네트워크 인터페이스 카드)

![No Image](/assets/images/posts/20200704/node_and_pod.png)
node 에는 외부 네트워크와 교류하기 위한 IP 존재함 
pod 의 네트워크 인터페이스에는 node 의 IP 와 연결되기 위한 IP 가 존재함 
  
실질적으로 사용되는 컨테이너는 pod 내부에 구성되기 때문에 접근하기 위한 방법이 필요함 



### 쿠버네티스 오브젝트
[쿠버네티스 오브젝트 doc](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/kubernetes-objects/)

쿠버네티스 오브젝트는 시스템에서 영속성을 갖는 개체
쿠버네티스는 클러스터의 상태를 나타내기 위해서 이 개체를 이용
- 컨테이너 화 된 어플리케이션이 동작중인지
- 어플리케이션이 이용할 수 있는 리소스
- 어플리케이션이 어떻게 재구동하는 정책, 업그레이드

오프젝트 spec 과 status
- spec : 오브젝트를 생성할 때 리소스의 원하는 특징(의도한 상태)에 대한 설정
- status : 오브젝트의 현재 상태를 기술 
    * 쿠버네티스 컴포넌트에 의해서 제공되고 업데이트 됨 
    * 쿠버네티스 control plain 은 모든 오브젝트의 실제 상태를 사용자가 의도한 상태로 만들기 위해서 능동적으로 관리함 

오브젝트 기술 예제
```yaml
# 쿠버네티스 api 버전
apiVersion: apps/v1 # apps/v1beta2를 사용하는 1.9.0보다 더 이전의 버전용
# 오브젝트의 종류 
kind: Deployment
metadata:
  name: nginx-deployment
# 의도한 상태를 기술 
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # 템플릿에 매칭되는 파드 2개를 구동하는 디플로이먼트임 
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
해당 yaml 을 master 에 업로드하여 정의된 내용을 기반으로 pod 를 쿠버네티스가 생성해줌 

- 노드나 pod 에 장애 발생 
쿠버네티스는 N 개의 pod 가 클러스터내에 항상 실행되도록 구성가능 
특정 node, pod 가 장애가 발생하면 다른 node 에서 이를 대체하는 동일한 pod 를 구동하여 장애가 발생한 pod 를 대체함 

### Service 개념 


### 쿠버네티스 deployment
레플리카셋을 생성하는 디플로이먼트를 정의하여 배포 작업을 좀 더 세분화 하여 조작할 수 있음



### 쿠버네티스 rbac
RBAC(Role-Based Access Control) 은 role 기반으로 쿠버네티스 시스템의 권한 관리 
특정 사용자와 role 을 조합하여 사용자에게 권한을 부여할 수 있음 

- role 바인딩
role 은 특정 API, 리소스 (pod, deploy) , 사용권한 을 메니페스트 파일에 명시해둔 규칙의 집합으로 **특정 네임스페이스**에 대한 권한을 관리
롤 바인딩은 롤과 사용자를 묶어주는 역할을 수행 

### 쿠버네티스 볼륨
https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/

컨테이너 특성상 문제가 발생해서 컨테이너가 삭제되면 마운트되지 않은 데이터들은 삭제됨 
중요한 데이터는 볼륨을 사용하여 데이터를 보관함

- emptyDir
Pod 가 사라지면 볼륨도 같이 삭제되는 임시 볼륨 
ㄴ pod 생성시 pod 의 로컬 볼륨 할당 


- hostPath
노드의 디스크에 볼륨을 생성하여 Pod 가 삭제 되더라도 볼륨에 있던 데이터는 유지됨 




### 쿠버네티스 영속성 볼륨 
- PV(Persistent Volume)
PV 는 관리자에 의해 생성된 볼륨을 의미함 

- PVC(Persistent Volume Cliam)
PVC 는 사용자가 볼륨을 사용하기 위해 PV 에 요청을 함 

#### Life cycle 
1. 프로비저닝
static, dynamic 의 pv 를 생성하는 단계 
static 프로비저닝은 메니페스트 파일 등을 통해 PV 미리 생성해두고, 요청시 마다 PV 를 할당하여 사용
dynamic 프로비저닝은 사용자가 요청할때 PV 를 생성하여 할당하고, 사용자는 원하는 만큼의 용량을 생성해서 자유롭게 사용 

2. 바인딩
PV 를 PVC 에 연결시키는 단계 
PVC 는 사용자가 요청하는 볼륨을 PV 에 요청하고 PV 는 그에 맞는 볼륨을 할당함

3. 사용
Pod 는 PVC 를 볼륨으로 사용
클러스터는 PVC 를 확인하여 바인딩된 PV 를 찾아 해당 볼륨을 Pod 가 사용할 수 있도록 함 

4. 회수
PV 는 기존에 사용했던 PVC 가 아니라도 다른 PVC 로 재활용이 가능함 
사용이 종료된 PVC 를 삭제할 때, 사용했던 PV 의 데이터를 어떻게 처리할지에 대한 설정을 하게 됨 
- retain: pv 데이터를 그대로 보존
- recycle: 재사용하게 될 경우 기존의 PV 데이터들을 모두 삭제 후 재사용함 
- delete: 사용이 종료되면 해당 볼륨을 삭제 

### 라벨과 어노테이션 
쿠버네티스 에서 자원들의 메타 데이터를 관리하는데 라벨과 어노테이션을 사용함  
라벨은 주로 특정 자원을 선택할 때 사용  
어노테이션은 주석 성격의 메타 데이터를 기록해 두기위해서 사용  

### 라벨 
key/value 쌍으로 구성되며 라벨은 사용자가 클러스터내에 객체를 만들때 메타 데이터로 붙일 수 있다
생성된 후 언제든지 수정이 가능  
쿠버네티스 내에 컨트롤러들이 pod 를 관리할 때 자신이 관리해야 할 pod 를 구분할 수 있는 key 역할을 한다  
라벨은 중간에 변경하게 되면 컨트롤러를 인식할 수 없는 문제가 발생함  
이런 유연성 때문에 특정 pod 만 라벨을 수정하여 디버깅에 사용가능하다  
노드에도 라벨을 붙일 수 있으며 클러스터내의 노드들의 라벨을 통해서 구분된 노드에만 pod 를 띄우는 것도 가능함  

### 어노테이션 
key/value 쌍으로 구성되며 라벨과의 차이점은 라벨은 특정 자원을 선택하는 데 사용하는 것이라고 하면,  
어노테이션은 쿠버네티스 시스템이 필요한 정보들을 담고 있어서 쿠버네티스 클라이언트나 라이브러리가 활용하는데 사용됨  



[아리수 블로그](https://arisu1000.tistory.com/27841)

## configmap
어플리케이션 배포를 하다 보면, 환경에 따라서 다른 설정 값을 사용할 경우가 있음  
ㄴ 실제 어플리케이션 같지만 접속 정보가 다른 ... 
환경 변수들을 configmap 과 secret 으로 관리하면 Pod 가 생성될 때 해당 값을 넣어줄 수 있음  
  
configmap 과 secret 을 pod 로 넘기는 방법 
- 정의해 놓은 값을 Pod 의 환경변수로 넘기는 방법
- 정의해 놓은 값을 Pod 의 디스크 볼륨을 마운트 하는 방법 
  
configmap 은 key/value 형식으로 저장되며, literal 이나 파일로 생성이 가능함  

### literal(문자) 로 생성하는 방법
key = language, value = java 를 설정해본다고 하자 
- 명령어로 생성 
```shell script
# kubectl create configmap [이름] --from-literal=[키]=[값]
$ kubectl create configmap my-config --from-literal=language=java
```

- yaml 작성 
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data: # 여기에 작성 
  language: java
```

### pod 에서 configmap 참조하기 
```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: cm-deployment
spec:
  replicas: 3
  minReadySeconds: 5
  selector:
    matchLabels:
      app: cm-literal
  template:
    metadata:
      name: cm-literal-pod
      labels:
        app: cm-literal
    spec:
      containers:
      - name: cm
        image: gcr.io/terrycho-sandbox/cm:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        env: # 어플리케이션에서 사용할 환경변수를 지정 
        - name: LANGUAGE
          valueFrom:
            configMapKeyRef:
               name: hello-cm
               key: language
```
### 디스크 볼륨으로 생성하는 방법
configmap 의 정보를 pod 의 디스크 볼룸으로 마운트 시키는 방법
```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: cm-file-deployment-vol
spec:
  replicas: 3
  minReadySeconds: 5
  selector:
    matchLabels:
      app: cm-file-vol
  template:
    metadata:
      name: cm-file-vol-pod
      labels:
        app: cm-file-vol
    spec:
      containers:
      - name: cm-file-vol
        image: gcr.io/terrycho-sandbox/cm-file-volume:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        volumeMounts: # 마운트 
          - name: config-profile
            mountPath: /tmp/config
      volumes: # 볼륨 
        - name: config-profile
          configMap:
            name: cm-file
```

## helm
https://github.com/helm/helm

차트: 사전에 구성된 쿠버네티스 리소스 패키지 
helm 은 차트 관리 도구 

[k2hub 헬름 젠킨스](http://k2hub.9rum.cc/charts/kakao-community/jenkins)

helm repo add kakao-community http://charts-kakao-community.k8s.9rum.cc
helm repo update


helm install kakao-community/jenkins --version 0.18.5



$ helm install --name sangwookjenkins --set Master.HostName=sangwookjenkins.dev.daumkakao.io kakao-community/jenkins

$ printf $(kubectl get secret --namespace default sangwookjenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
2mLGYlISeA


### 차트 생성하기 
$ helm create echo
- 기본 디렉토리 구조
ㄴ echo
ㄴ Chart.yaml           차트의 정보가 정의된 파일
ㄴ charts               차트가 의존하는 차트의 디렉토리 (의존차트)
ㄴ templates            메니페스트 파일 템플릿 디렉토리 
    ㄴ NOTE.txt         차트 사용법에 대한 참조 문서 
    ㄴ _helpers.tpl     메니페스트 랜더링에 사용되는 템플릿 헬퍼 
    ㄴ deployment.yaml  
    ㄴ ingress.yaml     
    ㄴ service.yaml     
ㄴ values.yaml          이 차트에서 사용하는 기본 설정 값 
ㄴ requirements.yaml    차트의 의존성을 명시한 파일 

[](https://arisu1000.tistory.com/27861)



## pod 의 라이프 사이클 
https://kubernetes.io/ko/docs/concepts/workloads/pods/pod-lifecycle/#%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%ED%94%84%EB%A1%9C%EB%B8%8Cprobe

파드는 pending 단계에서 시작해서 기본 컨테이너 중 1개 이상이 OK 로 시작하면 Running 단계로 통과하고, 
파드의 컨테이너가 실패로 종료되었는지 여부에 따라 Succeeded 또는 Failed 단계로 이동  

파드가 실행되는 동안 kubectl 은 일종의 오류를 처리하기 위해서 컨테이너를 다시 시작할 수 도 있음  
파드 내에서 쿠버네티스는 다양한 컨테이너 상태와 핸들을 추적함  

### 파드의 수명 
파드가 생성되면 UID 가 할당되고, 종료 또는 삭제 될때까지 노드에 스케줄된다  
노드가 종료되면 파드는 타임아웃 이후에 삭제되도록 스케줄 된다  

파드는 자체적으로 치유되지 않기 때문에 파드가 노드에 스케줄된 후에 실패하거나, 스케줄 작업 자체가 실패하면, 파드는 삭제된다  
파드는 리소스 부족 또는 노드 유지 관리 작업으로 인해 제거 되지는 않는다  
쿠버네티스는 컨트롤러라고 불리는 하이-레벨 추상화를 사용하여 파드 인스턴스를 관리함  

UID 로 정의된 특정 파드는 다른 노드로 다시 스케줄 되지 않음  
대신 해당 파드는 UID 가 다른 거의 동일한 파드로 대체될 수 있음  

### 파드의 단계
파드의 status 필드는 phase 필드를 포함하는 PodStatus 오브젝트로 정의됨 
파드의 phase 는 파드의 라이프 사이클 중 어느 단계에 해당하는지를 표현하는 요약정보  

phase 의 종류
- Pending : 파드가 쿠버네티스 클러스터에서 승인되었지만, 하나 이사으이 컨테이너가 설정되지 않았고 실행할 준비가 되지 않았음
여기에는 파드가 스케줄 되기 이전까지의 시간 뿐만아니라 네트워크를 통한 컨테이너 이미지 다운로드 시간도 포함됨 
- Running : 파드가 노드에 바인딩되어 모든 컨테이너가 생성되었음 
적어도 1개 이상의 컨테이너가 아직 실행중이거나, 시작 or 재시작 중에 있음 
- Succeeded : 파드에 있는 모든 컨테이너들이 성공적으로 종료되었고 재시작 되지 않음을 의미
- Failed : 파드에 있는 모든 컨테이너가 종료되었고, 적어도 1개 이상의 컨테이너가 실패로 종료됨을 의미
실패한 컨테이너는 non-zero 상태로 exited 되었거나 시스템에 의해서 종료됨 
- Unknown : 어떤 이유에 의해서 파드의 상태를 얻을 수 없음
일반적으로 파드가 실행되어야 하는 노드와의 통신 오류로 볼 수 있음 

### 컨테니어의 상태
쿠버네티스는 컨테이너의 상태를 추적함  
컨테이너 라이프 사이클 훅을 사용하여 컨테이너 라이프사이클의 특정 지점에 실행할 이벤트를 트리거 할 수 있음  

스케줄러가 노드에 파드를 할당하면, kubelet 은 컨테어너 런타임을 사용하여 해당 파드에 대한 컨테이너 생성을 시작함  
파드의 컨테이너 상태 확인 명령어
```
$ kubectl describe pod <name-of-pod>
```

표시되는 컨테이너의 상태 
- waiting
컨테이너가 running, terminated 상태가 아닐때 waiting 상태로 노출된다  
wating 상태의 컨테이너는 시작을 완료하는 데 필요한 작업을 실행중에 있음 
kubectl 을 사용하여 컨테이너가 waiting 인 파드를 쿼리하면, 컨테이너가 해당 상태에 있는 이유(reason) 를 노출해준다 
- running
컨테이너가 문제없이 실행되고 있음을 나타냄 
- terminated
컨테이너는 실행을 시작한 다음 완료될 때까지 실행되었거나 어떤 이유로 실해함  

### 컨테이너 재시작
파드의 spec 에는 restartPolicy 필드가 있음 
- Always(기본), OnFailure, Never


### 컨테이너 probe
프로브는 컨테이너에서 kubelet 에 의해 주기적으로 수행되는 진단  






## 래퍼런스 
- [공식 도큐 - 쿠버네티스란 무엇인가?](https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/)
- [위키북스 도커/쿠버네티스를 활용한 컨테이너 개발입문](https://wikibook.co.kr/docker-kubernetes/)
- [subicura - 쿠버네티스 기본](https://subicura.com/2019/05/19/kubernetes-basic-1.html)

