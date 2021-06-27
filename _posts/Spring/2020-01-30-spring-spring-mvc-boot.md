---
layout: post
title:  "SpringFramework & SpringMVC & SpringBoot"
date:   2020-01-30 00:56:40
author: me
categories: Spring
tags: Spring Framework SpringMVC SpringBoot
cover:  "/assets/images/Spring/springcover.png"
---

# Spring Framework
__스프링 프레임워크__ 의 가장 중요한 특징은 __의존성 주입(Dependency Injection)__ 이다. 모든 스프링 모듈들의 핵심에는 __의존성 주입__ 이나 __IOC(Inversion of Control)__ 가 있다.

이것이 중요한 이유는 DI나 IOC를 적절히 사용하면 우리는 느슨하게 결합된 애플리케이션들을 개발할 수 있기 때문이다. 또한 느슨하게 결합된 애플리케이션들은 단위테스트를 하기가 쉽다.

간단한 예시...
## 의존성 주입이 없는 예제
아래의 예제를 보면, WelcomeController 는 welcome 메시지를 얻기위해 WelcomeService 에 의존적이다. 그러면 WelcomeService 인스턴스를 얻기 위해서는 어떤 작업을 진행할까?
```
WelcomeService service = new WelcomeService();
```

그것은 WelcomeService 의 인스턴스를 생성하는 것이며, 그것은 그것들이 강하게 결합된다는 것을 의미한다. 예를들어 우리가 WelcomeController 의 단위테스트에서 WelcomeService 에 대한 목(mock)을 생성한다면, 우리는 어떻게 WelcomeController 가 목을 사용하도록 할 수 있을까? 결코 쉽지 않다.

## 의존성 주입을 이용한 예제
의존성 주입을 이용하면 훨씬 더 단순해 보인다. 우리는 단지 2개의 간단한 애너테이션인 @Component, @Autowired를 사용한다.

아래의 예제에서 스프링 프레임워크는 WelcomeService 인스턴스를 생성하고 WelcomeController 내에 @autowired 가 선언된
곳에 주입한다.

단위테스트시 우리는 스프링 프레임워크에게 WelcomeService 의 목을 WelcomeController 안으로 알아서 주입해 달라고 요청할 수 있다. (스프링 부트는 @MockBean 을 사용해서 이러한 작업을 쉽게 할 수 있도록 해줄 수 있다.)

```
@Component
public class WelcomeService {
    ...
}

@RestController
public class WelcomeController {
    @Autowired
    private WelcomeService service;
    @RequestMapping("/welcome")
    public String welcome() {
        return service.retrieveWelcomeMessage();
    }
}
```

## 그밖에 스프링 프레임워크가 해결하는 것은 무엇이 있을까?
### 1. Duplication & Plumbing Code
스프링 프레임워크는 의존성 주입만 할까? 그렇지 않다 다음과 같은 많은 스프링 모듈들을 이용해서 의존성 주입의 핵심을 구성한다.
* Spring JDBC
* Spring MVC
* Spring AOP
* Spring ORM
* Spring JMS(Java Message Service)
* Spring Test

Spring JMS와 Spring JDBC를 생각해 보자. 이 모듈들이 어떤 새로운 기능을 제공하는 걸까? __No!!__
우리는 J2EE 나 Java EE 를 이용해서 이 모듈들이 제공하는 모든 기능들을 사용할 수 있다. 그렇다면 이 모듈들이 제공하는 것은 무엇일까? 이 모듈들은 단순한 추상(abstraction) 을 제공하는데 이 추상들의 목적은 다음과 같다.

* 반복 코드나 중복코드를 줄임
* 디커플링(Coupling: 결합도)을 높이고 단위 테스트성을 증가시킨다.

예를들어 과거의 JDBC 나 JMS 에 비해서 JDBCTemplate 나 JMSTemplate 를 사용하면 훨씬 적은 코드를 작성해도 된다.

### 2. 다른 프레임워크들과의 좋은 통합
스프링 프레임워크가 하는 모든 것은 훌륭한 솔루션을 제공하는 프레임워크들을 훌륭하게 통합해 주는 일이다.

* Hibernate for ORM
* iBatis for Object Mapping
* JUnit and Mockito for Unit Testing

