---
title: dockerfile 작성법
date: 2023-07-07 14:25:00 +/-TTTT
categories: [Docker, Dockerfile]
tags: [docker, dockerfile]     # TAG names should always be lowercase
# thumbnail: "/assets/img/default_thumbnail1.png"
---



# 개요

본 문서는 이하 Dockerfile을 보고 해석하기 위한 학습내용을 정리한 것이다.

```dockerfile
FROM golang:1.18 AS build-env
LABEL stage=build-env
WORKDIR /dir
COPY . /dir
RUN mkdir /root/dir
RUN CGO_ENABLED=0 GOOS=linux go build --mod=vendor -o build/dir main/main.go

FROM scratch
COPY --from=build-env /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=build-env /dir/build/dir /usr/local/bin/
ENV PATH=/usr/local/bin:$PATH
WORKDIR /root/dir
EXPOSE 50251
ENTRYPOINT ["dir"]
CMD ["up", "--grpc-port=50251"]
```

위 dockerfile을 해석하자면, 다음과 같은 과정으로 실행되게 된다.

TODO



# 포맷

위 dockerfile의 예시처럼 작성하며, 사실 FROM, LABEL 등의 Instruction을 대문자로 작성해야만 하는 것은 아니다. 대소문자를 구별하진 않으나, 개발자가 용이하게 인수와 쉽게 구별하기 위해 대문자로 표기한다.

줄바꿈 : TODO

# FROM

어떤 이미지를 기반으로 만들 것인지에 대한 설명이다.

만약, `FROM golang:1.18` 한줄만 dockerfile에 작성되어있다면, dockerbuild 시 golang 이미지가 빌드될 것이다.

그런데, 위 예시와 같이 FROM이 여러개 올 수 있다. 이런 경우... 이미지가 동시에 여러 개 생성된다는데...

# RUN

커맨드라인에 입력하는 것과 동일한 동작을 한다. 예를들어 우리 예시에서,

`RUN mkdir /root/dir`

역시 그냥 `mkdir /root/dir`하는 것과 동일하다.


# LABEL

라벨 작성. docker inspect 명령으로 label을 확인할 수 있다는데...

# WORKDIR

`cd` 명령어처럼 디렉토리를 이동한다. 상대경로/절대경로가 다 가능하다.


# COPY

파일을 복사한다. 사실, 위 예시의 `COPY . /dir`를,

`RUN cp -r . /dir`로 해도 되지 않을까??

# ENV

예시코드 `ENV PATH=/usr/local/bin:$PATH`처럼, 환경변수를 설정한다.

docker을 실행시키게 되면, 컨테이너 가상머신 위에 돌아가기때문에 이와 같은 방법으로 별도의 환경변수 설정이 필요하다.

# EXPOSE

서비스 배포할 포트 정의.

```dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```

위와 같이 작성하며, tcp 혹은 udp를 명시하지 않았을 경우의 기본값은 tcp다.

예를 들어 docker run을 할때, EXPOSE를 미리 지정해주지 않았다면 다음과 같이 어떤 포트로 실행할지 명시해야한다.

`$ docker run -p 80:80/tcp -p 80:80/udp`

혹은 dockerfile에 expose를 작성해두었더라도, 위 명령어와 같이 재명시하여 run 하면 해당 설정으로 변경되어 실행된다.


# ENTRYPOINT, CMD

## ENTRYPOINT

두가지 작성법이 있다.

1. `ENTRYPOINT ["executable", "param1", "param2"]`

2. `ENTRYPOINT command param1 param2`

여러번 작성하여도 맨 마지막에 설정된 하나만 적용된다.

설정한 커맨드를, docker run할때, 뒤에 넣은 파라미터들을 넣어 실행하게 된다.

우리 예시의 경우, `ENTRYPOINT ["dir"]` 로 작성되었으니, ...????? TODO

# CMD

기본적으로 ENTRYPOINT와 동일한 기능이지만, 사용자가 docker run 할때 인자값을 추가하여 실행하게 되면, 이게 실행되지 않고 대신 사용자가 추가한 설정으로 덮어씌워져 실행되게 된다.

예를 들어 본 예시의 경우, `CMD ["up", "--grpc-port=50251"]`인데, 실행할 때

`docker run --name dir dir/tag up "--grpc-port=50000` 으로 하면 포트값이 변경되어 up되게 될 것이다.

# ARG

# VOLUME

# ADD

# escape





