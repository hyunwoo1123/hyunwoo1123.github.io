---
title: 전자정부 표준프레임워크(e gov standard framework) 설치(Mac M1)
categories: [egov]
tags: [egov]     # TAG names should always be lowercase
---

본 문서는 전자정부 표준프레임워크 개발환경 4.1 버전을 Mac M1환경에서 설치하려니 온갖 오류가 발생하여, 실행에 성공한 방법을 정리한 문서임.


1. [다운로드 링크](https://www.egovframe.go.kr/home/sub.do?menuNo=94)에서, `개발자용 개발환경 for Mac (x86_64/AArch64) (Implementation Tool) Version 4.1.0`를 클릭하고, `eGovFrameDev-4.1.0-Mac-AArch64.dmg` 를 설치한다.

2. 설치하면 파일이 손상되었다며 즉시 삭제하기 창이 뜨지만 무시하고 닫는다.

3. 터미널을 열어 `xattr -cr "/Applications/eGovFrameDev-4.1.0-Mac-AArch64.app"` 를 입력하여, 설치한 프로그램의 모든 확장 속성을 제거한다.

4. 응용 프로그램에서, 설치한 `eGovFrameDev-4.1.0-Mac-AArch64`를 다시 실행한다. 이때는 정상적으로 실행될 것이다.

5. 그 후, 다음과 같이 new -> project -> eGovFrame Web Project 를 만든다

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project1.png)

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project2.png)

finish가 아니라 next를 눌러 다음과 같이 `Generate Sample Code`를 체크한다

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project2-1.png)

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project2-2.png)


* 서버를 추가한다. 이때, 현재 4.1 버전의 전자정부 표준프레임워크는 tomcat10 버전에서는 오류가 발생하므로, 버전 8 혹은 9로 설치한다. 추가하는 방법은 다음과 같다.

우선, 프로잭트를 생성했다면 그 후 프로잭트의 아래쪽에 다음 그림과 같이`Terminal`, `Server`등이 있는 탭을 찾을 수 있을 것이다.

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project3-0.png)

만약, 이 탭이 보이지 않는다거나 실수로 없애버렸다면, 다음 방법을 통해 다시 보이게 할 수 있다.

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project3-01.png)

우리는 이 탭에서 `Server` 탭을 클릭한 후, `No servers are available. Click this link to create a new server` 링크를 클릭한다.

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project3.png)

Tomcat 버전을 선택한다. 현재 4.1 버전의 전자정부 표준프레임워크는 tomcat10 버전에서는 오류가 발생하므로, 버전 8 혹은 9로 설치한다. 그 후 next를 클릭한다

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project4.png)

그러면 아래와 같은 화면이 뜨는데, 여기서 `Download and Install` 버튼을 클릭한다.

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project4-1.png)

동의하고 진행한다

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project4-2.png)

그러면 다음과 같은 화면으로 돌아오게 된다. 이때 **중요한 것은** Finish 버튼이 바로 활성화되지 않으나 내부적으로는 설치가 진행되고 있는 상황이라, Finish 버튼이 활성화 될 때 까지 기다려야 한다!!

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project4-3.png)

Finish 버튼이 활성화되면, 클릭하여 설치를 완료한다.


* 실행

프로젝트 폴더를 우클릭하여 `Run as` -> `Run on server`를 클릭한다.

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project5.png)

그러면 다음과 같이 실행할 서버를 고르는 창이 나오는데, 여기서 방금 설치한 tomcat을 고르고 `Finish`를 클릭한다.

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project6.png)

다음과같은 화면이 최종적으로 출력된다.

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-make-new-project7.png)


