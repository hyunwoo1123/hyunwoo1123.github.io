---
title: 전자정부 표준프레임워크(egov standard framework) Hello world Tutorial
categories: [egov]
tags: [egov]     # TAG names should always be lowercase
---

본 문서는 전자정부 표준프레임워크의 구조(MVC패턴)에 대한 설명을 포함하여 차근차근 간단한 Web Project Helloworld 예제를 작성하는 가이드입니다.

우선 본 문서를 열람하기 전에, [설치 가이드](https://hyunwoo1123.github.io/posts/install/)를 잘 따라서 진행해 주세요.

# 도메인을 통한 클라이언트의 접근 구조

위 설치 가이드를 통해 예제 프로젝트를 생성하고 실행했다면, `http://localhost:8080/프로젝트명/egovSampleList.do` 에 샘플 페이지가 보여지는 것을 확인할 수 있다.

어떤 과정을 통해 이 페이지에 접근이 가능한걸까?


사용자가 홈페이지에 주소 `http://localhost:8080/프로젝트명/`를 입력하면, 우선 `src/main/webapp` 디렉토리 내에 위치하는 `index.jsp`을 찾아간다.

그리고 그 `index.jsp` 내부의 코드를 보면

```jsp
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<jsp:forward page="egovSampleList.do"/>
```

로 되어있고, 이 코드를 통해 즉시 `http://localhost:8080/프로젝트명/egovSampleList.do` 으로 이동된다.

그렇다면 이 `egovSampleList.do` 라는 url은 어디서 어떻게 인식하여 페이지를 보여주는걸까?

아래 스크린샷의 디렉토리에 위치하는 

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-helloworld-controller1.png)

`Controller` 에서 인식하게 된다.

```java
@RequestMapping(value = "/egovSampleList.do")
public String selectSampleList(@ModelAttribute("searchVO") SampleDefaultVO searchVO, ModelMap model) throws Exception {

    /** EgovPropertyService.sample */
    searchVO.setPageUnit(propertiesService.getInt("pageUnit"));
    searchVO.setPageSize(propertiesService.getInt("pageSize"));

    /** pageing setting */
    PaginationInfo paginationInfo = new PaginationInfo();
    paginationInfo.setCurrentPageNo(searchVO.getPageIndex());
    paginationInfo.setRecordCountPerPage(searchVO.getPageUnit());
    paginationInfo.setPageSize(searchVO.getPageSize());

    searchVO.setFirstIndex(paginationInfo.getFirstRecordIndex());
    searchVO.setLastIndex(paginationInfo.getLastRecordIndex());
    searchVO.setRecordCountPerPage(paginationInfo.getRecordCountPerPage());

    List<?> sampleList = sampleService.selectSampleList(searchVO);
    model.addAttribute("resultList", sampleList);

    int totCnt = sampleService.selectSampleListTotCnt(searchVO);
    paginationInfo.setTotalRecordCount(totCnt);
    model.addAttribute("paginationInfo", paginationInfo);

    return "sample/egovSampleList";
}
```

여기서 잡다한 코드를 제외하고

```java
	@RequestMapping(value = "/egovSampleList.do")
	public String selectSampleList() throws Exception {
		return "sample/egovSampleList";
	}
```

만 우선 살펴보자.

우선 `@RequestMapping`은, 어떤 url에 대하여 반응할 것인지를 지정한다.

`value = "/egovSampleList.do"`부분을 통해 `egovSampleList.do` url이 입력되었을 때 `selectSampleList()`가 동작하게 된다.

그리고 `selectSampleList()`는 `return "sample/egovSampleList";` 을 통해 실제로 사용자에게 보여줄 페이지를 정해준다.

그리고 그 페이지는 아래 그림처럼, `src/main/webapp/WEB-INF/jsp/egovframework/example/sample/` 폴더 안에 위치해있다.

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-helloworld-jsp1.png)

