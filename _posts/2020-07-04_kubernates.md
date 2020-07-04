# kubernates

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

### 쿠버네티스 deployment
레플리카셋을 생성하는 디플로이먼트를 정의하여 배포 작업을 좀 더 세분화 하여 조작할 수 있음



### 쿠버네티스 rbac
RBAC(Role-Based Access Control) 은 role 기반으로 쿠버네티스 시스템의 권한 관리 
특정 사용자와 role 을 조합하여 사용자에게 권한을 부여할 수 있음 

- role 바인딩
role 은 특정 API, 리소스 (pod, deploy) , 사용권한 을 메니페스트 파일에 명시해둔 규칙의 집합으로 **특정 네임스페이스**에 대한 권한을 관리
롤 바인딩은 롤과 사용자를 묶어주는 역할을 수행 

### 쿠버네티스 볼륨
컨테이너 특성상 문제가 발생해서 컨테이너가 삭제되면 마운트되지 않은 데이터들은 삭제됨 
중요한 데이터는 볼륨을 사용하여 데이터를 보관함

- emptyDir
Pod 가 사라지면 볼륨도 같이 삭제되는 임시 볼륨 

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