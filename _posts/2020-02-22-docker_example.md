---
title: "도커 실습 해보기"
excerpt: ""
categories: devops
tags:
    - docker 설치 
    - dockerfile 명령어 
    - dockerfile 작성법
    - docker container ssh 
last_modified_at: 2020-05-31
---

# 도커 설치 
도커 설치는 리눅스 기반에서 설치하는 것과 mac 이나 window 에서 설치하는 것에는 차이가 있다.    
- 실제로 리눅스 환경과 mac/window 도커 환경은 차이가 있다고 함 
도커는 기본적으로 리눅스 컨테이너 기술이기 때문에 리눅스에서 설치 하는것을 설명하려고 함   
  
## 도커 설치
centos 7 에서 설치를 진행함  
- [centos 도커 설치](https://docs.docker.com/engine/install/centos/)   
  
```shell
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
 
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install -y docker-ce docker-ce-cli containerd.io
 
$ sudo systemctl start docker
```

- 이렇게 도커를 설치하고 나면 도커 데몬이 뜨게 된다 
  
리눅스 환경에서는 도커 데몬을 위한 유저를 추가해주어야 함 
- 도커 데몬은 root 권한이 필요한데 도커 유저인 docker 는 root 권한이 없기 때문에 추가가 필요
 
```shell
# 확인
$ ls -lah /var/run/docker.sock
srw-rw---- 1 root root 0  8월  3 14:49 /var/run/docker.sock
   
# 도커 유저 추가
$ sudo groupadd docker
$ sudo systemctl restart docker
   
# docker 로 변경됨
$ ls -lah /var/run/docker.sock
srw-rw---- 1 root docker 0  8월  3 14:50 /var/run/docker.sock
 
# 도커 group 에 유저 추가
$ sudo usermod -aG docker me 
# - me 는 현재 접속중인 사용자라고 생각하면 됨 
 
# 그룹 확인
$ cat /etc/group | grep "docker:"
``` 
  
## 맥 환경 설치
필자는 mac 환경이기 때문에 mac 환경 설치를 추가적으로 작성했다  
로컬에서 도커를 테스트 해보기 위해서는 설치가 필요하기 때문에 ...   
mac 환경 설치는 아래 가이드에 따라서 설치 파일을 다운로드 하면된다   
- [맥 환경에서 도커 설치](https://docs.docker.com/docker-for-mac/install/)
  
설치를 진행하고 나면 도커에 대한 환경 설정을 볼수 있음   
- 우측 상단에 도커 toolbar 가 표시됨 
![No Image](/assets/images/posts/20200222/docker_toolbar.png)

- preferences 메뉴에서 도커 관련 설정을 할 수 있음 
![No Image](/assets/images/posts/20200222/dockerfile_image_container.png)

## 도커 설치 확인하기  
```shell
$ docker version
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.12.12
 Git commit:        633a0ea
 Built:             Wed Nov 13 07:22:34 2019
 OS/Arch:           darwin/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.5
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.12
  Git commit:       633a0ea
  Built:            Wed Nov 13 07:29:19 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
```
  
# Dockerfile 
Dockerfile 은 도커 이미지를 만들때 사용하는 코드로 [도커 허브](https://hub.docker.com/) 에 있는 도커 이미지들은 모두 Dockerfile 을 빌드한 결과 이미지이다  
  
![No Image](/assets/images/posts/20200222/dockerfile_image_container.png)  
- 출처: [geekflare의 docker_tutorial](https://geekflare.com/dockerfile-tutorial/https://geekflare.com/dockerfile-tutorial/)
위 이미지에서 알 수 있듯이 **Dockerfile 은 도커 이미지 생성에 사용하는 템플릿 파일**    
- [이전 포스팅](https://kimkoungho.github.io/devops/docker_basic/)에도 설명했지만 도커는 Dockerfile 을 작성하여 코드로 관리하는 인프라를 구현하고 있음  
    
## Dockerfile 작성하기 
Dockerfile 에 사용되는 명령어들을 살펴보자  
### FROM 
현재 만들 이미지의 베이스 이미지를 지정하는 것으로 기본적으로 도커 허브에 지정된 이미지를 받아올 수 있다
```dockerfile
FROM centos:7
```
- centos:7 만 설치된 도커 이미지를 base image 로 해서 이미지를 생성한다 
- :7 는 해당 이미지의 태그이며 일반적으로 버전정보를 나타낼때 사용된다 

### RUN
Dockerfile 을 이용해서 이미지를 빌드할 때, 도커 컨테이너 내부에 실행할 명령어를 지정할 수 있다  
RUN 에는 2가지 형태가 존재하는데 
```dockerfile
# format
RUN "command"

# example
RUN yum install -y python
``` 
- /bin/bash -c command 형식으로 shell 명령을 실행하는 것 

```dockerfile
RUN ["executable", "param1", "param2"]
# example
RUN ["/bin/bash", "-c", "echo hello"]
```
- executable 은 명령어를 실행할 양식 (bash, zsh, etc)
- executable 을 이용해서 별도의 python, ruby 스크립트를 실행 가능하다 
- param1, param2 는 해당 명령어를 실행하면서 전달할 파라미터
  
RUN 명령어의 결과물은 이미지 내부에 저장된다 
- 도커에서는 이미지를 빌드 하는 시점과 컨테이너를 구동 하는 시점에 따라서 행위를 구분할 수 있으며 CMD, RUN 명령의 가장 큰 차이라고 보면 된다 

### CMD
이미지를 도커 컨테이너로 실행하기 전에 먼저 실행할 명령어를 지정할 수 있다  
RUN 명령어는 이미지를 빌드하는 시점에 실행되는 반면에 CMD 는 컨테이너를 실행할 때 명령어를 수행한다  
주의 해야할 점은 Dockerfile 에서 CMD 는 1개만 적용되며 여러 개를 지정한 경우 마지막 CMD 만이 적용된다  
  
CMD 에는 3가지 명령어 형식이 있음  
```dockerfile
# 가장 기본 형식 
CMD ["java ", "param1", "param2"]
# 쉘 명령어 실행 
CMD command param1 param2

# ENTRYPOINT 의 매개변수 전달 
CMD ["", ""]
```
- 컨테이너에 기본 값을 지정하기 위해서 일반적으로 사용되며, 컨테이너 구동시 실행할 어플리케이션 시작 명령을 포함할 수 있다 

### ENTRYPOINT 
CMD 와 같이 도커 컨테이너를 실행하기 전에 실행할 명령어     
CMD 와 마찬가지로 1번만 수행할 수 있음  
```dockerfile
# 바람직한 형태 
ENTRYPOINT ["executable", "param1", "param2"]
# 쉘 명령어 형태 
ENTRYPOINT command param1 param2
```
CMD 와의 차이점은 ENTRYPOINT 는 CMD 를 인자로 받아 사용할 수 있는 스크립트  


### LABEL
Dockerfile 로 생성하려는 이미지에 메타 정보를 추가할 때 사용  
key 와 value 의 쌍으로 표현함  
```dockerfile
LABEL <key>=<value> <key>=<value>
LABEL "com.example.vendor"="ACME Incorporated"
``` 
- 하나의 이미지에 여러개의 라벨을 지정 가능함 
- 부모 이미지의 라벨은 자식 이미지가 상속 
  
라벨 출력방법
```shell
$ docker image inspect --format='' myimage
```

### ENV
Dockerfile 에서 사용할 환경 변수를 지정  
2 가지 형식으로 모두 지정가능   
```dockerfile
ENV <key> <value>
ENV <key>=<value>

ENV JAVA_HOME /usr/lib/java
``` 
- 해당 환경 변수는 자동으로 등록되는 특징이 있음
- 도커 컨테이너 구동후 echo $JAVA_HOME 을 입력하면 /usr/lib/java 가 출력됨  

### COPY 
파일을 도커 컨테이너 내부로 복사하는 명령이다  
```dockerfile
COPY [--chown=<user>:<group>] <src>... <dest>
# 공백이 포함된 경우 
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```
- 파일, 디렉토리를 복사하는 명령어 
  
### ADD 
COPY 와 같이 파일을 복사하는데 사용하는 명령으로 URL 및 압축파일도 지정이 가능하다
```dockerfile
ADD [--chown=<user>:<group>] <src>.. <dest>
# 공백이 필요한 경우 
ADD [--chown=<user>:<group>] ["<src>".. "<dest>"]

ADD test.txt /absoluteDir/
```
- 이미지를 빌드 환경에 있는 파일을 이미지로 복사하는 것 
 
### EXPOSE
도커가 컨테이너의 런타임에 지정된 네트워크 포트에서 수신 대기를 한다는 것을 나타냄 
```dockerfile
EXPOSE <port> [<port>/<protocol>...]

EXPOSE 80/tcp
``` 
- 프로토콜의 기본 값은 tcp 
- EXPOSE 명령은 실제로 해당 포트를 띄우는 것이 아니라 이미지를 만드는 사람과 컨테이너를 띄우는 사람간의 문서로써의 역할 

실제로 포트를 바인딩하는 예
```shell
$ docker run -p 80:80/tcp
```
- 호스트 os 의 80 포트를 컨테이너 80 포트로 바인딩

### VOLUME
지정된 이름으로 마운트 포인트를 생성하고 호스트 OS 에서 마운트된 볼륨을 보유하는 것으로 표시  
```dockerfile
VOLUME ["/data"]
VOLUME ["/var/log/"]
```

### WORKDIR
작업 디렉토리를 설정하는 것으로 WORKDIR 을 지정하면 RUN, CMD, ENTRYPOINT, COPY, ADD 등의 명령이 영향을 받는다 
```dockerfile
WORKDIR /path/to/workdir

WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
# /a/b/c 가 출려됨 
``` 


### STOPSIGNAL
컨테이너로 보내질 시스템 호출 신호를 설정  
해당 신호는 커널의 syscall 테이블에 위치   
```dockerfile
STOPSIGNAL signal
```

### USER 
도커를 실행하는 사용자를 지정하는 명령어 
```dockerfile
USER <user>[:<group>]
USER <UID>[:<GID>]
```
- 사용자 지정후 RUN, CMD, ENTRYPOINT 등의 명령은 해당 사용자의 권한으로 실행됨 
- 컨테이너 구동시에도 해당 사용자가 기본 사용자가 된다 

### ONBUILD
빌드된 이미지를 기반으로 하는 다른 이미지가 Dockerfile 로 생성될 때 실행할 명령어를 추가 
```dockerfile
ONBUILD <INSTRUCTION>
```
- 상속 구조를 갖는 이미지들이 있을 경우 base 가 되는 이미지에서 미리 수행할 명령을 등록하는 것 
- 하나의 트리거로써 동작함 
- ONBUILD 명령은 해당 이미지가 FROM 절에 있을때 실행되는 명령 
    * 이미지 자체를 빌드할 때는 수행되지 않음 

## Dockerfile 작성 예제 
centos 기반으로 ssh 접속이 가능한 도커 이미지를 생성한다 
```dockerfile
FROM centos:7

### user
ARG user=deploy
ARG group=deploy
ARG uid=1000
ARG gid=1000
ARG USER_HOME=/home/deploy

# user home
RUN mkdir -p ${USER_HOME} \
    && chown ${uid}:${gid} ${USER_HOME} \
    && groupadd -g ${gid} ${group} \
    && useradd -d "${USER_HOME}" -u ${uid} -g ${gid} -m -s /bin/bash ${user}
# sudo user
RUN yum install -y sudo \
    && echo "${user} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

### ssh
RUN yum install -y epel-release \
    && yum install -y openssh-server openssh-clients net-tools hostname \
    && yum clean all

# 비밀번호 인증을 비활성화
RUN sed -i -e 's~^PasswordAuthentication yes~PasswordAuthentication no~g' /etc/ssh/sshd_config

## create ssh key 
RUN ssh-keygen -q -b 1024 -N '' -t rsa -f /etc/ssh/ssh_host_rsa_key \
    && ssh-keygen -q -b 1024 -N '' -t dsa -f /etc/ssh/ssh_host_dsa_key \
    && ssh-keygen -q -b 521 -N '' -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key \
    && ssh-keygen -q -b 521 -N '' -t ed25519 -f /etc/ssh/ssh_host_ed25519_key

# sudo 명령어 경로 추가 - secure_path 주석처리 
RUN echo 'Defaults  env_keep += "PATH PROMPT_COMMAND"' >> /etc/sudoers

# ssh user
RUN mkdir -p ${USER_HOME}/.ssh \
    ## 생성된 키를 인증할 수 있는 키 목록에 추가
    && cat /etc/ssh/ssh_host_rsa_key >> ${USER_HOME}/.ssh/id_rsa \
    && cat /etc/ssh/ssh_host_rsa_key.pub >> ${USER_HOME}/.ssh/authorized_keys \
    && chown -R ${uid}:${gid} ${USER_HOME} \
    && chmod 600 ${USER_HOME}/.ssh/id_rsa \
    && chmod 644 ${USER_HOME}/.ssh/authorized_keys

### utility lib
RUN yum -y install bash-completion curl hostname vim-enhanced wget

EXPOSE 22
USER ${user}

CMD ["sudo", "/usr/sbin/sshd", "-D"]
```
- [github](https://github.com/kimkoungho/dockerfile_project/blob/master/centos7_ssh/Dockerfile)

## Dockerfile 실행 
- 이미지 빌드하기 

```shell
# 형식: docker image build -t 이미지명[:태그명] Dockerfile 경로
$ docker build -t centos7_ssh:latest . 
# -t 이미지_이름:버전 
#   태그명을 지정하지 않으면 기본적으로 latest 가 됨 
# . 은 현재 디렉토리를 의미

$ docker images
REPOSITORY                                           TAG                 IMAGE ID            CREATED             SIZE
centos7_ssh                                          latest              47e9c5c43729        2 months ago        553MB
```

- 컨테이너 뛰워보기  

```shell
$ docker run --name node1 \
      --rm \
      -d centos7_ssh 
$ docker run --name node2 \
      --rm \
      -d centos7_ssh
# --rm 은 컨테이너 종료시 자동으로 삭제 하는 옵션
# -d 는 백그라운드로 컨테이너를 구동 

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
f73276ed5a2d        centos7_ssh         "sudo /usr/sbin/sshd…"   2 seconds ago       Up 1 second         22/tcp              node2
d41861cf9683        centos7_ssh         "sudo /usr/sbin/sshd…"   6 seconds ago       Up 6 seconds        22/tcp              node1
```

- 실제 접속을 해보자 

```shell
# node1 에 bash 로 접속 
$ docker exec -it node1 /bin/bash
bash-4.2$

# 배시 를 새창으로 띄워서 
# node2 에 bash 로 접속 
$ docker exec -it node2 /bin/bash
bash-4.2$
```

- ssh 로 접속해보기 
node1 에서 node2 로 ssh 접속   
먼저 node2 의 IP 를 확인  

```shell
bash-4.2$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      inet 172.17.0.3  netmask 255.255.0.0  broadcast 172.17.255.255
...
```
node1 에서 node2 로 접속

```shell
bash-4.2$ ssh 172.17.0.3
The authenticity of host '172.17.0.3 (172.17.0.3)' can't be established.
ECDSA key fingerprint is SHA256:CEoPcuVROOzbABUO5CCsEf0KI+58qxIagaizYTblLHU.
ECDSA key fingerprint is MD5:12:0b:86:9a:fb:db:de:6f:81:46:1b:47:1b:bc:79:b6.
Are you sure you want to continue connecting (yes/no)? 
```
- known_hosts 에 추가할 것인지 묻는데 yes 때리면 됨 

```shell
-bash-4.2$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      inet 172.17.0.3  netmask 255.255.0.0  broadcast 172.17.255.255

-bash-4.2$ exit
bash-4.2$
```

## TODO:
- docker 관련 명령어 정리
* 이미지 search, pull, ls, rm
* 컨테이너 ls, stop, start, rm, stats
- 이미지에 태그 부여하기 
- prune : 일괄삭제 명령어
- 도커 컴포즈 


## Reference
- [위키북스 도커/쿠버네티스를 활용한 컨테이너 개발입문](https://wikibook.co.kr/docker-kubernetes/) 2장 
- [도커 dockerfile 래퍼런스](https://docs.docker.com/engine/reference/builder/)