# Spring MVC
### Spring MVC 프레임워크가 해결하는 핵심 문제는 무엇일까?
Spring MVC 프레임워크는 디커플된 웹 애플리케이션 개발 방법을 제공한다.

Dispatcher Servlet, ModelAndView, View Resolver 과 같은 단순개념을 이용해서 웹 애플리케이션 개발을 쉽게 할 수 있도록 해준다.

# Spring Boot
스프링 기반 애플리케이션등은 많은 환경설정을 포함한다.

우리가 Spring MVC 를 사용할 때, 우리는 Component scan, Dispatcher Servlet, View Resolver, Web jar 들(정적 컨텐츠를 제공하기 위한) 를 설정해야 한다.

​__스프링 부트__ 는 클래스패스상에 사용가능한 프레임워크와 이미있는 환경설정을 바라본다. 이것들을 기반으로 스프링 부트는 애플리케이션을 이 프레임워크들과 함께 구성하는데 필요한 기본 환경설정을 제공한다. 이것을 "Auto Configuration" 이라고 부른다.

### Spring Boot Starter 프로젝트 옵션들
Spring Boot Starter Web 에서 살펴보았듯이 스타터 프로젝트들은 특정 애플리케이션 개발을 빠르게 시작할 수 있도록 도와 준다.

* __spring-boot-starter-web-services:__ SOAP Web Services
* __spring-boot-starter-web:__ Web and RESTful applications
* __spring-boot-starter-test:__ Unit testing and Integration Testing
* __spring-boot-starter-jdbc:__ Traditional JDBC
* __spring-boot-starter-hateoas:__ Add HATEOAS features to your services
* __spring-boot-starter-security:__ Authentication and Authorization using Spring Security
* __spring-boot-starter-data-jpa:__ Spring Data JPA with Hibernate
* __spring-boot-starter-cache:__ Enabling Spring Framework’s caching support
* __spring-boot-starter-data-rest:__ Expose Simple REST Services using Spring Data REST

### Spring Boot 의 다른 목표
개발이나 유지보수에 도움을 주는 몇가지 스타터들이 있는데 다음과 같다.

* __spring-boot-starter-actuator:__ 여러분의 애플리케이션 추적하거나 모니터링하는 등의 기능을 사용할 수 있도록 한다. 
* __spring-boot-starter-undertow, spring-boot-starter-jetty, spring-boot-starter-tomcat:__ 내장된 서블릿 컨테이너를 선택할 수 있도록 해준다.
* __spring-boot-starter-logging:__ logback 을 사용해서 로깅할 수 있도록 해준다.
* __spring-boot-starter-log4j2:__ Log4j2 를 사용해서 로깅할 수 있도록 해준다.
스프링부트는 빠른시간에 애플리케이션이 제품이 될 수 있도록 하는 것을 목표로 한다.
* __Actuator:__ 애플리케이션을 고수준에서 모니터링하고 추적 할 수 있도록 해준다.
* __Embedded Server Integrations:__ 서버가 애플리케이션에 통합되기 때문에 우리는 서버에 설치되는 별도의 애플리케이션 서버를 가질 필요가 없다.
* Default Error Handling

# 결론
__Spring MVC__ 는 말 그대로 MVC 구현을 할 수 있도록 지원해주는 스프링 프레임워크 이다. 

__Spring Boot__ 는 Spring에서 제공하는 많은 라이브러리들을 기본 설정 값으로 __자동__ 으로 설정할 수 있도록 해준다. 따라서, Spring Boot는 Spring MVC를 편하게 사용할 수 있도록 해준다는 것 이다.

또한, __Spring Boot__ 는 SpringMVC, Jackson, Validation 등 다양한 종속성들을 각각 패키지해 놓은 것을 함께 가져오기 때문에 개발자들은 이것들의 호환버전에 대해 걱정할 필요가 없다. 

> _Spring MVC와 Spring Boot는 서로의 역할이 전혀 다르기 떄문에 비교 대상이 될 수 없다..._

# 참고문서
* [http://blog.naver.com/PostView.nhn?blogId=sthwin&logNo=221271008423&parentCategoryNo=&categoryNo=50&viewDate=&isShowPopularPosts=true&from=search](http://blog.naver.com/PostView.nhn?blogId=sthwin&logNo=221271008423&parentCategoryNo=&categoryNo=50&viewDate=&isShowPopularPosts=true&from=search)