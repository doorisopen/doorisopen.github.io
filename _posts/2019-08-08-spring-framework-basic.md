---
layout: post
title:  "[Spring] 0 - 웹프레임워크 기초"
date:   2019-08-08 10:00:00
author: me
categories: Spring
tags: Spring Framework web 
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

### 목차

<strong>

>> 1. 웹 어플리케이션 아키텍처 개념을 이해
>> 2. 스프링 프레임워크 특징을 이해

</strong>


<br />
<br />


<hr />


## 웹 어플리케이션
### 일반적인 웹 시스템
* __정적 컨텐츠(HTML)__ <br>
웹 브라우저가 웹 서버로부터 저장된 HTML을 읽어와 표현한다.
  + __종류 :__ HTML, CSS, Javascript 
* __동적 컨텐츠(JSP)__ <br>
웹 브라우저가 웹 서버에 동적 페이지를 요청하면 이 요청을 어플리케이션 서버가 실행, 처리결과를 브라우저가 HTML 형식으로 받아서 표현한다.

<a href="{{ site.spring_img }}/spring_basic_01.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_basic_01.JPG" title="Check out the image">
</a>

### Spring Framework
* __EJB(Enterprise JavaBean)__
1. 분산 환경에서의 트랜잭션 처리를 위한 콤포넌트
2. JSP와 서블릿은 프리젠테이션 구현
3. EJB은 비즈니스 로직을 구현
4. __J2EE 버전업과 함께 EJB 복잡도는 높아지고 확장성은 떨어짐__

* __Spring Framework 란?__
1. 2003년 Rod Johnson이 개발
2. 자바 플랫폼을 위한 오픈소스 애플리케이션 프레임워크 (사실상의 J2EE 의 표준 프레임워크)
3. __POJO(Plain Old Java Object)__ 지원을 통해 기존 EJB의 문제점인 __복잡성__ 을 줄임
4. POJO는 특정 인터페이스 또는 클래스를 상속받을 필요가 없는 가벼운 객체를 의미
5. 대한민국 전자정부 프레임워크의 기반 기술


<hr />


## 웹 어플리케이션 아키텍처
### 아키텍처가 중요한 이유 

__1. 서버 어플리케이션 구조의 변화__
  + Client-Server 구조에서 Web Server 기반 구조로 진화
  + 요구사항 변경, 잦은 기능 추가 발생, 미래 환경의 빠른 변화에 대응 필요
  + 비즈니스 로직과 UI 로직이 서버에 종속

__2. 서버 어플리케이션 특징에 적합한 구조가 필요__
  + 유연하게 대응할 수 있는 개발 효율성인 구조는 __서버 어플리케이션의 수명을 유지__

<a href="{{ site.spring_img }}/spring_basic_02.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_basic_02.JPG" title="Check out the image">
</a>


### 웹 어플리케이션 아키텍처
* 물리층인 __티어(Tier)__ 와 논리층인 __레이어(Layer)__ 로 구분 한다.

<a href="{{ site.spring_img }}/spring_basic_03.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_basic_03.JPG" title="Check out the image">
</a>

#### 서버 레이어(Layer)
* 단방향 3개 층(Layer)으로 구성된다.
<a href="{{ site.spring_img }}/spring_basic_04.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_basic_04.JPG" title="Check out the image">
</a>

* __컨트롤러(Controller):__ 페이지 화면 전환 또는 동작 제어 
* __서비스(Service):__ 유스케이스로 표현되는 특정 업무 처리, 트랜잭션 기점
* __도메인(Domain):__ 서비스로부터 기능을 실행하는데 사용되는 고객 또는 주문 같은 클래스의 집합. 관련 정보들을 저장

#### 오목형 레이어(Layer)
* 프리젠테이션 층 또는 영속화 메커니즘이 변경되어도 __비즈니스 로직층에는 영향을 최소화__
* 레이어 층 도입과 함께 __결합 부분에 약한 결합 설계 구현(인터페이스 도입)__ 필요
<a href="{{ site.spring_img }}/spring_basic_05.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_basic_05.JPG" title="Check out the image">
</a>

