---
layout: post
title:  "[Spring] 2 - Maven"
date:   2019-08-14 14:51:59
author: me
categories: Spring
tags: Spring Framework web Maven
cover:  "/assets/images/Spring/springcover.png"
---


<br />
<br />


### 목차
>> 1. __Maven의 기본 정의 이해__
>> 2. __Maven의 프로젝트 관리 방법 이해__
>> 3. __Maven의 빌드 자동화 기능을 이해__


<br />
<br />


<hr />

## Maven
### Maven 이란?
* 프로젝트를 관리하는 도구
* 빌드 자동화 기능과 프로젝트 관리 기능을 제공
* 소프트웨어 빌드를 위한 공통 인터페이스를 제공하는 프레임워크
  - __플러그인 설정을 통해 기능을 위임__

<hr />

### 프로젝트(라이브러리) 관리 란?
* pom.xml 파일을 이용하여 프로젝트 관련된 jar 파일을 다운로드하고 관리
* 프로젝트 산출물을 일관된 구조로 관리


#### 프로젝트 관리 기능
* 일반 개발자 프로젝트 관리 설정들을 메이븐이 미리 정의한 설정들로 대체한다

<br />

* __정형화된 프로젝트 디렉토리 구조 관리 (pom.xml)__
  + Convention over Configuration(CoC) 패러다임을 따름 : 설정보다는 규범

<br />

* __의존성 관리기능__
  + 편리한 라이브러리 관리 기능 (pom.xml, Repositorty)
    - __의존관계(라이브러리) 설정 (pom.xml)__
      - 프로젝트 당 한 개의 pom.xml 파일 관리
      - 최상위 엘리먼트(root element) : project
      - 3개의 필수 필드를 가짐(groupId: artifactId: version)
        - __groupId__ : 프로젝트 조직 고유 도메인
        - __artifactId__ : 프로젝트 명
        - __version__ : 프로젝트 버전
      - 프로젝트 의존관계의 라이브러리 관리 : dependency


    - __프로젝트 빌드__ 에 필요한 라이브러리, 플러그인을 개발자 PC에 자동으로 다운로드
      - __프로젝트 빌드 설정__
        - 프로젝트 기본 정보, 저장소, 프로퍼티, 디렉토리 구조
        - 플러그인(plugins)
        - 골(goals)

  + __Maven Repository__
    - 메이븐 저장소는 프로젝트에 사용되는 프로젝트 jar 파일, 라이브러리 jar 파일들이 위치하며 3가지 타입이 있다
    - __중앙(central) 저장소__ 
      - 오픈 소스 라이브러리, 메이븐 플러그인, 메이븐 아키타입을 관리하는 저장소이다.
      - 중앙 저장소는 __(https://repo.maven.apache.org/maven2), (http://search.maven.org/), (http://mvnrepository.com/)__ 이 있다.
    - __로컬(local) 저장소__
      - 로컬 저장소는 메이븐을 빌드할 때 다운로드하는 라이브러리, 플러그인을 관리하는 저장소이다. 기본 로컬 저장소는 __USER_HOME/.m2/repository__ 디렉토리이다.
    - __원격(remote) 저장소__ 
      - 메이븐 기반으로 프로젝트를 진행하는 경우 프로젝트에 필요한 모든 라이브러리가 메이븐 중앙 저장소에 존재하는 것이 아니다. 이처럼 중앙 저장소에 존재하지 않는 라이브러리를 관리하기 위하여 별도의 메이븐 저장소를 설치해 관리하는 것이 가능하다.
  - __메이븐 의존성 검색 절차(Maven Dependency Search Sequence)__
    1.  __지역(local) 저장소__ 를 검색한다. 찾는 라이브러리가 없을 경우 2단계로 넘어간다.
    2. __중앙(central) 저장소__ 를 검색한다. 찾은 라이브러리는 지역 저장소에 저장한다. 만약 찾는 라이브러리가 없을 경우 3단계로 넘어간다. 만약 원격 저장소가 존재 하지 않을 경우 에러를 발생시키고 종료한다.
    3. __원격(remote) 저장소__ 를 검색한다. 찾은 라이브러리는 지역 저장소에 저장한다. 만약 찾는 라이브러리가 없을 경우 에러를 발생시키고 종료한다.

  - __의존 라이브러리 적용 스코프__
    - 의존 라이브러리를 적용할 수 있는 __시점을 제한__ 할 수 있다. – Scope 설정
<a href="{{ site.spring_img }}/spring_maven_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_maven_2.JPG" title="Check out the image">
</a>

<br />

* __빌드 프로세스를 관리 (pom.xml)__
  + 플러그인 설정을 통해 빌드 자동화 한다
  + 모든 POM은 __Super POM(기본 POM)__ 으로부터 상속받는다
    - Super POM은 기본 설정 정보를 포함한다
    - Super POM을 수정하려면 POM에서 오버라이드한다
<a href="{{ site.spring_img }}/spring_maven_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_maven_1.JPG" title="Check out the image">
</a>


#### 빌드 자동화 란?
* 빌드 작업들을 간단하고 쉽게 그리고 일관성 있게 수행할 수 있는 통합 환경을 제공
* 빌드는 소스 코드 파일을 실행 코드로 변환하여 배포하는 과정

<br />

* __빌드 단계__ (컴파일, 테스트, 패키징, 배포)들을 __빌드 라이프 사이클__ 이라고 한다.
* 각 빌드 단계에서 수행되는 작업을 __골(Goal)__ 이라고 한다.
* 실제 골은 그 단계에 연결된 __플러그인(Plugin)__ 에 의해 실행된다.
<a href="{{ site.spring_img }}/spring_maven_3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_maven_3.JPG" title="Check out the image">
</a>

* __Maven 빌드 라이프사이클(lifecycle)__
  - Maven은 __기본, clean, site__ 3개의 라이프사이클이 있다.
  - 기본 라이프사이클은 여러 단계의 __페이즈(Phase)__ 로 나뉘어져 있으며, 각 페이즈는 의존관계를 가진다. __compile, test, package deploy__ 순서로 진행된다.
  - __clean 라이프사이클__ 은 __clean 페이즈__ 를 이용하여 이전 빌드에서 생성된 모든 파일
  (target 디렉토리)들을 삭제한다.
  - __site 라이프사이클__ 은 __site, site-deploy 페이즈__ 를 이용하여 생성된 문서들을 대상 사이
  트에 배포한다. 
<a href="{{ site.spring_img }}/spring_maven_4.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_maven_4.JPG" title="Check out the image">
</a>

* __Maven Phase & Goal__
  - 빌드 라이프사이클은 하나 이상의 골을 수행하는 __페이지(Phase)__ 들로 구성.
  - 각 페이즈 별로 플러그인이 작업을 수행한다. 이 작업을 __Goal__ 이라고 한다. 
<a href="{{ site.spring_img }}/spring_maven_5.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_maven_5.JPG" title="Check out the image">
</a>

* __Maven Plugin__
- 대부분의 기능들은 __플러그인__ 을 통해서 제공된다.
- 예) maven-compiler-plugin, maven-clean-plugin, maven-surefire-plugin
<a href="{{ site.spring_img }}/spring_maven_6.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_maven_6.JPG" title="Check out the image">
</a>


<hr />


### 관련된 Post


<hr />


### References
* 대학교 웹 프레임워크 수업 정리한 내용 입니다.