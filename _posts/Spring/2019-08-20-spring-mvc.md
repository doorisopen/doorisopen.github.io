---
layout: post
title:  "[Spring] 5 - MVC"
date:   2019-08-19 10:40:59
author: me
categories: Spring
tags: Spring Framework web MVC Model View Controller
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

>> 해당 게시글의 전체 소스코드는 이곳 <a href="https://github.com/doorisopen/SpringSpring/tree/6ff48dd154337dfa402914322d2a74c8a9c21e4d"><strong>Github</strong></a>를 참고해 주세요

<br />

### 목차
>> 1. __스프링 MVC의 기본 개념을 이해__
>> 2. __스프링 MVC 아키텍처를 이해__
>> 3. __스프링 MVC 환경 설정 방식 이해__


<br />
<br />


<hr />

## 스프링 MVC
### 스프링 MVC 설계
* __프론트 컨트롤러 패턴__ 에 기초한 __MVC 프레임워크__
  + 자바 웹 어플리케이션 설계 방식 ( 모델1 , 모델2, 프론트 컨트롤러 )

* 모델, 뷰, 컨트롤러는 각각의 인터페이스가 정의되어 있어 서로 구현에 의존적이지 않아 __약한 결합도로 구성되어 유연하고 확장하기 쉬움__

* 다양한 __서드 파티 라이브러리 연계를 지원__
  + Jackson, Google Gson, Google Protocol Buffers, Apache Tiles, FreeMakers, Rome
  + JasperReports, ApachePOI, Hibernate Validators, Joda-Time, Thymeleaf, HDIV

* __애노테이션의 도입__ 으로 스프링 MVC 보급이 확대됨
  + @Controller, @RequestMapping 등의 선언으로 코딩이 간단해짐

<hr />

### 모델(Model) 1, 2, 프론트 컨트롤러 패턴(스프링 MVC)
* __모델(Model) 1__
  + __JSP 만 사용하여 개발하거나 Java bean을 포함하여 개발하는 방식을 의미__
  + JSP에 뷰와 비즈니스 로직이 혼재되어 복잡도가 높음 – 유지보수 어려움

* __모델(Model) 2__
  + __Model – View – Controller 로 분리__
  + 뷰와 비즈니스 로직의 분리로 유지보수성 및 확장 용이

#### 프론트 컨트롤러 패턴 (스프링 MVC)
* __클라이언트 요청을 별도의 프론트 컨트롤러에 집중하는 방식__
  + 모든 요청의 공통 부분을 별도(프론트 컨트롤로)로 처리
<a href="{{ site.spring_img }}/spring_mvc_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_mvc_1.JPG" title="Check out the image">
</a>

* __프론트 컨트롤러 패턴 실행 프로세스__
  + 프론트 컨트롤러 (= DispatcherServlet)
> 1. DispatcherServlet이 HTTP 요청을 받음
> 2. DispatcherServlet은 서브 Controller로 HTTP 요청 위임
> 3. 서브 Controller는 클라이언트의 요청 처리를 위해 DAO 객체를 호출
> 4. DAO 객체는 리소스를 엑세스하여 Model 객체를 생성 후 요청 결과 리턴
> 5. DispatcherServlet 은 처리 결과에 적합한 뷰에 화면 처리 요청
> 6. 선택된 뷰는 화면에 모델 객체를 가져와 화면 처리
> 7. HTTP 응답

<hr />

### 스프링 MVC의 주요 구성요소
<a href="{{ site.spring_img }}/spring_mvc_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_mvc_2.JPG" title="Check out the image">
</a>

<hr />

### 스프링 스테레오타입 애노테이션 (Spring Stereotype Annotation)
* @Component, @Service, @Controller, @Repository는 스프링이 관리하는 컴포넌트를 나타내는 일반적인 스테레오 타입이다. 
* __MVC 아키텍쳐 기반에서 Service, Presentation, Persistence 계층__ 이 분명한 컴포넌트는 __아래와 같은 스테레오 타입을 사용하여 구체화__
한다.
  + __Service 계층: @Service__
  +  __Presentation 계층: @Controller__
  + __Persistence 계층: @Repository__
* @Service, @Controller, @Repository는 __@Component의 전문화된 타입__ 이다.
<a href="{{ site.spring_img }}/spring_mvc_3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_mvc_3.JPG" title="Check out the image">
</a>

<hr />

### 스프링 MVC 환경(라이브러리) 설정
#### Pom.xml 설정
* __spring-webmvc__ 를 설정하면 __스프링 웹(spring-web)과 기타 스프링 프레임워크 의존 모듈(spring-context)에 대한 의존 관계도 함께 처리__
* 스프링 MVC는 __Bean Validation 구조체(hibernate-validator)를 이용해 입력 값의 유효성을 검증__

```
<org.springframework-version>5.1.3.RELEASE</org.springframework-version>
    ...

<!-- spring-webmvc은 기존에 이미 추가 되어있음 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
<!-- hibernate-validator 이 부분만 추가 해주면됨 -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.2.4.Final</version>
</dependency>
```

