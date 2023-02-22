---
title: github blog가 로컬에서는 잘 되는데 push하면 디자인 없이 index.html만 뜰 때
date: 2023-02-21 14:04:00 +/-TTTT
categories: [Blog]
tags: [blog, trubleshooting]     # TAG names should always be lowercase
# thumbnail: "/assets/img/default_thumbnail1.png"
---

내 경우 _config.yml 파일에 디폴트 썸네일 이미지를 수정하려고 구글링하여 찾아낸 headers를 덧붙였는데, 이게 로컬에서 빌드할때나 push했을때도 빌드는 정상적으로 수행되지만

디자인 없이 빈 index.html만 표기되는 황당한 일이 발생했다.

config 파일에 문제가 있으면 local에서는 정상동작하지만, 배포했을 때 문제가 생기는 경우가 있다.


2.22 추가

그냥 단순히 아무 문제가 없더라고 github에서 배포 pipline이 돌아가는 과정에서 에러가 생겨 안되는 경우가 생각보다 잦게 일어난다.

이 경우 그냥 다시 깃허브에서 rerun all-jobs를 수행하다보면 된다.

이거 나중에 계속 문제생기면 싸그리 갈아엎어야할수도...



