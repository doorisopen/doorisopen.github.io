---
layout: post
title: "[가지마켓] #5 회원 도메인 구현과 리팩토링"
date: 2021-01-29 00:00:59
author: me
categories: Project
tags: gagi
cover: "/assets/instacode.png"
---


> 중고 거래 플랫폼 **가지마켓 프로젝트** 의 진행상황 및 이슈를 공유합니다.<br/>
> 프로젝트 저장소는 [이곳](https://github.com/GagiMarket/gagi)에 확인할 수 있습니다.

## 목차
* 회원(Member) 도메인
  + 로그인 기능
* 로그인 비즈니스 로직 추상화
* CORS Global Config 설정
* 인증된 사용자 정보를 저장하는 Dto 사용한 이유
* 세션값 가져오는 중복 코드 어노테이션 기반으로 개선하기

## 회원(Member) 도메인
회원(Member) 도메인은 다음과 같은 정보를 포함하고 있습니다.

|Name|Field Name|Type|
|:---:|:---:|:---:|
|이메일|memberEmail|String|
|비밀번호|memberPw|String|
|전화번호|memberPhoneNumber|String|
|주소|memberAddress|String|
|등록일자|createdDate|LocalDateTime|
|수정일자|modifiedDate|LocalDateTime|

**회원 등록, 조회(단건), 수정, 삭제** 기능들을 제공합니다.

|HTTP Method|URI|Description|
|:---:|:---:|:---:|
|POST|/api/v1.0/members|회원을 등록한다.|
|GET|/api/v1.0/members/{memberEmail}|{memberEmail} 회원을 조회한다.|
|PUT|/api/v1.0/members/{memberEmail}|{memberEmail} 회원을 수정한다.|
|DELETE|/api/v1.0/members/{memberEmail}|{memberEmail} 회원을 삭제한다.|

#### 로그인 기능
로그인 기능은 **로그인, 이메일 중복 확인, 로그아웃** 기능이 있습니다.

|HTTP Method|URI|Description|
|:---:|:---:|:---:|
|POST|/api/v1.0/members/login|로그인 한다.|
|GET|/api/v1.0/members/duplicated/{memberEmail}|{memberEmail}과 중복된 이메일이 있는지 확인 한다.|
|GET|/api/v1.0/members/logout|로그아웃 한다.|


> 회원 API에 대한 자세한 스펙은 [Wiki - 회원(member) API](https://github.com/GagiMarket/gagi/wiki/%ED%9A%8C%EC%9B%90(member)-API)을 확인해 주세요

## 로그인 기능 역할과 구현 분리하기
현재 로그인 기능의 경우 **세션을 내장 톰캣의 메모리에 저장**하는 방식으로 구현하였습니다. 그러나 이는 애플리케이션이 **재실행 됨에 따라 로그인이 풀리는 문제점**을 가지고 있습니다. 

따라서, 추후 다른 **세션 저장소를 도입함에 있어서 구현 객체를 조립할 수 있게 설계**했습니다.(OCP  준수) 따라서 세션 저장소 또한 **유연하게 변경이 가능**합니다.

로그인 기능의 역할과 구현을 분리하여 아래와 같이 **로그인 기능 역할을 담당**하는 `LoginService`를 만들고 

```java
package com.gagi.market.member.service;

import com.gagi.market.member.api.dto.SessionMember;

public interface LoginService {
    SessionMember login(String memberEmail, String memberPw);
    void logout();
}
```

아래와 같이 `LoginService`를 구현하는 `SessionLoginService` 구현체를 만들었습니다.

```java
package com.gagi.market.member.service;

import com.gagi.market.member.api.dto.SessionMember;
import com.gagi.market.member.domain.Member;
import com.gagi.market.member.domain.MemberRepository;
import org.springframework.stereotype.Service;

import javax.servlet.http.HttpSession;
import java.util.Optional;

@Service
public class SessionLoginService implements LoginService {
    private static final String SESSION_MEMBER = "SESSION_MEMBER";
    private final HttpSession httpSession;
    private final MemberRepository memberRepository;

    public SessionLoginService(HttpSession httpSession, MemberRepository memberRepository) {
        this.httpSession = httpSession;
        this.memberRepository = memberRepository;
    }

    @Override
    public SessionMember login(String memberEmail, String memberPw) {
        SessionMember sessionMember = null;
        Optional<Member> findMember = memberRepository.findMemberByMemberEmailAndMemberPw(memberEmail, memberPw);
        if (findMember.isPresent()) {
            sessionMember = new SessionMember(findMember.get());
            setSessionOfMember(sessionMember);
        }
        return sessionMember;
    }

    private void setSessionOfMember(SessionMember sessionMember) {
        httpSession.setAttribute(SESSION_MEMBER, sessionMember);
    }

    @Override
    public void logout() {
        httpSession.removeAttribute(SESSION_MEMBER);
    }
}
```

> 로그인 기능 역할과 구현 분리에 관한 자세한 이야기는 이곳 [#4 로그인 관심사 분리 및 세션 처리](https://doorisopen.github.io/project/2021/01/28/gagi-project-separaiont-of-login-concerns-and-session-storage.html)을 참고해주세요

## CORS Global Config 설정
기존에 아래와 같이 설정이 필요한 Controller 클래스에 `@CrossOrigin` 어노테이션으로 CORS 설정을 해주었습니다.

```java
@CrossOrigin(origins = "*")
@RestController
@RequestMapping(ITEM_API_URI)
public class ItemApiController {
    ...이하 생략
}
```

그러나 서비스가 커짐에 따라 중복 코드가 발생하기 때문에 `com.gagi.market.config` 경로에 `WebConfig` 클래스 파일을 생성해서 Global 설정을 해주었습니다.

```java
package com.gagi.market.config;

import com.gagi.market.config.auth.LoginMemberArgumentResolver;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.List;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    ...(생략)
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .maxAge(3600);
    }
}
```

## 인증된 사용자 정보를 저장하는 Dto 사용한 이유
`SessionMember` 의 경우 세션 등록하기 위한 Dto 클래스 입니다. Member 클래스가 아니라 Dto를 사용한 이유는 **Member 클래스의 경우 엔티티 클래스** 이기 때문입니다. 

엔티티 클래스에는 언제 다른 엔티티와 관계가 형성될지 모릅니다. 예를 들어 `@OneToMany`, `@ManyToOne` 등 자식 엔티티를 갖고 있다면 직렬화 대상에 자식들 까지 포함되니 **성능 이슈, 부수 효과**가 발생할 확률이 높습니다.

따라서, 아래와 같은 직렬화 기능을 가진 세션 Dto를 생성하여 사용하였습니다. 추가로 운영 및 유지보수 측면에서 도움이 됩니다.

```java
package com.gagi.market.member.api.dto;

import com.gagi.market.member.domain.Member;
import lombok.Getter;

import java.io.Serializable;

@Getter
public class SessionMember implements Serializable {
    private String memberEmail;
    private String memberPhoneNumber;
    private String memberAddress;

    public SessionMember(Member member) {
        this.memberEmail = member.getMemberEmail();
        this.memberPhoneNumber = member.getMemberPhoneNumber();
        this.memberAddress = member.getMemberAddress();
    }
}
```

## 세션값 가져오는 중복 코드 어노테이션 기반으로 개선하기
아래와 같이 세션값을 가져오는 **중복 코드를 어노테이션 기반으로 개선**해보겠습니다.

```java
SessionMember sessionMember = (SessionMember) httpSession.getAttribute("SESSION_MEMBER");
```

먼저 `config.auth` 경로에 아래와 같이 **@LoginMember 어노테이션**을 생성합니다.

```java
package com.gagi.market.config.auth;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginMember {
}
```

그리고 해당 어노테이션을 특정 조건을 만족할때 사용 가능하게 하기 위한 설정을 하였습니다.

우선 같은 위치(auth 패키지)에 `HandlerMethodArgumentResolver` 인터페이스의 구현체 클래스를 `LoginMemberArgumentResolver` 이름으로 생성합니다.

```java
package com.gagi.market.config.auth;

import com.gagi.market.member.api.dto.SessionMember;
import org.springframework.core.MethodParameter;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;

import javax.servlet.http.HttpSession;

@Component
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {
    private static final String SESSION_MEMBER = "SESSION_MEMBER";
    private final HttpSession httpSession;

    public LoginMemberArgumentResolver(HttpSession httpSession) {
        this.httpSession = httpSession;
    }

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return (checkLoginMemberAnnotation(parameter) && checkSessionMemberClass(parameter));
    }

    private boolean checkLoginMemberAnnotation(MethodParameter parameter) {
        return parameter.getParameterAnnotation(LoginMember.class) != null;
    }

    private boolean checkSessionMemberClass(MethodParameter parameter) {
        return SessionMember.class.equals(parameter.getParameterType());
    }

    @Override
    public Object resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute(SESSION_MEMBER);
    }
}
```

위의 코드를 간단하게 설명하면

* `checkLoginMemberAnnotation()` 파라미터에 @LoginMember 어노테이션이 붙어있는지 확인
* `checkSessionMemberClass()` 파라미터 클래스 타입이 SessionMember.class 인지 확인

위의 2개의 조건이 참이면 `resolveArgument()` 에서 세션 객체를 가져와 반환합니다.

마지막으로 `LoginMemberArgumentResolver` 클래스가 스프링에서 인식될 수 있도록 WebMvcConfigurer에 추가하겠습니다. 해당 인터페이스의 구현 클래스는 위의 CORS 설정할때 `WebCongfig` 라는 이름으로 생성하였습니다. 다음 내용을 추가했습니다.

```java
package com.gagi.market.config;

import com.gagi.market.config.auth.LoginMemberArgumentResolver;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.List;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final LoginMemberArgumentResolver loginMemberArgumentResolver;

    public WebConfig(LoginMemberArgumentResolver loginMemberArgumentResolver) {
        this.loginMemberArgumentResolver = loginMemberArgumentResolver;
    }

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(loginMemberArgumentResolver);
    }

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .maxAge(3600);
    }
}
```

## References
* 스프링 부트와 AWS로 혼자 구현하는 웹 서비스(p187,195)