#### WAC(Web ApplicationContext) 설정
* 웹 애플리케이션에 사용할 __2가지 애플리케이션 컨텍스트 등록 (web.xml)__ <br />

__1. ContextLoadListner 클래스__
  + 서블릿 컨테이너(Tomcat)에 ContextLoadListner 클래스 등록
  + __서비스 계층 이하의 빈(@Service, @Repository 등)을 등록__ 하기 위한 클래스 
  
__2. DispatcherServlet 클래스__
  + 서블릿 컨테이너(Tomcat)에 프론트 컨트롤러인 DispatcherServlet 클래스 등록
  + __컨트롤러(@Controller 또는 @Component) 빈을 등록__ 하기 위한 클래스

* __Web.xml 내용__
  + 웹 어플리케이션 컨텍스트(WAC) 등록 설정
  + CharacterEncodingFilter 설정
    - 한국어 사용을 위한 문자 인코딩 방식을 설정
    
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee https://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

	<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
	<!-- ContextLoaderListener 설정 파일 -->
    <context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/root-context.xml</param-value>
	</context-param>
	
	<!-- Creates the Spring Container shared by all Servlets and Filters -->
    <!-- ContextLoaderListener 등록 -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
    <!-- /.ContextLoaderListener -->

    <!-- 프론트 컨트롤러(DispathcherServlet)를 서블릿 컨테이너에 등록 -->
	<!-- Processes application requests -->
	<servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
            <!-- 프론트 컨트롤러(DispathcherServlet) 설정 파일 -->
			<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<!-- /.프론트 컨트롤러(DispathcherServlet) -->

	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	
	<!-- UTF-8 인코딩 설정 by doop -->
	<filter>
	  <filter-name>CharacterEncodingFilter</filter-name>
	  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
	  <init-param>
	   <param-name>encoding</param-name>
	   <param-value>UTF-8</param-value>
	  </init-param>
	</filter>
	<filter-mapping>
	  <filter-name>CharacterEncodingFilter</filter-name>
	  <url-pattern>/*</url-pattern>
	</filter-mapping>
    <!-- /.UTF-8 -->

</web-app>
```

#### ContextLoadListner 설정
* @Service 지정된 클래스를 빈으로 등록

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:c="http://www.springframework.org/schema/c"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
	
	<!-- Root Context: defines shared resources visible to all other web components -->
		
	<context:component-scan base-package="org.doorisopen.myspring.Member.Service"></context:component-scan>
	<context:component-scan base-package="org.doorisopen.myspring.Member.Persistence"></context:component-scan>
	
	<context:component-scan base-package="org.doorisopen.myspring.Board.Service"></context:component-scan>
	<context:component-scan base-package="org.doorisopen.myspring.Board.Persistence"></context:component-scan>
	
</beans>
```

#### DispatcherServlet 설정
* __컨트롤러 등록__
  + DispatcherServlet 클래스의 설정 파일인 servlet-context.xml에서 컨트롤러 등록
* __DispatcherServlet 클래스 설정__
  + __<annotation-driven /> 태그__
    - 스프링 MVC를 이용하는데 필요한 컴포넌트 빈 정의가 자동으로 수행
    - __패키지 내부에서 찾은 빈(컨트롤러)과 URI를 맵핑__
  + __<context:component-scan base-package="org.doorisopen.myspring.Board.Controller" /> 태그__
    - base-package 내부의 클래스에서 @Controller 지정된 컨트롤러를 검색하여 빈으로 등록
  + __정적 리소스 파일 설정__
    - __resource 태그 사용__
  + __뷰 리졸브 설정__
    - JSP 뷰 리졸브 설정 - org.springframework.web.servlet.view.InternalResourceViewResolver
    - __뷰 객체 이름 생성을 위한 접두사, 접미사 지정__

* __servlet-context.xml 내용__

```

<annotation-driven />

<!-- 정적 리소스 파일(CSS, 이미지, JS) 엑세스 설정 -->
<resources mapping="/resources/**" location="/resources/" />

<!-- 뷰의 파일 위치 설정 /WEB-INF/views/*.jsp -->
<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <beans:property name="prefix" value="/WEB-INF/views/" />
    <beans:property name="suffix" value=".jsp" />
</beans:bean>

<context:component-scan base-package="org.doorisopen.myspring" />
<context:component-scan base-package="org.doorisopen.myspring.Member.Controller" />
<context:component-scan base-package="org.doorisopen.myspring.Board.Controller" />
    ...(중략)
```

<hr />
<hr />

> 여기까지 __스프링 MVC의 개념__, __환경 설정 방법__ 그리고 각각의 __설정 파일의 의미__ 들을 공부하였다
>> 이어서.. 다음 포스트에서는 __컨트롤러 구현 방식을 이해__ 를 위한 urlController 예제를 만들어 볼꺼다

<hr />
<hr />

### References
* 대학교 웹 프레임워크 수업 정리한 내용 입니다.