그런데 아까 Controller의 함수의 return값은 `WEB-INF/jsp/egovframework/example/sample/egovSampleList.jsp` 가 아니라,

`sample/egovSampleList` 였다. 이 앞쪽의 경로들과, 뒤의 확장자 jsp는 그러면 어디서 붙이는걸까?

이는 `webapp/WEB-INF/config/egovframework/springmvc/` 디렉토리에 위치하는 `dispatcher-servlet.xml` 파일에서 찾을 수 있다.

아래는 `dispatcher-servlet.xml`의 내부 코드이다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:p="http://www.springframework.org/schema/p"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
                http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">

    <context:component-scan base-package="egovframework">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>

    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="webBindingInitializer">
            <bean class="egovframework.example.cmmn.web.EgovBindingInitializer"/>
        </property>
    </bean>
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
        <property name="interceptors">
            <list>
                <ref bean="localeChangeInterceptor" />
            </list>
        </property>
    </bean>
    
    <bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver" />
    <!-- 쿠키를 이용한 Locale 이용시 <bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver"/> -->
    <bean id="localeChangeInterceptor" class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
        <property name="paramName" value="language" />
    </bean>

    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="defaultErrorView" value="cmmn/egovError"/>
        <property name="exceptionMappings">
            <props>
                <prop key="org.springframework.dao.DataAccessException">cmmn/dataAccessFailure</prop>
                <prop key="org.springframework.transaction.TransactionException">cmmn/transactionFailure</prop>
                <prop key="org.egovframe.rte.fdl.cmmn.exception.EgovBizException">cmmn/egovError</prop>
                <prop key="org.springframework.security.AccessDeniedException">cmmn/egovError</prop>
            </props>
        </property>
    </bean>

    <bean class="org.springframework.web.servlet.view.UrlBasedViewResolver" p:order="1"
	    p:viewClass="org.springframework.web.servlet.view.JstlView"
	    p:prefix="/WEB-INF/jsp/egovframework/example/" p:suffix=".jsp"/>

    <!-- For Pagination Tag -->
    <bean id="imageRenderer" class="egovframework.example.cmmn.web.EgovImgPaginationRenderer"/>

    <bean id="paginationManager" class="org.egovframe.rte.ptl.mvc.tags.ui.pagination.DefaultPaginationManager">
        <property name="rendererType">
            <map>
                <entry key="image" value-ref="imageRenderer"/>
            </map>
        </property>
    </bean>
	<!-- /For Pagination Tag -->

    <mvc:view-controller path="/cmmn/validator.do" view-name="cmmn/validator"/>
</beans>
```

위 코드의 49번째 줄

`p:prefix="/WEB-INF/jsp/egovframework/example/" p:suffix=".jsp"/>`

부분이 이를 설정하고있다!


## 정리

따라서 정리하자면, `Controller`에서 `/egovSampleList.do` url을 인식하고 `dispatcher-servlet.xml`의 설정을 참고하여 `WEB-INF/jsp/egovframework/example/sample/egovSampleList.jsp`를 화면에 표시하게 되는 것이다.


# HelloWorld 출력 페이지 만들어보기

위 파일 구조에 대한 이해를 바탕으로 HelloWorld 예제를 직접 만들어보자.

필요한 것은 Helloworld를 출력해줄 jsp파일과, 그걸 연결해주기 위해 Controller을 만들고 dispatcher-servlet.xml 파일을 수정할 것이다.


## Helloworld를 출력하는 jsp 파일 만들기

`index.jsp`가 위치한 디렉토리에 `helloworld.jsp` 파일을 만들어보자. 예제로 필자가 만든 코드는 다음과 같다.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<% out.print("Hello World!!"); %>
</body>
</html>
```

가장 간단하게 jsp를 통해 Hello World를 출력하는 jsp 예제이다.

## dispatcher-servlet.xml 수정하기

