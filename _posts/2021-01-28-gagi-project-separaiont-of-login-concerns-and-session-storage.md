---
layout: post
title: "[가지마켓] #4 로그인 관심사 분리 및 세션 처리"
date: 2021-01-28 00:50:59
author: me
categories: Project
tags: gagi
cover: "/assets/instacode.png"
---


> 중고 거래 플랫폼 **가지마켓 프로젝트** 의 진행상황 및 이슈를 공유합니다.<br/>
> 프로젝트 저장소는 [이곳](https://github.com/GagiMarket/gagi)에 확인할 수 있습니다.


## 목차
* 문제점 인식
* 문제점 분석
* 해결 방법 찾기
* 문제점 개선하기

## 문제점 인식
현재 로그인 기능의 경우 2가지 문제를 가지고 있습니다.

#### 세션 정보가 초기화되는 문제
첫 번째, 세션 저장소를 **내장 톰캣의 메모리에 저장하는 방식**으로 구현하여 애플리케이션을 재실행하게 되면 로그인이 풀리는 문제가 있습니다.

이러한 문제가 발생한 이유는 SpringBoot의 경우 자체 내장 톰캠을 가지고 있는 구조인데, 애플리케이션을 재 실행 하게되면 내장 톰캣 또한 재 실행되어 내장 톰캣의 메모리가 초기화 되기 때문입니다.

이러한 상태로 서버에 배포하게 되면 배포할 때마다 톰캣이 재시작되고 세션 정보를 유지할 수 없는 문제가 있습니다.

#### 다중 서버 운영시 세션 동기화 설정 문제
두 번째, 만약 2대 이상의 서버에서 서비스하고 있다면 **톰캣마다 세션 동기화 설정**을 해줘야 하는 문제가 있습니다.

## 문제점 분석
위와 같은 문제를 해결하기 위해 각 **세션 저장소**에 대한 대처 방법 있습니다.

#### 1.톰캣 메모리를 세션 저장소로 사용
현재 선택한 방법과 동일한 방식입니다. 일반적으로 별다른 설정을 하지 않았다면 선택되는 방식입니다.

이 방법은 선택한다면, 톰캣(WAS)에 세션이 저장되기 때문에 2대 이상의 WAS가 구동되는 환경에서는 톰캣들 간의 세션 공유를 위한 추가 설정이 필요합니다.

#### 2.DB를 세션 저장소로 사용
세션 정보를 DB에 저장하는 방식은 여러 WAS 간의 공용 세션을 사용할 수 있는 가장 쉬운 방법입니다.

많은 설정이 필요 없지만, 결국 로그인 요청마다 DB IO가 발생하고 성능상 이슈가 발생할 수 있는 단점이 있습니다.

일반적으로 로그인 요청이 많이 없는 백오피스, 사내 시스템 용도에서 사용합니다.

#### 3.메모리 DB를 세션 저장소로 사용
메모리 DB에는 Redis, Memcached 와 같은 것이 있습니다. 해당 방법의 경우 B2C 서비스에서 가장 많이 사용하는 방식입니다.

그러나, 실제 서비스로 사용하기 위해서 Embedded Redis와 같은 방식이 아닌 외부 메모리 서버가 필요합니다.

그리고 AWS에서 서비스를 배포하고 운영하고 메모리 DB의 사용하는 경우, 서비스(엘라스틱 캐시)에 별도 사용료를 지불해야 하기 때문에 비용 부담이 있습니다.

따라서, 톰캣 메모리 세션 저장소 방식에서 **DB, 메모리 DB로 유연하게 변경이 가능하게 로그인 기능의 역할과 구현을 분리하여 설계**하였습니다. 

## 해결 방법 찾기
#### 회원 로그인 구조 파악하기
현재 회원 로그인의 클래스 간의 협력관계는 다음과 같습니다.

<a href="{{ site.2021_project_img }}/project-gagi-member-login-relation.png" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2021_project_img }}/project-gagi-member-login-relation.png" title="Check out the image">
</a>

로그인 서비스 역할을 담당하는 클래스는 위와 같이 세션 로그인 이라는 하나의 구현체를 가지고 있음을 볼 수 있습니다.

그리고 아래는 로그인 클래스 다이어그램입니다.

<a href="{{ site.2021_project_img }}/project-gagi-member-login-implementation-change.png" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2021_project_img }}/project-gagi-member-login-implementation-change.png" title="Check out the image">
</a>

왼쪽의 로그인의 의존관계 형태에서 오른쪽 형태와 같이 **클라이언트가 바라보는 로그인 서비스 역할을 담당하는 클래스의 변경 없이 구현체만을 유연하게 변경 가능하도록** 하는 것이 목표 입니다.

## 문제점 개선하기
현재 로그인 서비스를 의존하는 클래스는 `MemberApiController` 입니다.

```java
@RestController
@RequestMapping(MEMBER_API_URI)
public class MemberApiController {
    public static final String MEMBER_API_URI = "/api/v1.0/members";

    private final MemberService memberService;
    private final LoginService loginService;

    public MemberApiController(MemberService memberService, LoginService loginService) {
        this.memberService = memberService;
        this.loginService = loginService;
    }
    ...(이하생략)
```

위와 같이 로그인 로직의 경우 여러 Controller에서 사용될 수 있기 때문에 아래와 같이 `LoginService`로 로그인 역할을 담당하는 인터페이스를 구현하였습니다.

```java
package com.gagi.market.member.service;

import com.gagi.market.member.api.dto.SessionMember;

public interface LoginService {
    SessionMember login(String memberEmail, String memberPw);
    void logout();
}
```

그리고 `LoginService`의 구현체는 **내장 톰캣 메모리에 세션을 저장하는 방식**으로 구현한 `SessionLoginService` 구현체를 만들었습니다.

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

이와 같이 역할과 구현을 분리해서 자유롭게 구현 객체를 조립할 수 있게 설계했습니다. 따라서 추후 새로운 세션 저장소(DB, 메모리DB 등)로 변경될 경우에 유연하게 변경이 가능합니다.

## References
* 스프링 부트와 AWS로 혼자 구현하는 웹 서비스
* 인프런 - 스프링 핵심 원리 기본편