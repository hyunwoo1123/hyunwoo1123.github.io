---
title: 전자정부 표준프레임워크(egov standard framework) 공통 컴포넌트 all_in_one 설치&초기 실행 Tutorial (ver.4.1)
categories: [egov]
tags: [egov]     # TAG names should always be lowercase
---

본 문서는 전자정부 표준프레임워크 4.1 버전의 공통 컴포넌트 all_in_one의 설치 및 세팅을 가이드하는 문서이다.

**전자정부 프레임워크 개발환경 설치 가이드**는 [이곳](https://hyunwoo1123.github.io/posts/egov-install/)에서,

**전자정부 프레임워크에 대한 기본적인 구조 설명 / 튜토리얼**은 [이곳](https://hyunwoo1123.github.io/posts/egov-helloworld/)에서 확인할 수 있고, 위 사항들에 대한 이해가 본 문서를 보기 전에 필요하다.

본 문서에서는 데이터베이스로 postgresql을 사용한다.

# 공통 컴포넌트란?

* 공통컴포넌트는 정보시스템 구축시 공통적으로 재사용이 가능한 기능위주로 개발한 컴포넌트의 집합
* 공통컴포넌트는 표준프레임워크 기반으로 실행환경의 MVC아키텍처를 준수하여 설계 및 개발
* 전자정부사업에서 쉽게 커스트마이징하여 재사용할수 있도록 전자정부 표준프레임워크 포털(www.egovframe.go.kr)을 통해 소스코드와 가이드를 제공
* 빌드 및 실행

![공통 컴포넌트 공식설명 이미지](https://hyunwoo1123.github.io/assets/img/egov/all_in_one_1.png)

![공통 컴포넌트 공식설명 이미지](https://hyunwoo1123.github.io/assets/img/egov/all_in_one_2.png)

![공통 컴포넌트 공식설명 이미지](https://hyunwoo1123.github.io/assets/img/egov/all_in_one_3.png)

# 설치

총 진행할 과정을 미리 살펴보면, 다음과 같다.

* 공통 컴포넌트 다운로드
* Project에 Import
* Database에 Table, 샘플 데이터 추가
* Database 연결정보 설정

## 공통 컴포넌트 다운로드

1. [링크](https://www.egovframe.go.kr/home/sub.do?menuNo=49)에서 **공통컴포넌트 4.1.0 all-in-one 배포파일**을 다운로드 받는다


## project에 import

2. egov 개발환경 이클립스를 실행하여, File > new > eGovFrame web project를 생성한다.

![공통 컴포넌트 튜토리얼 이미지](https://hyunwoo1123.github.io/assets/img/egov/egov_components_1.png)

![공통 컴포넌트 튜토리얼 이미지](https://hyunwoo1123.github.io/assets/img/egov/egov_components_2.png)

3. 2번에서 만든 프로젝트를 마우스 우클릭하여 import > Archive File > Next를 클릭하고, 1번에서 다운로드받은 all_in_one zip 파일을 선택하여 import한다.(전부 덮어쓰기한다)

![공통 컴포넌트 튜토리얼 이미지](https://hyunwoo1123.github.io/assets/img/egov/egov_components_3.png)

![공통 컴포넌트 튜토리얼 이미지](https://hyunwoo1123.github.io/assets/img/egov/egov_components_4.png)

![공통 컴포넌트 튜토리얼 이미지](https://hyunwoo1123.github.io/assets/img/egov/egov_components_5.png)


## Database에 Table, 샘플 데이터 추가

우리가 만든 프로젝트의 script 폴더에는 **comment,  ddl, dml** 폴더가 존재한다.

![공통 컴포넌트 튜토리얼 이미지](https://hyunwoo1123.github.io/assets/img/egov/egov_components_6.png)

ddl 디렉토리 내에 존재하는 sql은 Table을 추가하는 sql들이 들어있고,

dml에는 그 Table에 추가할 데이터들을 입력하는 sql이 위치한다.

comment는 각 Table들에 대한 주석을 덧붙이는 sql이 존재한다.

따라서 본 프로젝트에서는 postgresql database를 사용하므로, ddl > dml > comment 순서로 sql을 database에 실행하여 table 및 각종 필요한 데이터를 추가한다.


## database 연결정보 설정

데이터베이스 정보를 입력한 후, 연결 정보를 설정해야 한다.

연결 정보 설정 과정은 다음과 같다.

* database 연결 password 암호화
* globals.properties 정보 수정
* context-datasource.xml 정보 수정

관련 파일들에 대한 간략한 설명을 덧붙이자면 다음과 같다.

context-datasource.xml (위치 : 프로젝트/src/main/resources/egovframework/spring/com/context-datasource.xml)
 - globals.properties에서 db username, url, password 등의 정보를 가져와 db 연결을 설정함.

globals.properties (위치 : 프로젝트/src/main/resources/egovframework/egovProps/globals.properties)
 - 각종 global한 상수를 선언함. 특히, 어떤 database를 사용하는지, userid, password가 무엇인지 등도 설정함.

### database 연결 password 암호화

위 설명에 따라서, database에 연결하기 위해서는 우선 globals.properties에 어떤 database를 사용할 것인지, password, url, username등에 대해 설정해주어야 한다.

하지만 암호화 정책(globals.properties의 33~36줄의 comment 참고)에 의해 password를 암호화하는 과정이 필요하다.

해당 암호화 코드가 all_in_one에 포함되어있지 않고 따로 설명과 예제를 제공하기때문에, 본 튜토리얼에서도 암호화 코드를 따로 작성하여 암호화를 수행한다.

암호화 과정은 먼저 암호화 algorithmKeyHash를 만들고, 그 Hash를 통해 database password를 암호화해야한다.

따라서 먼저 algorithmKeyHash를 만드는 **EgovEnvCryptoAlgorithmCreateTest.java** 파일을 `프로젝트/src/test/java`에 만든다.

**EgovEnvCryptoAlgorithmCreateTest.java**
```java
import org.egovframe.rte.fdl.cryptography.EgovPasswordEncoder;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
 
public class EgovEnvCryptoAlgorithmCreateTest {
 
	private static final Logger LOGGER = LoggerFactory.getLogger(EgovEnvCryptoAlgorithmCreateTest.class);
 
	//계정암호화키 키
	public String algorithmKey = "egovframe";
 
	//계정암호화 알고리즘(MD5, SHA-1, SHA-256)
	public String algorithm = "SHA-256";
 
	//계정암호화키 블럭사이즈
	public int algorithmBlockSize = 1024;
 
	public static void main(String[] args) {
		EgovEnvCryptoAlgorithmCreateTest cryptoTest = new EgovEnvCryptoAlgorithmCreateTest();
 
		EgovPasswordEncoder egovPasswordEncoder = new EgovPasswordEncoder();
		egovPasswordEncoder.setAlgorithm(cryptoTest.algorithm);
 
		LOGGER.info("------------------------------------------------------");
		LOGGER.info("알고리즘(algorithm) : "+cryptoTest.algorithm);
		LOGGER.info("알고리즘 키(algorithmKey) : "+cryptoTest.algorithmKey);
		LOGGER.info("알고리즘 키 Hash(algorithmKeyHash) : "+egovPasswordEncoder.encryptPassword(cryptoTest.algorithmKey));
		LOGGER.info("알고리즘 블럭사이즈(algorithmBlockSize)  :"+cryptoTest.algorithmBlockSize);
 
	}
}
```

이 코드를 Run as Java application으로 실행하면,

```
INFO [all1.EgovEnvCryptoAlgorithmCreateTest] ------------------------------------------------------
INFO [all1.EgovEnvCryptoAlgorithmCreateTest] 알고리즘(algorithm) : SHA-256
INFO [all1.EgovEnvCryptoAlgorithmCreateTest] 알고리즘 키(algorithmKey) : egovframe
INFO [all1.EgovEnvCryptoAlgorithmCreateTest] 알고리즘 키 Hash(algorithmKeyHash) : gdyYs/IZqY86VcWhT8emCYfqY1ahw2vtLG+/FzNqtrQ=
INFO [all1.EgovEnvCryptoAlgorithmCreateTest] 알고리즘 블럭사이즈(algorithmBlockSize)  :1024
```

위와 같은 값이 출력된다.

이 값들을 `프로젝트/src/main/resources/egovframework/spring/com/context-crypto.xml`에 적어줘야 하는데, 

우리가 다운받은 all_in_one 패키지는 기본적으로 `context-crypto.xml`의 내용에 위 결과와 동일한, egovframe 키로 SHA-256, 1024 블럭사이즈로 생성한 Hash값이 입력되어있다.

따라서 원하는 키, 원하는 블럭사이즈로 재설정하여 `context-crypto.xml`의 내용을 수정하면 된다.


-----------

이제 알고리즘 Key hash를 만들었으니, 본격적으로 Database password를 암호화해야한다.

암호화하는 `EgovEnvCryptoUserTest.java` 파일을 `프로젝트/src/test/java` 생성한다.

```java
import org.egovframe.rte.fdl.cryptography.EgovEnvCryptoService;
import org.egovframe.rte.fdl.cryptography.impl.EgovEnvCryptoServiceImpl;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class EgovEnvCryptoUserTest {
 
	private static final Logger LOGGER = LoggerFactory.getLogger(EgovEnvCryptoUserTest.class);
 
	public static void main(String[] args) {
 
		String[] arrCryptoString = { 
		"",         //데이터베이스 접속 계정 설정
		"postgres",   //데이터베이스 접속 패드워드 설정
		"",            //데이터베이스 접속 주소 설정
		""  //데이터베이스 드라이버
              };
 
 
		LOGGER.info("------------------------------------------------------");		
		ApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"classpath:/egovframework/spring/com/context-crypto.xml"});
		EgovEnvCryptoService cryptoService = context.getBean(EgovEnvCryptoServiceImpl.class);
		LOGGER.info("------------------------------------------------------");
 
		String label = "";
		try {
			for(int i=0; i < arrCryptoString.length; i++) {		
				if(i==0)label = "사용자 아이디";
				if(i==1)label = "사용자 비밀번호";
				if(i==2)label = "접속 주소";
				if(i==3)label = "데이터 베이스 드라이버";
				LOGGER.info(label+" 원본(orignal):" + arrCryptoString[i]);
				LOGGER.info(label+" 인코딩(encrypted):" + cryptoService.encrypt(arrCryptoString[i]));
				LOGGER.info("------------------------------------------------------");
			}
		} catch (IllegalArgumentException e) {
			LOGGER.error("["+e.getClass()+"] IllegalArgumentException : " + e.getMessage());
		} catch (Exception e) {
			LOGGER.error("["+e.getClass()+"] Exception : " + e.getMessage());
		}
 
	}
 
}
```

password만 암호화하면 되기때문에, 16번째 줄의 데이터베이스 접속 패드워드를 입력한다. 본 튜토리얼에서 필자는 테스트용 password로 `postgres`를 사용했다.

단, 이 java 파일을 실행하면, 아래와 같은 에러가 발생한다.

`org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'egovEnvCryptoConfigurerService' defined in InputStream resource [resource loaded through InputStream]: Invocation of init method failed; nested exception is org.springframework.context.NoSuchMessageException: No message found under code 'error.properties.initialize.reason' for locale 'ko_KR'.`

이를 방지하기 위해 `context-crypto.xml`에 `messageSource` 관련 bean 내용을 추가해주어야 한다.

추가하기 전, `context-crypto.xml`의 코드 : 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:egov-crypto="http://maven.egovframe.go.kr/schema/egov-crypto"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://maven.egovframe.go.kr/schema/egov-crypto http://maven.egovframe.go.kr/schema/egov-crypto/egov-crypto-4.1.0.xsd">

	<!-- 
	initial : globals.properties 연계 Url, UserName, Password 값 로드 여부(설정값 : true, false)
	crypto : 계정 암호화 여부(설정값 : true, false)
	algorithm : 계정 암호화 알고리즘
	algorithmKey : 계정 암호화키 키
	cryptoBlockSize : 계정 암호화키 블록사이즈
	cryptoPropertyLocation : 설정파일 암복호화 경로 (선택) 기본값은 'classpath:/egovframework/egovProps/globals.properties'
	-->
    <egov-crypto:config id="egovCryptoConfig" 
    	initial="true"
    	crypto="true"
    	algorithm="SHA-256"
    	algorithmKey="egovframe"
    	algorithmKeyHash="gdyYs/IZqY86VcWhT8emCYfqY1ahw2vtLG+/FzNqtrQ="
		cryptoBlockSize="1024"
		cryptoPropertyLocation="classpath:/egovframework/egovProps/globals.properties"
	/>
 
</beans>
```

`messageSource` 관련 내용을 추가한 `context-crypto.xml`의 코드 : 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:egov-crypto="http://maven.egovframe.go.kr/schema/egov-crypto"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://maven.egovframe.go.kr/schema/egov-crypto http://maven.egovframe.go.kr/schema/egov-crypto/egov-crypto-4.1.0.xsd">

	<!-- 
	initial : globals.properties 연계 Url, UserName, Password 값 로드 여부(설정값 : true, false)
	crypto : 계정 암호화 여부(설정값 : true, false)
	algorithm : 계정 암호화 알고리즘
	algorithmKey : 계정 암호화키 키
	cryptoBlockSize : 계정 암호화키 블록사이즈
	cryptoPropertyLocation : 설정파일 암복호화 경로 (선택) 기본값은 'classpath:/egovframework/egovProps/globals.properties'
	-->
    <egov-crypto:config id="egovCryptoConfig" 
    	initial="true"
    	crypto="true"
    	algorithm="SHA-256"
    	algorithmKey="egovframe"
    	algorithmKeyHash="gdyYs/IZqY86VcWhT8emCYfqY1ahw2vtLG+/FzNqtrQ="
		cryptoBlockSize="1024"
		cryptoPropertyLocation="classpath:/egovframework/egovProps/globals.properties"
	/>
    <!--추가 부분-->
    <bean name="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="useCodeAsDefaultMessage">
            <value>true</value>
        </property>
        <property name="basenames">
            <list>
                <value>classpath:/egovframework/egovProps/globals</value>
            </list>
        </property>
    </bean>
    <!--추가 부분-->
</beans>
```

그 후 다시 `EgovEnvCryptoUserTest.java`를 실행하면, 다음과 같은 결과를 얻을 수 있다.

```
------------------------------------------------------
사용자 아이디 원본(orignal):
사용자 아이디 인코딩(encrypted):
------------------------------------------------------
사용자 비밀번호 원본(orignal):postgres
사용자 비밀번호 인코딩(encrypted):9OzT9pB%2Be45zOrWcBYwdaQ%3D%3D
------------------------------------------------------
접속 주소 원본(orignal):
접속 주소 인코딩(encrypted):
------------------------------------------------------
데이터 베이스 드라이버 원본(orignal):
데이터 베이스 드라이버 인코딩(encrypted):
------------------------------------------------------
```

이 값을 확인한 후, **반드시** 다시 `context-crypto.xml`에 추가했던 `messageSource` 관련 내용을 전부 **제거**하고 **원래 상태로 되돌려놓아야 한다**. 

이는 원래 해당 설정이 `context-common.xml`에 존재하기 때문으로, 만약 지우지 않으면 서로 **충돌**이 발생하여 서버가 정상적으로 실행되지 않는다.


### globals.properties 정보 수정

그 후, `EgovEnvCryptoUserTest.java`의 사용자 비밀번호 인코딩 결과값을 복사하여 globals.properties의 postgres 관련 항목에 입력하고, userid,url을 각자의 설정과 이름에 맞게 수정한다.

(위치 : 프로젝트/src/main/resources/egovframework/egovProps/globals.properties)

또한 Globals.DbType(20번째 줄)의 값을 postgres로 수정하고, Globals.OsType도 각자의 운영서버 타입에 맞는 값으로 수정한다.

아래는 그 예시로, 필자의 경우 예시를 위해 userId도 postgres, url도 jdbc:postgresql://127.0.0.1:5432/postgres, password도 postgres 로 사용하였다.

`globals.properties` :
```
...중략

# 운영서버 타입(WINDOWS, UNIX)
Globals.OsType = UNIX #필자의 경우 Mac OS이기 때문에 UNIX

# DB서버 타입(mysql, oracle, altibase, tibero, cubrid, maria, postgres, goldilocks) - datasource 및 sqlMap 파일 지정에 사용됨
Globals.DbType = postgres

...중략

#postgreSQL
Globals.postgres.DriverClassName=org.postgresql.Driver
Globals.postgres.Url=jdbc:postgresql://127.0.0.1:5432/postgres
Globals.postgres.UserName=postgres
Globals.postgres.Password=9OzT9pB%2Be45zOrWcBYwdaQ%3D%3D

...중략

```

### context-datasource.xml 정보 수정

이제 이 값을 가져다가 database 연결을 설정하는 `context-datasource.xml`를 수정해야한다.

db를 연결하는 부분 중 postgres를 제외한, 다른 db 연결을 모두 삭제하거나 주석처리한다. 필자의 경우 삭제하였다.

예시는 다음과 같다.

수정 전 `context-datasource.xml` : 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans 
	xmlns="http://www.springframework.org/schema/beans" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:util="http://www.springframework.org/schema/util" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd 
	 						http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

	<bean id="egov.propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath:/egovframework/egovProps/globals.properties</value>
                <!-- <value>file:/product/jeus/egovProps/globals.properties</value> -->
            </list>
        </property>
    </bean>
	
	<!-- DataSource -->
	<alias name="dataSource" alias="egov.dataSource" />
	
	<!-- MySQL -->
	<beans profile="mysql">  
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${Globals.mysql.DriverClassName}"/>
		<property name="url" value="${Globals.mysql.Url}" />
		<property name="username" value="${Globals.mysql.UserName}"/>
		<!-- 암호화(Crypto) 관련 서비스 https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte2:fdl:crypto_simplify_v3_8 참조 -->
		<property name="password" value="#{egovEnvCryptoService.getPassword()}"/>
	</bean>
	</beans>
	
	<!-- oracle -->
	<beans profile="oracle">
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${Globals.oracle.DriverClassName}"/>
		<property name="url" value="${Globals.oracle.Url}" />
		<property name="username" value="${Globals.oracle.UserName}"/>
		<property name="password" value="#{egovEnvCryptoService.getPassword()}"/>
	</bean>
	</beans>

	<!-- altibase -->
	<beans profile="altibase">
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${Globals.altibase.DriverClassName}"/>
		<property name="url" value="${Globals.altibase.Url}" />
		<property name="username" value="${Globals.altibase.UserName}"/>
		<property name="password" value="#{egovEnvCryptoService.getPassword()}"/>
		
		<property name="validationQuery" value="select 1"/>
        <property name="testWhileIdle" value="true"/>
	</bean>
	</beans>

	<!-- tibero -->
	<beans profile="tibero">
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${Globals.tibero.DriverClassName}"/>
		<property name="url" value="${Globals.tibero.Url}" />
		<property name="username" value="${Globals.tibero.UserName}"/>
		<property name="password" value="#{egovEnvCryptoService.getPassword()}"/>
	</bean>
	</beans>

    <!-- cubrid -->
    <beans profile="cubrid">
    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${Globals.cubrid.DriverClassName}"/>
        <property name="url" value="${Globals.cubrid.Url}" />
        <property name="username" value="${Globals.cubrid.UserName}"/>
        <property name="password" value="#{egovEnvCryptoService.getPassword()}"/>
        
        <property name="validationQuery" value="select 1"/>
        <property name="testOnBorrow" value="false"/>
    </bean>
    </beans>
    <!--
    	CUBRID 가이드 참조
    	https://www.cubrid.com/tutorial/3794188
    	* DBCP를 통해 최초 connection 시 해당 connection이 유효한지 체크하는 isValid를 호출
     -->

	<!-- MariaDB -->
	<beans profile="maria">
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${Globals.maria.DriverClassName}"/>
		<property name="url" value="${Globals.maria.Url}" />
		<property name="username" value="${Globals.maria.UserName}"/>
		<property name="password" value="#{egovEnvCryptoService.getPassword()}"/>
	</bean>
	</beans>

	<!-- PostresSQL -->
	<beans profile="postgres">  
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${Globals.postgres.DriverClassName}"/>
		<property name="url" value="${Globals.postgres.Url}" />
		<property name="username" value="${Globals.postgres.UserName}"/>
		<property name="password" value="#{egovEnvCryptoService.getPassword()}"/>
	</bean>
	</beans>
	
	<!-- GOLDILOCKS -->
	<beans profile="goldilocks">  
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${Globals.goldilocks.DriverClassName}"/>
		<property name="url" value="${Globals.goldilocks.Url}" />
		<property name="username" value="${Globals.goldilocks.UserName}"/>
		<property name="password" value="#{egovEnvCryptoService.getPassword()}"/>
	</bean>
	</beans>

    <!-- DB Pool이 생성이 되더라고 특정 시간 호출되지 않으면 DBMS 설정에 따라 연결을 끊어질 때
		이 경우 DBCP를 사용하셨다면.. 다음과 같은 설정을 추가하시면 연결을 유지시켜 줍니다. -->
	<!--
	<property name="validationQuery" value="select 1 from dual" />
	<property name="testWhileIdle" value="true" />
	<property name="timeBetweenEvictionRunsMillis" value="60000" /> -->  <!-- 1분 -->

	<!-- DBCP가 아닌 WAS의 DataSource를 사용하시는 경우도 WAS별로 동일한 설정을 하실 수 있습니다.
		(WAS별 구체적인 설정은 WAS document 확인) -->
</beans>
```

수정 후 `context-datasource.xml` : 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans 
	xmlns="http://www.springframework.org/schema/beans" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:util="http://www.springframework.org/schema/util" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd 
	 						http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

	<bean id="egov.propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath:/egovframework/egovProps/globals.properties</value>
                <!-- <value>file:/product/jeus/egovProps/globals.properties</value> -->
            </list>
        </property>
    </bean>
	
	<!-- DataSource -->
	<alias name="dataSource" alias="egov.dataSource" />
	<!-- PostresSQL -->
	<beans profile="postgres">  
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="${Globals.postgres.DriverClassName}"/>
		<property name="url" value="${Globals.postgres.Url}" />
		<property name="username" value="${Globals.postgres.UserName}"/>
		<property name="password" value="#{egovEnvCryptoService.getPassword()}"/>
	</bean>
	</beans>

    <!-- DB Pool이 생성이 되더라고 특정 시간 호출되지 않으면 DBMS 설정에 따라 연결을 끊어질 때
		이 경우 DBCP를 사용하셨다면.. 다음과 같은 설정을 추가하시면 연결을 유지시켜 줍니다. -->
	<!--
	<property name="validationQuery" value="select 1 from dual" />
	<property name="testWhileIdle" value="true" />
	<property name="timeBetweenEvictionRunsMillis" value="60000" /> -->  <!-- 1분 -->

	<!-- DBCP가 아닌 WAS의 DataSource를 사용하시는 경우도 WAS별로 동일한 설정을 하실 수 있습니다.
		(WAS별 구체적인 설정은 WAS document 확인) -->
</beans>

```

이로서 database 생성, 연결 설정까지 모두 마쳤다.

마지막으로, **Maven Update project**를 실행하고, **Maven Install**을 진행하여 필요한 dependency를 설치하고 빌드한다.

그후 본 프로젝트를 Tomcat server [(Tomcat 설치 튜토리얼 참고)](https://hyunwoo1123.github.io/posts/egov-install)로 실행하면, 다음과 같은 화면이 출력된다!

![공통 컴포넌트 튜토리얼 이미지](https://hyunwoo1123.github.io/assets/img/egov/egov_components_result.png)