index.jsp와 동일한 위치 즉, 패키지 폴더 내에 `src/main/webapp` 디렉토리에 `helloworld.jsp`를 만들었다.

하지만 현재 `dispatcher-servlet.xml`의 49번째 줄에서는, 

`p:prefix="/WEB-INF/jsp/egovframework/example/" p:suffix=".jsp"/>`

코드를 통해 기본 디렉토리를 `src/main/webapp/WEB-INF/jsp/egovframework/example/` 으로 설정하고있다.

따라서 49번째 줄의 내용을

`p:prefix="/" p:suffix=".jsp"/>`

로 수정한다.

## Controller 만들어보기

이제 `helloworld.jsp`를 출력하게 할, Controller가 필요하다.

물론 기본적으로 샘플로 제공되던 `EgovSampleController.java` 내에 단순히 `helloworld.go` 를 캐치하는 Controller 메소드를 하나 더 만들어도 되지만, 컨트롤러는 기능별로 다른 java 파일에서 수행하는 것이 좋다.

따라서 이번 helloworld에서는 helloworld만의 Controller을 따로 만들어보자.

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-helloworld-controller2.png)

새로운 패키지 `egovframework.helloworld.controller`를 만들고, 그 안에 `MyController.java` 파일을 생성하였다. 내부 코드는 다음과 같다.

```java
package egovframework.helloworld.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class MyController {
	@RequestMapping(value = "/helloworld.do")
	public String helloWorld() {

		return "helloworld";
	}
}
```

단순하게 helloworld.do 주소를 입력받으면, helloworld 라는 값을 return하고 있다.

이 값이 아까 수정한 설정파일 `dispatcher-servlet.xml`의 `p:prefix="/" p:suffix=".jsp"/>` 과 합쳐져

`index.jsp`가 위치한 `webapp` 디렉토리 내의 `helloworld.jsp` 파일을 연결하게 된다.

그 후 서버를 실행하여 주소창에 `localhost:8080/프로젝트명/helloworld.do`를 입력하게 되면,

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-helloworld-result1.png)

와 같은 화면이 출력되게 된다.

위 실습을 통해 우리는 전자정부 표준 프레임워크의 대략적인 구조를 훑어보고 간단한 HelloWorld를 출력하는 페이지를 만들어보았다.




# 데이터 주고받기(GET,POST)

이제 데이터를 주고받는 기능을 사용해보자.

## form을 통해 GET,POST 데이터 전송(Client to server)

form을 이용해 GET/POST 방식으로 데이터를 주고받아보자.

우선 form을 작성할 수 있는 페이지를 만들어야 한다.

`index.jsp`가 있는 위치에, `form_client_to_server.jsp` 파일을 추가로 만든다.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>form으로 GET POST 전송하기</title>
</head>
<body>

<form name="f1" method="post" action="post_client_to_server.do">
<table>
	<tr>
		<th>POST로 전송할 데이터:</th>
		<td><input type="text" name="data"></td>
</table>
</form>

<form name="f2" method="get" action="get_client_to_server.do">
<table>
	<tr>
		<th>GET으로 전송할 데이터:</th>
		<td><input type="text" name="data"></td>
</table>
</form>

</body>
</html>
```

MyController.java 에도 다음 메소드를 추가한다.

```java
	@RequestMapping(value = "form_client_to_server.do")
	public String formPostClientToServer (String data) {
		System.out.println(data);
		return "form_client_to_server";
	}

	//Post로 서버에서 클라이언트가 준 데이터 받
	@RequestMapping(value = "/post_client_to_server.do", method = RequestMethod.POST)
	public String postClientToServer (String data) {
		System.out.println(data);
		return "";
	}
	
	//Get로 서버에서 클라이언트가 준 데이터 받
	@RequestMapping(value = "/get_client_to_server.do", method = RequestMethod.GET)
	public String getClientToServer (String data) {
		System.out.println(data);
		return "";
	}
