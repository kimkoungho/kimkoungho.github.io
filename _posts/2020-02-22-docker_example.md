---
title: "dockerfile 명령어"
excerpt: ""
categories: devops
tags:
    - dockerfile 명령어 
last_modified_at: 2020-02-22
---

도커 설치는 도커 사이트에서 설치 파일을 받아서 진행하면된다
- [맥 환경에서 도커 설치](https://docs.docker.com/docker-for-mac/install/)

# Dockerfile  
Dockerfile 은 도커 이미지를 만들때 사용하는 코드로 [도커 허브](https://hub.docker.com/) 에 있는 도커 이미지들은 모두 Dockerfile 을 빌드한 결과 이미지이다.  

Dockerfile 에 사용되는 명령어들 
- FROM: 현재 만들 이미지의 베이스 이미지를 지정하는 것으로 기본적으로 도커 허브에 지정된 이미지를 받아올 수 있다

- RUN: Dockerfile 을 이용해서 이미지를 빌드할 때, 도커 컨테이너 내부에 실행할 명령어를 지정할 수 있다

- COPY: 파일을 도커 컨테이너 내부로 복사하는 명령이다 

- ADD: COPY 와 같이 파일을 복사하는데 사용하는 명령으로 URL 및 압축파일도 지정이 가능하다

- LABEL: 
- ENV: 
- EXPOSE: 
- STOPSIGNAL
- USER: 

- VOLUME:
- WORKDIR:


- CMD: 이미지를 도커 컨테이너로 실행하기 전에 먼저 실행할 명령어를 지정할 수 있다
RUN 명령어는 이미지를 빌드하는 시점에 실행되는 반면에 CMD 는 컨테이너를 실행할 때 명령어를 수행한다  
주의 해야할 점은 Dockerfile 에서 CMD 는 1개만 적용되며 여러 개를 지정한 경우 마지막 CMD 만이 적용된다  

```Dockerfile
# 가장 기본 형식 
CMD ["java ", "param1", "param2"]
# 쉘 명령어 실행 
CMD command param1 param2

# ENTRYPOINT 의 매개변수 전달 
CMD ["", ""]
```


- ENTRYPOINT: CMD 와 같이 도커 컨테이너를 실행하기 전에 실행할 명령어 이지만 
책에서는 ENTRYPOINT 를 go 로 지정해서 기본 명령어 .. ??


실제로 이미지를 빌드 
```shell script
# 형식: docker image build -t 이미지명[:태그명] Dockerfile 경로
$ docker image build -t test .
# 태그명을 지정하지 않으면 기본적으로 latest 가 됨 
# . 은 현재 디렉토리를 의미 
```


* 이미지와 컨테이너는 각각 고유의 해시 값을 갖는다
이미지에는 이름과 버전 태그를 지정 가능함 






- [위키북스 도커/쿠버네티스를 활용한 컨테이너 개발입문](https://wikibook.co.kr/docker-kubernetes/) 2장 
- [도커 dockerfile 래퍼런스](https://docs.docker.com/engine/reference/builder/)