* __프리젠테이션 층__
  + 사용자 인터페이스와 컨트롤러를 제공하는 역학
  + 컨트롤러는 사용자 인터페이스를 통해 사용자의 입력을 받아 적정한 로직을 호출하고, 그 결과를 사용자 인터페이스로 변환하는 작업
  + 사용자 인터페이스는 화면 인터페이스를 의미한다.
  + 다양한 사용자 인터페이스 등장
    - AJAX 비동기통신
    - 리액트(React), Angular 클라이언트 프레임워크
<br />
<br />
  + __모델 1 방식__
    - JSP만 사용하여 개발하거나 Java bean을 포함하여 개발하는 방식을 의미
    - JSP에 뷰와 비즈니스 로직이 혼재되어 복잡도가 높다 - 유지보수 어려움
<a href="{{ site.spring_img }}/spring_basic_06.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_basic_06.JPG" title="Check out the image">
</a>

  + __모델 2 방식__
    - __모델(Model):__ 뷰에 필요한 비즈니스 영역의 로직을 처리
    - __뷰(View):__ 비즈니스 영역에 대한 프리젠테이션 뷰(결과 화면)를 담당
    - __컨트롤러(Controller):__ 사용자의 입력 처리와 화면의 흐름 제어를 담당
<a href="{{ site.spring_img }}/spring_basic_07.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_basic_07.JPG" title="Check out the image">
</a>

<br />

* __비즈니스 로직 층__
  + __유스케이스로 표현되는 특정 업무 처리를 위한 서비스와 도메인으로 구성__
  + 개발, 운영 업무의 경우 웹 애플리케이션의 기능 추가와 변경은 비즈니스 로직 변경
  + __트랜잭션 스크립트 방식__
    - 비즈니스 로직이 적은 일반적인 입출력 애플리케이션 일 경우 사용
    - 비즈니스 로직은 서비스 클래스에 포함
    - __도메인은 가능한 로직을 포함하지 않고 단순한 값만 저장한다__
      - VO(Value Object), DTO(Data Transfer Object)

<br />

  + __도메인 모델__
    - 에릭 에반스가 제안한 도메인 주도 설계 개념
      - 데이터와 애플리케이션을 설계할 때 업무 도메인 별로 분리하여 설계
      - 시스템 요구를 기술하기 위해 도메인 전문가가 도메인 모델 제공
    - 최근 대규모 시스템의 복잡한 비즈니스 로직을 방지하기 위한 대안
    - __도메인 패키지에 도메인 로직을 두는 방식__
  > 스프링 프레임워크는 도메인 모델의 구축과 별도의 큰 시너지 효과는 없다.
<br />
  > 도메인의 생성과 관리는 DI 컨테이너가 아닌 데이터 액세스 층의 구조에 의존한다.
  
<br />

  + __트랜잭션 관리__
    - 트랜잭션의 ACID 특성 중 __원자성__ 과 __독립성__ 을 관리
    - __원자성__ : 트랜잭션 내의 모든 처리는 전부 실행됐거나 아무것도 실행되지 않음
    - __독립성__ : 병행해서 실행되는 트랜잭션 간에는 간섭 받지 않고 독립적임

<br />

  + __트랜잭션 경계__
    - 트랜잭션 경계는 프리젠테이션과 비즈니스 로직 층 사이에 존재하는 것이 일반적
    - 프리젠테이션 층에 공개된 비즈니스 로직 층의 서비스 클래스가 트랜잭션 시작과 끝
      - 프리젠테이션 층에 공개된 비즈니스 로직 층의 서비스 클래스의 메서드가 호출되면 트랜잭션이 시작되고, 서비스 클래스의 메서드가 종료되고 프리젠테이션 층의 클래스로 되돌아갈 때 트랜잭션이 종료
<a href="{{ site.spring_img }}/spring_basic_08.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_basic_08.JPG" title="Check out the image">
</a>

<br />
  + __명시적 트랜잭션__
    - 트랜잭션 시작과 커밋, 롤백과 같은 RDB에 대한 트랜잭션 관리를 소스 코드로 명시