```

그 후 `localhost:8080/프로젝트명/form_client_to_server.do` 에 들어가면, 다음과 같은 화면을 확일할 수 있다.

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-helloworld-form1.png)

그 후 예를들어 다음과 같이 입력 후 엔터를 누르게 된다면,

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-helloworld-form2.png)

결과 : 

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-helloworld-post.png)

Console에 Post로 전달된 데이터가 출력되는 것을 확인할 수 있다.

마찬가지로, 아래와 같이 GET 방식으로 데이터를 전달하여도

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-helloworld-form3.png)

결과 : 

![egov-make-new-project예제사진](https://hyunwoo1123.github.io/assets/img/egov/egov-helloworld-get.png)


똑같이 Console에 전달된 데이터가 출력된다.

## java서버에서 jsp로 데이터 보내기(Server to Client)

방법이 3가지가 있다. 3가지 예제를 모두 추가하여 테스트해보자.

우선 `MyController.java`에 다음 메소드들을 추가하자

```java
	@RequestMapping(value = "/server_to_client1.do")
	public ModelAndView serverToClient1(ModelAndView mv) {
		mv.addObject("data", "서버에서 보낸 데이터1");
		mv.setViewName("server_to_client1");
		return mv;
	}

	@RequestMapping(value = "/server_to_client2.do")
	public String serverToClient2(Model model) {
        model.addAttribute("data", "서버에서 보낸 데이터2");
		return "server_to_client1";
	}
	
	@RequestMapping(value = "/server_to_client3.do")
	public String serverToClient2(HttpServletRequest request) {
        request.setAttribute("data", "서버에서 보낸 데이터3");
        return "server_to_client2";
	}
```

맨 위에서부터 각각 `ModelAndView`,`Model`,`HttpServletRequest` 로 데이터를 전송하는 방법이다.

그리고 이 데이터를 받아 출력할 jsp파일 2개를 추가하자. `ModelAndView`,`Model`로 전송한 데이터는 클라이언트에서 받는 방법이 동일하기 때문에 3개가 아니라 2개의 jsp를 추가한다.

1. server_to_client1.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>서버에서 받은 데이터</title>
</head>
<body>
    <h1>${data}</h1>
</body>
</html>
```

2. server_to_client2.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>서버에서 받은 데이터</title>
</head>
<body>
    <h1>${requestScope.data}</h1>
