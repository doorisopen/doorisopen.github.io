---
layout: post
title:  "[Spring Boot] Chap 1.스프링 부트 시작하기"
date:   2020-02-24 14:26:40
author: me
categories: Spring
tags: Spring Framework Boot
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

[스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)](https://jojoldu.tistory.com/463) 책에서 공부한 내용을 정리한 게시글입니다.

해당 시리즈의 소스코드는 [이곳](https://github.com/doorisopen/freelec-springboot2-webservice)에서 확인할 수 있습니다. 

<br />

### 목차
>> 1. 스프링 부트와 gradle
>> 2. mavenCentral, jcenter 비교 
>> 3. intelliJ와 github



<br />
<br />


<hr />

# 스프링 부트와 gradle

* __build.gradle 초기상태__

```
plugins: {
    'java'
}

group 'com.doop.book'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

위의 내용은 gradle 프로젝트를 생성하고 나서 보여지는 `build.gradle`파일의 내용이다. 자바 개발에 가장 기초적인 설정만 되어있는 상태다. 

이 프로젝트에서는 `스프링 이니셜라이저`를 통해서 진행하지 않는다. 이유는 build.gradle의 코드가 무슨 역할을 하는지, 의존성 추가가 필요할때 어떻게 해야 할지 공부하기 위해서이다.


* __build.gradle 수정__

```
buildscript {
    ext { // (1)
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories { // (3)
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}
// (2)
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group 'com.doop.book'
version '1.0-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies { // (4)
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

변경된 코드를 하나하나 설명 하면..

* (1)`ext`라는 키워드는 build.gradle에서 사용하는 __전역변수를 설정__ 하겠다는 의미인데, 여기서는 `springBootVersion 전역변수`를 생성하고 그 값을 `'2.1.7.RELEASE'`로 하겠다는 의미이다.
  + 즉, `spring-boot-gradle-plugin`라는 스프링 부트 그레이들 플러그인이 2.1.7.RELEASE를 의존성으로 받겠다는 의미이다.

* (2)`apply plugin: 'java'`포함 4개의 플러그인은 자바와 스프링 부트를 사용하기 위해서는 필수 플러그인들이기 때문에 항상 추가한다.
  + 특히 `'io.spring.dependency-management'`플러그인은 스트링 부트의 의존성을 관리해 주는 플러그인이라 꼭 추가한다.

* (3)`repositories`는 각종 의존성(라이브러리)들을 어떤 원격 저장소에서 받을지를 정한다.
  + 기본적으로 __mavenCentral__ 을 많이 사용함, 최근에는 __라이브러리 업로드 난이도__ 때문에 __jcenter__ 도 많이 사용한다.

* (4) `dependencies`는 프로젝트 개발에 필요한 의존성들을 선언하는 곳이다.
  + __Tip.__ 인텔리제이는 메이븐 저장소의 데이터를 인덱싱해서 관리하기 때문에 의존성 자동완성이 가능하다.
  + __complie 메소드__ 안에 라이브러리의 이름의 앞부분만 추가한 뒤 __자동완성(Ctrl+Space)__ 을 사용하면 라이브러리 목록을 확인 할 수 있다. 
  + 단, 특정 버전을 명시하면 안된다. 버전을 명시하지 않아야만 맨 위에 작성한 __org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}__ 의 버전을 따라가게 된다.

코드 설정이 끝이나면 오른쪽 하단에 Event Log에서 __build.gradle에 변경이 있으니 반영하라__ 라는
알람이 나온다 __Import Changes__ 는 1회 변경을 허용하는 것이고, __Enable Auto-Import__ 는 수정될때 마다 자동으로 반영이 된다.

# mavenCentral와 jcenter
### mavenCentral
`mavenCentral`은 이전부터 많이 사용하는 저장소지만, 본인이 만든 라이브러리를 업로드하기 위해서는 __많은 과정과 설정__ 이 필요하다.

### jcenter
`jcenter`이 최근에 나오면서 많은 과정과 설정의 문제점을 개선하여 __라이브러리 업로드__ 를 간단하게 하였다. 

더 나아가 `jcenter`에 라이브러리를 업로드하면 `mavenCentral`에도 업로드될 수 있도록 자동화를 할 수 있다.


# intelliJ와 github 
intelliJ에서 생성한 프로젝트를 gitgub와 연동하는 방법을 설명할 것이다. 

1. __윈도우 기준(Ctrl + Shift + A)를 사용해 Action 검색창을 열어 share project on github을 검색__ 한다.

2. __github 로그인을하고 저장소 이름을 입력하여 저장소를 생성__ 한다. 

만약 동기화 과정에서 __커밋 항목으로 추가할 것인지__ 묻는 안내문이 나올 수 있는데 본인이 필요로 한다면 Yes 혹은 No를 선택하면된다. 

그리고 프로젝트의 첫 번째 커밋을 위한 팝업창이 뜨는데 여기서 .idea 디렉토리는 체크를 해제하여 커밋 항목에서 제외 해준다. 이유는 인텔리제이에서 프로젝트 실행시 자동으로 생성되는 파일들 이기 때문이다.

추가로 __gitignore에 .idea와 .gradle를 추가해 주면 커밋 항목에서 자동으로 제외__ 된다.


# IntelliJ 단축키
* __build.gradle 의존성 자동완성__
  + Win: Ctrl + Space
  + Mac: Ctrl + Space
* __Action 검색창__
  + Win: Ctrl + Shift + A
  + Mac: Command + Shift + A
* __github commit__
  + Win: Ctrl + K
  + Mac: Command + K
* __github push__
  + Win: Ctrl + Shift + K
  + Mac: Command + Shift + K

# Related Posts
* [Chap 1.스프링 부트 시작하기 - Now](#)
* [Chap 2.테스트 코드 작성하기](https://doorisopen.github.io/spring/2020/02/24/spring-freelec-springboot-chap2.html)
* [Chap 3.스프링 부트에서 JPA사용하기](https://doorisopen.github.io/spring/2020/02/26/spring-freelec-springboot-chap3.html)
* [Chap 4.Mustache로 화면 구성하기](https://doorisopen.github.io/spring/2020/03/03/spring-freelec-springboot-chap4.html)
* [Chap 5.스프링 시큐리티와 OAuth2.0](https://doorisopen.github.io/spring/2020/03/03/spring-freelec-springboot-chap5.html)
* [Chap 6.AWS EC2 서버 환경 구축하기](https://doorisopen.github.io/spring/2020/03/10/spring-freelec-springboot-chap6.html)
* [Chap 7.AWS RDS 생성하기](https://doorisopen.github.io/spring/2020/03/11/spring-freelec-springboot-chap7.html)
* [Chap 8.EC2 서버에 프로젝트 배포하기](https://doorisopen.github.io/spring/2020/03/12/spring-freelec-springboot-chap8.html)
* [Chap 9.Travis CI 배포 자동화](https://doorisopen.github.io/spring/2020/03/13/spring-freelec-springboot-chap9.html)
* [Chap 10.Nginx를 이용한 무중단 배포](https://doorisopen.github.io/spring/2020/03/18/spring-freelec-springboot-chap10.html)

# References
* 스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)