<a href="{{ site.spring_img }}/spring_basic_09.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_basic_09.JPG" title="Check out the image">
</a>

  + __선언적 트랜잭션__
    - 프레임워크에서 제공하는 정의 파일 선언을 통해 트랜잭션 관리
<a href="{{ site.spring_img }}/spring_basic_10.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_basic_10.JPG" title="Check out the image">
</a>

<br />

* __데이터 액세스 층__
  + __RDB 액세스를 비즈니스 로직으로 숨기고, 비즈니스 로직에 필요한 데이터를 테이블 에서 취득해서 오브젝트에 매칭__
    - 이때 오브젝트와 __RDB를__ 매핑하는 것을 __O/R 매핑__ 이라고 함
      - 객체지향 분석설계 단계에서의 엔터티를 테이블로 작성한 형태
  
  <br />

  + __DB 액세스 프레임워크의 종류__
    - __Hybernate:__ 대표적인 ORM(O/R Mapping 프레임워크)
    - __MyBatis__, __스프링 JDBC__ : SQL 문 사용 전제로 한 DB 액세스 프레임워크

  <br />

  + __데이터 액세스 층의 설계 지침__
    - 커넥션 풀을 사용한다.
    - RDB 제품이 바뀌어도 구현에 영향을 미치지 않는다
    - 이용하는 RDB에 의존적인 SQL 문을 기술하지 않는다

  <br />

  + __부품화__
    - 개발 효율성과 유연성 향상을 위해서는 티어 또는 레이어를 통해 부품화
    - 부품이 큰 쪽이 티어, 레이어 그리고 작은 부품은 패키지 또는 컴포넌트
    - 부품화를 위해서는 __인터페이스가 중요__
  + __부품화를 위해서는 2가지 중요점__
    - __연결해야 할 부품 간에 결국 중요한 부품이 인터페이스를 가진다__
    - 절대적 기준은 없다. 성능이 허용하는 한 부품화할 필요가 있는 만큼 부품화한다.


<hr />


## Spring Framework의 주요 특징
* __POJO(Plain Old Java Object) 관리__
  + 특정한 인터페이스를 구현하거나 상속을 받을 필요가 없는 가벼운 객체를 관리

<br /> 

* __IoC(Inversion of Control) 경량 컨테이너(light-weight Container)__
  + 필요에 따라 스프링 프레임워크에서 사용자의 코드를 호출한다.
  + 일반 오브젝트의 생애 주기 관리나 오브젝트 간의 의존관계를 해결하는 아키텍처를 구현

<br />

* __DIxAOP (Dependency Injection, Aspect Oriented Programming ) 지원__
  + DI: 각각의 계층이나 서비스들 간에 의존성이 존재할 경우 프레임워크가 서로 연결시켜줌 ( 변경 용이성, 확장성, 품질관리 용이 )
  + AOP : 트랜잭션이나 로깅, 보안과 같이 여러 모듈에서 공통적으로 사용하는 기능의 경우 해당 기능을 분리하여 관리 ( 프로그램 가독성, 기술 은닉 )

<br />

* __O/R 매핑 프레임워크 지원__
  + 완성도가 높은 데이터베이스 처리 라이브러리와 연결할 수 있는 인터페이스를 제공 ( Mybatis, Hibernate )

<br />

* __스프링 데이터를 통한 다양한 데이터 연동 기능__
  + 그래프 DB, 다큐넌트 DB, NoSQL 등의 데이터 저장소를 엑세스 ( JPA, MongoDB, Neo4j, Redis, Cassandra, Sola 등 )

<br />

* __스프링 배치(Spring Batch)__
  + 대량의 데이터를 일괄 처리, 병행 처리할 수 있는 템플릿 제공

<br />

* __스프링 시큐리티(Spring Security)__
  + 인증(Authentication), 인가(Authorization) 기능 제공
  + 베이직 인증 또는 OAuth(Open Authorization) 개방형 표준 인증 서비스 제공


<hr />


### References
* 대학교 웹 프레임워크 수업 정리한 내용 입니다.
