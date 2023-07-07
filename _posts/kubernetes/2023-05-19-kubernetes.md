---
title: 쿠버네티스(Kubernetes)
categories: [kubernetes,docker]
tags: [kubernetes,docker]     # TAG names should always be lowercase
---

도커, 쿠버네티스, 서비스메시에 대한 개념/설치과정 정리


# 도커

## 도커의 역사 

서버와 여러 실행환경 관련된 프로그램을 설치할 때 마다 각종 설정/충돌/오류가 발생하기 마련임.

그리고 이러한 문제는 마이크로서비스 아키텍쳐가 유행하게되며 프로그램 단위는 더 작게 쪼개지고, 관리가 더 복잡해게 되며 부각됨.

이때 도커가 출시됨.

2013년 3월 산타클라라에서 열린 Pycon Conference에서 dotCloud의 창업자인 Solomon Hykes가 The future of Linux Containers 라는 세션을 발표하면서 처음 공개함.

이 발표 이후 도커가 인기를 얻으면서 2013년 10월 아예 회사이름을 도커(Docker Inc.)로 바꾸고 2014년 6월 도커 1.0을 발표함. 

2014년 8월 도커에 집중하기 위해 dotCloud 플랫폼을 매각하고 2015년 4월 $95M(약 1,100억원) 투자를 유치하였음.

## 컨테이너

컨테이너 개념은 도커에서 처음 나온게 아니다.

원래 독립적인 프로그램의 실행환경을 제공하기 위해, 예전에는 OS를 추가적으로 설치하여 격리하는, 우리가 사용하는 VMware, Virtual Box와 같은 방법들이 사용되었다.

하지만 이 방식은 구조적으로 성능의 손실이 있을 수 밖에 없고, 이를 개선하기 위해 프로세스를 격리하는 방식이 등장했다. 이 방식에서, `격리된 프로세스`를 컨테이너 라 부른다.

따라서 도커 역시 당시 완전히 새로운 기술은 아니었고, 다만 `사용하기 쉽게` 컨테이너, 오버레이 네트워크, 유니온 파일 시스템 등의 기능을 묶은 것이다.

TODO 오버레이 네트워크, 유니온 파일 시스템

## 유니온 파일 시스템

각 컨테이너는 완전한 독립된 프로세스를 제공해야 하기 때문에, 용량이 어느정도 클 수 밖에 없다.

비슷한 예시를 들자면, 가장 간단하게 예전 방식인 OS 가상화를 예시로 들 수 있다.

프로그램을 독립된 환경에서 실행시키기 위해 온전한 OS 하나를 추가로 설치해야 했으니, 용량의 낭비가 심할 수 밖에 없다.

컨테이너는 프로세스만을 독립시켜 OS보다는 가벼우나 그래도 용량이 수백MB정도는 된다.

따라서 도커에서는 `레이어 저장방식`을 통해 각 프로그램을 레이어별로 합쳐 사용하게 해 준다.

(이미지)

이미지는 여러개의 읽기 전용read only 레이어로 구성되고 파일이 추가되거나 수정되면 새로운 레이어가 생성됩니다. ubuntu 이미지가 A + B + C의 집합이라면, ubuntu 이미지를 베이스로 만든 nginx 이미지는 A + B + C + nginx가 됩니다. webapp 이미지를 nginx 이미지 기반으로 만들었다면 예상대로 A + B + C + nginx + source 레이어로 구성됩니다. webapp 소스를 수정하면 A, B, C, nginx 레이어를 제외한 새로운 source(v2) 레이어만 다운받으면 되기 때문에 굉장히 효율적으로 이미지를 관리할 수 있습니다.

## 이미지 경로

TODO

## Dockerfile

예제 : 

```
FROM node:12-alpine
RUN apk add --no-cache python3 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

* FROM: 베이스 이미지이다. 간단하게 말해 OS라고 생각하면 되며, 참고로 Alpine은 매우 작은 Linux 배포판인 Alpine Linux를 기반으로 하는 기본 이미지이다. node:12-alpine의 의미는 노드 12에 설치된 알파인 기반 이미지를 의미하며, node:alpine 사용시 최신 버전의 알파인을 사용할 수 있다. 지금은 테스트이기 때문에 무거운 베이스 이미지 대신 알파인을 사용한 것이다.)
* RUN: 도커 이미지가 생성되기 전에 수행할 쉘 명령어이다.
* WORKDIR: WORKDIR 지시자는 도커 파일에서 리눅스 명령어의 cd와 유사하게 뒤에 오는 모든 지시자(RUN, CMD, COPY, ADD 등)에 대한 작업 디렉토리를 설정한다. 특히 원래 최상위 파일이나 폴더에 있던 이름과 COPY해오는 파일이나 폴더의 이름이 같은게 있다면, 기존에 파일을 덮어쓰게되어 문제가 생길 수 있으며, 최상위 폴더에 모두 있을 경우 가독성 또한 좋지 않다.
* COPY: Docker클라이언트의 현재 디렉토리에서 파일을 추가한다. (COPY (source) (dist)). WORKDIR를 별도로 지정했다면 로컬에 있는 파일들이 도커 컨테이너로 복사될때 WORKDIR에 정의한 디렉토리로 복사된다.
* CMD: 컨테이너 내에서 실행할 명령을 지정하며, 도커 파일 내에서 한 번만 사용할 수 있다.

```
$ docker build -t getteing-started .
```



## Docker Hub

TODO


## 컨테이너 다루기

실행 명령어 : 

```
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

TODO 관련 명령어 / 설정들

## 컨테이너 만들기

TODO

## Docker Compose

TODO


# 쿠버네티스(Kubernetes)

클라우드 계의 리눅스라 불림.

어떤 OS를 쓰느냐와 상관없이 Kubernetes로 다 굴러감.

## 쿠버네티스의 아키텍처

Master - Node 구조

### Kubectl

### Master


#### Kube Controller

#### API Server

#### Scheduler


### Node

#### Kublet

#### Docker



## Object

## State

## YAML

