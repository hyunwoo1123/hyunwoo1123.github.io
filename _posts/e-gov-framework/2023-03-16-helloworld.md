---
title: 전자정부 표준프레임워크(egov standard framework) Hello world Tutorial
categories: [egov]
tags: [egov]     # TAG names should always be lowercase
---

본 문서는 전자정부 표준프레임워크의 구조(MVC패턴)에 대한 설명을 포함하여 차근차근 간단한 Web Project Helloworld 예제를 작성하는 가이드입니다.

우선 본 문서를 열람하기 전에, [설치 가이드](https://hyunwoo1123.github.io/posts/tutorial/)를 잘 따라서 진행해 주세요.

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