</body>
</html>
```

그 후 `localhost:8080/프로젝트명/server_to_client1.do`, `localhost:8080/프로젝트명/server_to_client2.do`, `localhost:8080/프로젝트명/server_to_client3.do`

url에 접속하면 각 데이터가 클라이언트로 전송되어 출력되는 것을 확인할 수 있다.

<!-- # 데이터베이스 연결 -->
# 어노테이션(Annotation)

egov framework의 java 파일들에는 `@` 로 시작하는 다양한 `Annotation`이 존재한다.

본래 Annotation 단어의 의미는 주석이라는 의미이지만, Spring 구조에서는 무언가 선언을 하는 용도로 사용된다.

[위 예시](#도메인을-통한-클라이언트의-접근-구조)에서도 `@RequestMapping(value = "/egovSampleList.do")` 라는 Annotation을 활용한 것을 볼 수 있다.

그중 본 예제에 사용된 용어들에 대한 간단한 설명을 하자면, 다음과 같다.

* @Controller
    - Controller을 선언할 때 사용한다. Controller는 요청을 처리하고 응답을 생성한다.
* @RequestMapping
    - URL 경로와 HTTP 요청 메소드에 따라 요청을 처리할 메소드를 지정한다.
* @Service
    - 해당 클래스는 서비스이며, DAO와 같이 데이터 처리를 담당하는 클래스에서 사용된다.
    - 스프링 구조의 Impl 클래스에 사용된다.
* @Repository
    - 데이터베이스 접근을 위한 클래스에 선언한다.

# 전자정부 표준프레임워크의 전체 구조

이제 본격적으로 `Client`, `Server`, `Database`가 서로 연결되어 정보를 제공하는, 하나의 구조를 갖춰가고 있다.

따라서 본 실습에 앞서 전자정부 표준프레임워크의 전체적인 서비스 구조와 [mvc 패턴](#)이 어떤 방식으로 적용되는지 알아야한다.

전자정부 표준 프레임워크의 동작 구조를 가장 간단하게 표현하자면 다음과 같다.

![egov mvc 구조도](https://hyunwoo1123.github.io/assets/img/egov/mvc/egov0.svg)

사용자가 `url`을 통해 요청을 하면, `Controller`에서 그것을 캐치하여 그에 맞는 `Data`를 반환한다. 아주 쉽고 깔끔하다.

이걸 조금 더 자세하게 표현해보자

`사용자`가 `url`을 통해 요청할 때, 예를 들어 `로그인`을 하려 한다면 자신의 `id, password`를 함께 전달할 것이다.

이렇게 `사용자`가 `url`과 더불어 다양한 `data`들을 보내면, `Controller`는 그 Data들을 VO(Value Object)를 통해 한번에 묶어서 받게 된다.

이걸 위 그림에 추가하여 표현한다면 다음과 같다.

![egov mvc 구조도](https://hyunwoo1123.github.io/assets/img/egov/mvc/egov1.svg)

그 후 Controller에서는, `해당 id, password가 올바른 값이 들어왔는지 확인하고, Database에 연결하여 해당 계정이 Database에 존재하는지 확인하는 등 복잡한 과정`을 거쳐, 그 결과를 사용자에게 보낼 것이다.

이 `복잡한 과정`이라 표현한 부분을 우선 뭉뚱그려 `Service`라고 표현하자.(MVC 패턴에서는 이 `Service`를 `Model` 이라 표현한다.)

이걸 다시 그림에 추가하면 다음과 같다.

![egov mvc 구조도](https://hyunwoo1123.github.io/assets/img/egov/mvc/egov2.svg)


그림에서는 `VO`를 더 그려넣진 않았으나, 내부적으로 데이터를 공유하거나 이동시킬때에도 `VO`를 사용하고 있다고 이해하면 된다.

이제 이 `Service` 내부에서는 어떤 일이 일어나고있는지 더 자세히 알아보자.

Service에는 해당 Service가 어떤 기능을 수행하는지에 대한 morkup인 `interface`,

그 interface의 기능을 실제로 구현하는 class인 `impl`,

그리고 데이터베이스와의 연결 부분을 담당하는, `sql`문을 호출하는 `DAO`,

그리고 그 `sql`들을 선언해 둔 `.xml` 파일이 있다.

이것까지 추가하면 최종적으로 다음과 같은 그림이 된다.

![egov mvc 구조도](https://hyunwoo1123.github.io/assets/img/egov/mvc/egov3.svg)


TODO 각 기능들에 대한 링크 달아두고 상세한 설명 더 추가하기


# 데이터베이스 연결하기(Database[Postgresql])

이제 이 db를 연결하고, 정보를 입력/출력하는 예제를 만들어보자.

Postgresql을 설치하고, `test_db`를 만들고, 그 내부에 테스트용으로 `books` 라는 테이블을 아래와 같은 sql으로 만들었다.

postgresql 관련 설치 및 사용법은 [링크 참고](#)

```sql
CREATE TABLE books (
  id              SERIAL PRIMARY KEY,
  title           VARCHAR(100) NOT NULL,
  primary_author  VARCHAR(100) NULL
);
```

그리고 더미 데이터를 조금 insert 한다.

```sql
insert into books(title,primary_author) values ('해리포터','조엔롤링');
insert into books(title,primary_author) values ('노인과 바다','헤밍웨이');
insert into books(title,primary_author) values ('코스모스','칼 세이건');
```


TODO sql.xml은 mybatis 문법을 사용한다... 요거 사용법 추가하자...


