---
layout: post
title:  "[Spring Boot] Chap 5.스프링 시큐리티와 OAuth2.0"
date:   2020-03-03 16:20:40
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
>> 1. 스프링 시큐리티와 OAuth2 클라이언트
>> 2. 구글 서비스 등록 및 구글 로그인 연동
>> 3. 애너테이션 기반으로 개선하기
>> 4. 세션 저장소로 데이터베이스 사용하기
>> 5. 네이버 로그인 
>> 6. 기존 테스트에 시큐리티 적용하기





<br />
<br />


<hr />


# 스프링 시큐리티와 OAuth2 클라이언트
`스프링 시큐리티`는 막강한 인증(Authentication)과 인가(Authorization)기능을 가진 프레임워크입니다. 사실상 스프링 기반의 애플리케이션에서는 보안을 위한 표준이라고 보면 됩니다. 인터셉터, 필터 기반의 보안 기능을 구현하는 것보다 스프링 시큐리티를 통해 구현하는 것을 적극적으로 권장하고 있습니다.

왜 많은 서비스에서 `소셜 로그인(OAuth2)`을 사용할까?? 만약 로그인을 직접 구현하면 다음을 전부 구현해야 합니다.

* 로그인 시 보안
* 비밀번호 찾기
* 비밀번호 변경
* 회원정보 변경
* 회원가입 시 이메일 혹은 전화번호 인증

OAuth 로그인 구현 시 앞선 목록의 것들을 모두 구글, 네이버 등에 맡기면 되니 __서비스 개발에 집중__ 할 수 있기 때문입니다.

스프링 부트 1.5에서의 OAuth2 연동방법이 2.0에서는 크게 변경되었습니다. 하지만, 인터넷 자료들을 보면 __설정 방법에 크게 차이가 없는 경우를 자주 봅니다.__ 이는 `spring-security-oauth2-autoconfigure` 라이브러리 덕분입니다. 이를 사용할 경우 스프링 부트 2에서도 1.5에서 쓰던 설정을 그대로 사용할 수 있습니다.

또한, 스프링 부트 2에 관련한 다른 인터넷 자료를 참고할때 다음 __2가지__ 를 확인하면 됩니다.

1. spring-security-oauth2-autoconfigure 라이브러리를 사용했는가?
2. application.properties 혹은 application.yml 에 작성된 정보의 차이

* 스프링 부트 1.5 방식에서는 url 주소를 모두 명시해야 하지만, 2.0 방식에서는 __client 인정 정보만 입력__ 하면 됩니다. 
* 1.5버전에서 직접 입력했던 값들은 __2.0버전으로 오면서 모두 enum으로 대체__ 되었습니다.
  + __CommonOAuth2Provider__ 라는 enum이 새롭게 추가되어 구글, 깃허브, 페이스북, 옥타(Okta)의 기본 설정값은 모두 여기서 제공합니다.


# 구글 서비스 등록 및 구글 로그인 연동
구글 로그인을 사용하기 위해서 [구글 클라우드 플랫폼](https://console.cloud.google.com/?hl=ko)에 서비스를 등록해야합니다. 

1. 새 프로젝트 > 만들기
2. 왼쪽 상단 네비 > API 및 서비스 > OAuth 동의 화면 구성
3. 사용자 인증 정보 > 상단에 사용자 인증 정보 만들기 선택 > OAuth 클라이언트 ID
4. __승인된 리디렉션 URI__ 에만 > __http://localhost:8080/login/oauth2/code/google__ 입력 후 생성
5. 클라이언트 ID, 클라이언트 보안 비밀번호 생성 확인

* __승인된 리디렉션 URI__
  + 서비스에서 파라미터로 인증 정보를 주었을 때 인증이 성공하면 구글에서 리다이렉트할 URL입니다.
  + 스프링 부트 2 버전의 시큐리티에서는 기본적으로 {도메인}/login/oauth2/code/{소셜서비스코드}로 리다이렉트 URL을 지원하고 있습니다.
  + 현재는 개발 단계이므로 http://localhost:8080/login/oauth2/code/google로만 등록합니다.
  + AWS 서버에 배포하게 되면 localhost 외에 추가로 주소를 추가해야하며, 이건 이후 단계에서 진행합니다.


구글 서비스 등록이 끝났다면 __src/main/resources/ 디렉토리에 application-oauth.properties 파일을 생성__ 합니다.

* application-oauth.properties

```
spring.security.oauth2.client.registration.google.client-id=클라이언트ID
spring.security.oauth2.client.registration.google.client-secret=클라이언트 보안 비밀
spring.security.oauth2.client.registration.google.scope=profile,email
```

* __scope=profile,email__
  + 많은 예제에서는 이 scope를 별도로 등록하지 않고 있습니다.
  + 기본값이 openid, profile, email이기 때문입니다.
  + 강제로 profile, email를 등록한 이유는 openid라는 scope가 있으면 Open Id Provider로 인식하기 때문입니다.
  + 이렇게 되면 OpenId Provider인 서비스(구글)와 그렇지 않은 서비스(네이버/카카오 등)로 나눠서 각각 OAuth2Service를 만들어야 합니다.
  + 하나의 OAuth2Service로 사용하기 위해 일부러 openid scope를 빼고 등록합니다.

application.properties에 다음 코드를 추가합니다.

* application.properties

```
spring.profiles.include=oauth
```

스프링 부트에서는 properties의 이름을 application-xxx.properties 로 만들면 xxx라는 이름의 profile이 생성되어 이를 통해 관리할 수 있습니다. 즉, profile=xxx라는 식으로 호출하면 __해당 properties의 설정들을 가져올__ 수 있습니다. 

추가로 클라이언트ID와 PW가 깃 허브에 올라가지 않게 __application-oauth.properties__ 를 .gitignore에 추가해줍니다.


## 구글 로그인 연동하기
1. __User 클래스__: 사용자 정보를 담당할 도메인 (domain 아래 user 패키기 생성 후 User 클래스 생성)
2. __Role Enum 클래스__: 각 사용자의 권한을 관리할 Enum 클래스
3. __UserRepository 클래스__: User의 CRUD를 책임질 클래스
4. __SecurityConfig 클래스__: OAuth 라이브러리를 이용한 소셜 로그인 설정 코드 작성(springboot 패키지 아래 config.auth 패키지 생성 후 클래스 생성)
5. __CustomOAth2UserService 클래스__: 구글 로그인 이후 가져온 사용자의 정보(email, name, picture 등)들을 기반으로 가입 및 정보수정, 세션 저장 등의 기능을 지원합니다.
6. __OAuthAttributes 클래스__: OAuth의 dto클래스 (config.auth.dto 패키지를 만들어 그 안에 클래스 생성합니다)
7. __SessionUser 클래스__: 인증된 사용자 정보만 필요로한다. (dto 패키지에 생성)


build.gradle에 스프링 시큐리티 관련 의존성 추가 합니다.

* build.gradle

```
dependencies {
    ...
    compile('org.springframework.boot:spring-boot-starter-oauth2-client') // chap5 (1)
}
```

* (1) `spring-boot-starter-oauth2-client`
  + 소셜 로그인 등 클라이언트 입장에서 소셜 기능 구현 시 필요한 의존성입니다.
  + spring-boot-starter-oauth2-clientdhk spring-boot-starter-oauth2-jose를 기본으로 관리해줍니다.

### 1. User 클래스

```
package com.doop.book.springboot.domain.user;

import com.doop.book.springboot.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column
    private String picture;

    @Enumerated(EnumType.STRING) // (1)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(String name, String email, String picture, Role role) {
        this.name = name;
        this.email = email;
        this.picture = picture;
        this.role = role;
    }

    public User update(String name, String picture) {
        this.name = name;
        this.picture = picture;

        return this;
    }
    public String getRoleKey(){
        return this.role.getKey();
    }
}
```

* (1) `@Enumerated(EnumType.STRING)`
  + JPA로 데이터베이스로 저장할 때 Enum 값을 어떤 형태로 저장할지를 결정합니다.
  + 기본적으로는 int로 된 숫자가 저장됩니다.
  + 숫자로 저장되면 데이터베이스로 확인할 때 그 값이 무슨 코드를 의미하는지 알 수가없습니다.
  + 그래서 문자열(EnumType.STRING)로 저장될 수 있도록 선언합니다.

### 2. Role Enum 클래스

```
package com.doop.book.springboot.domain.user;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum Role {

    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;
}
```

스프링 시큐리티에서는 __권한 코드에 항상 ROLE_ 이 앞에 있어야만 합니다.__ 그래서 코드별 키 값을 ROLE_GUEST, ROLE_USER 등으로 지정합니다.

### 3. UserRepository 클래스
user 패키지 아래 UserRepository 클래스 생성합니다.

```
package com.doop.book.springboot.domain.user;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email); // (1)
}
```

* (1) `findByEmail`
  + 소셜 로그인으로 반환되는 값 중 email을 통해 이미 생성된 사용자인지 처음 가입하는 사용자인지 판단하기 위한 메소드입니다.

### 4. SecurityConfig 클래스
여기 부터는 OAuth 라이브러리를 이용한 소셜 로그인 설정 코드를 작성하는 부분입니다.

springboot 패키지 아래 config.auth 패키지를 생성하고 __시큐리티 관련 클래스는 모두 이 패키지에 담아질 것__ 입니다.

```
package com.doop.book.springboot.config.auth;

import com.doop.book.springboot.domain.user.Role;
import lombok.RequiredArgsConstructor;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@RequiredArgsConstructor
@EnableWebSecurity // (1)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable().headers().frameOptions().disable() // (2)
                .and()
                    .authorizeRequests() // (3)
                    .antMatchers("/","/css/**","/images/**","/js/**","/h2-console/**").permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name()) // (4)
                    .anyRequest().authenticated() // (5)
                .and()
                    .logout()
                        .logoutSuccessUrl("/") // (6)
                .and()
                    .oauth2Login()// (7)
                        .userInfoEndpoint() // (8)
                            .userService(customOAuth2UserService); // (9)
    }
}
```

* (1) `@EnableWebSecurity`
  + Spring Security 설정들을 활성화시켜 줍니다.
* (2) `.csrf().disable().headers().frameOptions().disable()`
  + h2-console 화면을 사용하기 위해 해당 옵션들을 disable 합니다
* (3) `authorizeRequests`
  + URL별 권한 관리를 설정하는 옵션의 시작점입니다.
  + authorizeRequests가 선업되어야만 antMatchers 옵션을 사용할 수 있습니다.
* (4) `antMatchers`
  + 권한 관리 대상을 지정하는 옵션입니다.
  + URL, HTTP 메소드별로 관리가 가능합니다.
  + "/" 등 지정된 URL 들은 permitAll() 옵션을 통해 전체 열람 권한을 주었습니다.
  + "/api/v1/**" 주소를 가진 API는 USER 권한을 가진 사람만 가능하도록 했습니다.
* (5) `anyRequest`
  + 설정된 값들 이외 나머지 URL들을 나타냅니다.
  + 여기서는 authenticated()을 추가하여 나머지 URL들은 모두 인증된 사용자들에게만 허용하게 합니다.
  + 인증된 사용자 즉, 로그인한 사용자들을 이야기합니다.
* (6) `logout().logoutSuccessUrl("/")`
  + 로그아웃 기능에 대한 여러 설정의 진입점입니다.
  + 로그아웃 성공 시 / 주소로 이동합니다.
* (7) `oauth2Login()`
  + OAuth 2 로그인 기능에 대한 여러 설정의 진입점입니다.
* (8) `userInfoEndpoint()`
  + OAuth 2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당합니다.
* (9) `userService`
  + 소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록합니다.
  + 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고 자 하는 기능을 명시할 수 있습니다.

### 5. CustomOAth2UserService 클래스

```
package com.doop.book.springboot.config.auth;

import com.doop.book.springboot.config.auth.dto.OAuthAttributes;
import com.doop.book.springboot.config.auth.dto.SessionUser;
import com.doop.book.springboot.domain.user.User;
import com.doop.book.springboot.domain.user.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.client.userinfo.DefaultOAuth2UserService;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserRequest;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserService;
import org.springframework.security.oauth2.core.OAuth2AuthenticationException;
import org.springframework.security.oauth2.core.user.DefaultOAuth2User;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Service;

import javax.servlet.http.HttpSession;
import java.util.Collections;

@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest
                .getClientRegistration()
                .getRegistrationId(); // (1)
        String userNameAttributeName = userRequest
                .getClientRegistration()
                .getProviderDetails()
                .getUserInfoEndpoint()
                .getUserNameAttributeName(); // (2)

        OAuthAttributes attributes = OAuthAttributes
                .of(registrationId, userNameAttributeName, oAuth2User.getAttributes()); // (3)

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user)); // (4)

        return new DefaultOAuth2User(
                Collections.singleton(
                        new SimpleGrantedAuthority(user.getRoleKey())
                ),
                attributes.getAttributes(),
                attributes.getNameAttributeKey()
        );
    }
    private User saveOrUpdate(OAuthAttributes attributes) { // (5)
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(),
                        attributes.getPicture()))
                .orElse(attributes.toEntity());
        return userRepository.save(user);
    }
}
```

* (1) `getRegistrationId`
  + 현재 로그인 진행 중인 서비스를 구분하는 코드입니다.
  + 지금은 구글만 사용하는 불필요한 값이지만, 이후 네이버 로그인 연동 시에 네이버로 로그인인지, 구글 로그인인지 구분하기 위해 사용합니다.
* (2) `getUserNameAttributeName`
  + OAuth2 로그인 진행 시 키가 되는 필드값을 이야기합니다. Primary Ket와 같은 의미입니다.
  + 구글의 경우 기본적으로 코드를 지원하지만, 네이버 카카오 등은 기본 지원하지 않습니다. 구글의 기본 코드는 "sub" 입니다.
  + 이후 네이버 로그인과 구글 로그인을 동시 지원할 때 사용됩니다.
* (3) `OAuthAttributes`
  + OAuth2userService를 통해 가져온 OAuth2User의 attribute를 담을 클래스입니다.
  + 이후 네이버 등 다른 소셜 로그인도 이 클래스를 사용합니다.
  + 바로 아래에서 이 클래스의 코드가 나오니 차례로 생성하시면 됩니다.
* (4) `SessionUser`
  + 세션에 사용자 정보를 저장하기 위한 Dto 클래스입니다.
  + __WHY, User 클래스를 쓰지 않고 새로 만들어서 쓰는가?__
* (5) `saveOrUpdate`
  + 구글 사용자 정보가 업데이트 되었을 때를 대비하여 update 기능도 같이 구현하였습니다.
  + 사용자의 이름이나 프로필 사진이 변경되면 User 엔티티에도 반영됩니다.


### 6. OAuthAttributes 클래스
config.auth.dto 패키지를 만들어 해당 패키지에 OAuthAttributes 클래스 생성합니다.

```
package com.doop.book.springboot.config.auth.dto;

import com.doop.book.springboot.domain.user.Role;
import com.doop.book.springboot.domain.user.User;
import lombok.Builder;
import lombok.Getter;

import java.util.Map;

@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes,
                           String nameAttributeKey, String name,
                           String email, String picture) {
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    // (1)
    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        if("naver".equals(registrationId)) {
            return ofNaver("id", attributes);
        }

        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofGoogle(String userNameAttributeName,
                                            Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes) {
        Map<String, Object> response = (Map<String, Object>) attributes.get("response");

        return OAuthAttributes.builder()
                .name((String) response.get("name"))
                .email((String) response.get("email"))
                .picture((String) response.get("profile_image"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    // (2)
    public User toEntity() {
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST)
                .build();
    }
}
```

* (1) `of()`
  + OAuth2User에서 반환하는 사용자 정보는 Map이기 때문에 값 하나하나를 변환해야만 합니다.
* (2) `toEntity()`
  + User 엔티티를 생성합니다.
  + OAuthAttribute에서 엔티티를 생성하는 시점은 처음 가입할 때입니다.
  + 가입할 때의 기본 권한을 GUEST로 주기 위해서 role 빌더값에는 Role.GUEST를 사용합니다.

### 7. SessionUser 클래스
OAuthAttributes 클래스 생성이 끝났으면 같은 패키지에 __SessionUser 클래스__ 를 생성합니다.

```
package com.doop.book.springboot.config.auth.dto;

import com.doop.book.springboot.domain.user.User;
import lombok.Getter;

import java.io.Serializable;

@Getter
public class SessionUser implements Serializable {
    private String name;
    private String email;
    private String picture;

    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}
```

#### WHY, User 클래스를 쓰지 않고 새로 만들어서 쓰는가?
User 클래스가 엔티티이기 때문입니다. 엔티티 클래스에는 언제 다른 엔티티와 관계가 형성될지 모릅니다. 예를 들어 @OneToMany, @ManyToMany 등 자식 엔티티를 갖고 있다면 직렬화 대상에 자식들까지 포함되니 성능 이슈, 부수 효과가 발생할 확률이 높습니다. 그래서 직렬화 기능을 가진 세션 Dto를 하나 추가로 만드는 것이 이후 운영 및 유지보수 때 많은 도움이 됩니다.

#### 로그인 테스트

* index.mustache

```
<h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <!-- 로그인 기능 영역  참고!!! {.{#userName}.} 에서 . 은 지우고 사용해야 합니다. -->
        <div calss="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
                {.{#userName}.} <!-- (1) -->
                    Logged in as: <span id="user">{{userName}}</span>
                    <a href="/logout" class="btn btn-info active" role="button">Logout</a> <!-- (2) -->
                {.{/userName}.}

                {.{^userName}.} <!-- (3) -->
                    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a> <!-- (4) -->
                {.{/userName}.}
            </div>
        </div>
    </div>
    <br>
    <!--목록 출력 영역-->
```

* (1) `{.{#userName}.}`
  + 머스테치는 다른 언어와 같은 if문(if userName != null)을 제공하지 않습니다.
  + true/false 여부만 판단할 뿐입니다.
  + 그래서 머스테치에서는 항상 최종값을 넘겨줘야 합니다.
  + 여기서도 역시 userName이 있다면 userName을 노출시키도록 구성했습니다.
* (2) `a href="/logout"`
  + 스프링 시큐리티에서 기본적으로 제공하는 로그아웃 URL 입니다.
  + 즉, 개발자가 별도로 저 URL에 해당하는 컨트롤러를 만들 필요가 없습니다.
  + SecurityConfig 클래스에서 URL을 변경할 순 있지만 기본 URL을 사용해도 충분하니 여기서는 그래도 사용합니다.
* (3) `{.{^userName}.}`
  + 머스테치에서 해당 값이 존재하지 않는 경우에는 ^를 사용합니다.
  + 여기서는 userName이 없다면 로그인 버튼을 노출시키도록 구성했습니다.
* (4) `a href="/oauth2/authorization/google"`
  + 스프링 시큐리티에서 기본적으로 제공하는 로그인 URL입니다.
  + 로그아웃 URL과 마찬가지로 개발자가 별도의 컨트롤러를 생성할 필요가 없습니다.


index.mustache에서 userName을 사용할 수 있게 IndexController에서 userName을 model에 저장하는 코드를 추가해줍니다.

* IndexController

```
...
public class IndexController {
    ...
    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());

        SessionUser user = (SessionUser) httpSession.getAttribute("user"); // (1)
        if(user != null) { // (2)
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }
    ...
}
```

* (1) `(SessionUser) httpSession.getAttribute("user");`
  + 앞서 작성된 CustomOAuth2UserService에서 로그인 성공 시 세션에 SessionUser를 저장하도록 구성했습니다.
  + 즉, 로그인 성공 시 httpSession.getAttribute("user")에서 값을 가져올 수 있습니다.
* (2) `if(user != null)`
  + 세션에 저장된 값이 있을 때만 model에 userName으로 등록합니다.
  + 세션에 저장된 값이 없으면 model엔 아무런 값이 없는 상태이니 로그인 버튼이 보이게 됩니다.

로그인은 정상적으로 동작하는 것을 볼 수 있습니다. 그러나 게시글 등록은 403 에러를 띄우게 되는데 이는 사용자의 권한이 GUEST 이기 때문입니다. h2-console에서 __권한을 USER로 바꿔주면(update user set role="USER") 게시글 등록 또한 정상적으로 동작__ 함을 볼 수 있습니다.

# 애너테이션 기반으로 개선하기
일반적인 프로그래밍에서 개선이 필요한 나쁜 코드에는 __같은 코드가 반복__ 되는 부분입니다. 같은 코드가 반복되는 부분이 많으면 이후 수정이 필요할 때 모든 부분을 하나씩 찾아가며 수정해야만 합니다. 이렇게 될 경우 유지보수성이 떨어질 수 밖에 없으며, 혹시나 수정이 반영되지 않은 반복 코드가 있다면 문제가 발생할 수밖에 없습니다.

이러한 반복 코드가 발생하는 부분인 IndexController에서 아래 코드를 수정 해보겠습니다.

```
SessionUser user = (SessionUser) httpSession.getAttribute("user");
```

index 메소드 외에 다른 컨트롤러와 메소드에서 세션값이 필요하면 그 때마다 직접 세션에서 값을 가져와야 합니다. 그래서 이 부분을 __메소드 인자로 세션값을 바로 받을 수 있도록__ 변경해 보겠습니다.

config.auth 패키지에 __@LoginUser 어노테이션__ 을 생성합니다.

```
package com.doop.book.springboot.config.auth;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.PARAMETER) // (1)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser { // (2)

}
```

* (1) `@Target(ElementType.PARAMETER)`
  + 이 어노테이션이 생성될 수 있는 위치를 지정합니다.
  + PARAMETER로 지정했으니 메소드의 파라미터로 선언된 개겣에서만 사용할 수 있습니다.
  + 이 외에도 클래스 선언문에 쓸 수 있는 TYPE 등이 있습니다.
* (2) `@interface`
  + 이 파일을 어노테이션 클래스로 지정합니다.
  + LoginUser라는 이름을 가진 어노테이션이 생성되었다고 보면 됩니다.

그리고 같은 위치에 __LoginUserArgumentResolver를 생성__ 합니다. 이 클래스는 HandlerMethodArgumentResolver 인터페이스를 구현한 클래스입니다. 

HandlerMethodArgumentResolver는 한가지 기능을 지원합니다. 바로 조건에 맞는 경우 메소드가 있다면 HandlerMethodArgumentResolver의 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있습니다.

```
package com.doop.book.springboot.config.auth;

import com.doop.book.springboot.config.auth.dto.SessionUser;
import lombok.RequiredArgsConstructor;
import org.springframework.core.MethodParameter;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;

import javax.servlet.http.HttpSession;

@RequiredArgsConstructor
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {
    private final HttpSession httpSession;

    @Override
    public boolean supportsParameter(MethodParameter parameter) { // (1)
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());

        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) throws Exception { // (2)
        return httpSession.getAttribute("user");
    }
}
```

* (1) `supportsParameter()`
  + 컨트롤러 메서드의 특정 파라미터를 지원하는지 판단합니다.
  + 여기서는 파라미터에 @LoginUser 어노테이션이 붙어 있고, 파라미터 클래스 타입이 Sessionuser.class인 경우 true를 반환합니다.
* (2) `resolveArgument()`
  + 파라미터에 전달할 객체를 생성합니다.
  + 여기서는 세션에서 객체를 가져옵니다.


마지막으로 IndexController의 코드에서 반복되는 부분들을 모두 __@LoginUser로 개선__ 해 줍니다.
* IndexController

```
...
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user) { // (1)
        model.addAttribute("posts", postsService.findAllDesc());

        if(user != null) {
            model.addAttribute("userName", user.getName());
        }

        return "index";
    }
    ...
}
```

* (1) `@LoginUser SessionUser user`
  + 기존에 (User) httpSession.getAttribute("user") 로 가져오던 세션 정보 값이 개선 되었습니다.
  + 이제는 어느 컨트롤러든지 @Loginuser만 사용하면 세션 정보를 가져올 수 있게 되었습니다.

# 세션 저장소로 데이터베이스 사용하기
지금 까지 만든 로그인 기능은 애플리케이션을 재실행하면 로그인이 풀립니다. 그 이유는 세션이 __내장 톰켓의 메모리에 저장__ 되기 때문입니다. 기본적으로 세션은 실행되는 WAS의 메모리에서 저장되고 호출됩니다. 메모리에 저장되다 보니 __내장 톰캣처럼 애플리케이션 실행 시 실행되는 구조에선 항상 초기화__ 가 됩니다. 즉, 배포할 때마다 톰캣이 재시작되는 것 입니다. 이 외에도 한 가지 문제가 더 있습니다. 2대 이상의 서버에서 서비스하고 있다면 톰캣마다 세션 동기화 설정을 해야만 합니다. 그래서 실제 현업에서는 세션 저장소에 대해 다음의 3가지 중 한 가지를 선택합니다.

* __(1) 톰캣 세션을 사용한다__
  + 일반적으로 별다른 설정을 하지 않을 때 기본적으로 선택되는 방식입니다.
  + 이렇게 될 경우 톰캣(WAS)에 세션이 저장되기 떄문에 2대 이상의 WAS가 구동되는 환경에서는 톰캣들 간의 세션 공유를 위한 추가 설정이 필요합니다.
* __(2) MySQL과 같은 데이터베이스를 세션 저장소로 사용한다.__
  + 여러 WAS 간의 공용 세션을 사용할 수 있는 가장 쉬운 방법입니다.
  + 많은 설정이 필요 없지만, 결국 로그인 요청마다 DB IO가 발생하여 성능상 이슈가 발생할 수 있습니다.
  + 보통 로그인 요청이 많이 없는 백오피스, 사내 시스템 용도에서 사용합니다.
* __(3) Redis, Memcached와 같은 메모리 DB를 세션 저장소로 사용한다.__
  + B2C 서비스에서 가장 많이 사용하는 방식입니다.
  + 실제 서비스로 사용하기 위해서는 Embedded Redis와 같은 방식이 아닌 외부 메모리 서버가 필요합니다.

build.gradle에 __spring-session-jdbc 의존성을 등록__ 해 줍니다.

* build.gradle

```
compile('org.springframework.session:spring-session-jdbc')
```

그리고 application.properties에 세션 저장소를 jdbc로 선택하도록 코드를 추가합니다.

```
spring.session.store-type=jdbc
```

내용을 추가 하였으면 h2-console에서 결과를 확인 할 수 있습니다. 그러나 여전히 스프링을 재시작하면 세션이 풀리는데 이는 h2도 같이 재시작 되기 때문입니다. 이는 추후 AWS로 배포하면 해결될 문제이니 넘어가겠습니다. 

# 네이버 로그인 
네이버 로그인을 추가하기 위해서 [네이버 로그인 서비스 등록](https://developers.naver.com/apps/#/register?api=nvlogin) 에 접속하여 아래 내용을 참고하여 서비스를 등록합니다.

1. 서비스 URL: `http://localhost:8080/`
2. 네이버아이디로로그인 Callback URL: `http://localhost:8080/login/auth2/code/naver`

등록을 완료하였으면 application-oauth.properties에 키값들을 등록해 줍니다.

네이버에서는 스프링 시큐리티를 공식 지원하지 않기 때문에 그동안 Common-OAuth2Provider에서 해주던 값들도 전부 수동으로 입력해야 합니다.

```
...
# registration
spring.security.oauth2.client.registration.naver.client-id=클라이언트ID
spring.security.oauth2.client.registration.naver.client-secret=클라이언트 비밀
spring.security.oauth2.client.registration.naver.redirect-uri={baseUrl}/{action}/oauth2/code/{registrationId}
spring.security.oauth2.client.registration.naver.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.naver.scope=name,email,profile_image
spring.security.oauth2.client.registration.naver.client-name=Naver

# provider -> https://developers.naver.com/apps/#/myapps/eR05U0GAJw6PSXCVMKSs/overview
spring.security.oauth2.client.provider.naver.authorization-uri=https://nid.naver.com/oauth2.0/authorize
spring.security.oauth2.client.provider.naver.token-uri=https://nid.naver.com/oauth2.0/token
spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
spring.security.oauth2.client.provider.naver.user-name-attribute=response
```

* (1) `user-name-attribute=response`
  + 기준이 되는 user-name의 이름을 네이버에서는 response로 해야 합니다.
  + 이유는 네이버의 회원 조회 시 반환되는 JSON 형태 떄문입니다.

스프링 시큐리티에선 __하위 필드를 명시할 수 없습니다.__ 최상위 필드만 user-name으로 지정 가능합니다. 하지만 네이버의 응답값 최상위 필드는 resultCode, message, response 입니다. 이러한 이유로 스프링 시큐리티에서 인식 가능한 필드는 저 3개 중에 골라야 합니다. 본문에서 담고 있는 response를 user-name으로 지정하고 이후 자바 코드로 response의 id를 user-name으로 지정하겠습니다.

네이버 로그인을 등록하기 위해서 __OAuthAttributes에 네이버인지 판단하는 코드와 네이버 생성자만 추가__ 해 주면 됩니다. 추가로  __index.mustache에 네이버 로그인 버튼을 추가__ 해 줍니다.

* OAuthAttributes

```
@Getter
public class OAuthAttributes {
    ...

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        if("naver".equals(registrationId)) {
            return ofNaver("id", attributes);
        }

        return ofGoogle(userNameAttributeName, attributes);
    }
    ...
    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes) {
        Map<String, Object> response = (Map<String, Object>) attributes.get("response");

        return OAuthAttributes.builder()
                .name((String) response.get("name"))
                .email((String) response.get("email"))
                .picture((String) response.get("profile_image"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }
    ...
}
```

* index.mustache

```
...
    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
    <a href="/oauth2/authorization/naver" class="btn btn-secondary active" role="button">Naver Login</a> <!-- (1) -->
{.{/userName}.}
...
```

* (1) `href="/oauth2/authorization/naver"`
  + 네이버 로그인 URL은 application-auth.properties에 등록한 redirect-uri 값에 맞춰 자동으로 등록됩니다.
  + /oauth2/authorization/ 까지는 고정이고 마지막 Path만 각 소셜 로그인 코드를 사용하면 됩니다.
  + 여기서는 naver가 마지막 Path가 됩니다.

이것으로 구글과 네이버 로그인 기능을 적용해 보았습니다. 마지막으로 기존 테스트 케이스에 시큐리티를 적용해 보겠습니다.

# 기존 테스트에 시큐리티 적용하기
시큐리티를 적용하면서 기존 테스트들에 문제가 발생하였습니다. 그 이유는 기존에는 바로 API를 호출할 수 있어 테스트 코드 역시 바로 API를 호출하도록 구성하였습니다. 하지만, 시큐리티 옵션이 활성화되면 인증된 사용자만 API를 호출할 수 있습니다. 기존의 API 테스트 코드들이 모두 인증에 대한 권한을 받지 못하였으므로, 테스트 코드마다 인증한 사용자가 호출한 것처럼 작동하도록 수정하겠습니다. 

gradle > Tasks > verification > test 실행 해보면 모든 테스가 실패함을 볼 수 있습니다.

## 문제 1. CustomOAuth2UserService을 찾을 수 없음
이 문제는 CustomOAuth2UserService를 생성하는데 필요한 소셜 로그인 관련 설정값들이 없기 때문에 발생합니다. application-oauth.properties에 설정값을 추가 했지만 __이는 src/main 환경과 src/test 환경의 차이 때문입니다.__ 둘은 본인만의 환경 구성을 가집니다. 다만, src/main/resources/application.properties가 테스트 코드를 수행할 때도 적용되는 이유는 __test에 application.properties가 없으면 main의 설정을 그대로 가져오기__ 때문입니다. __다만, 자동으로 가져오는 옵션의 범위는 application.properties 파일까지입니다.__ (즉, application-oauth.properties는 안가져옴)

이 문제를 해결하기위해 __src/test/resources/application.properties 를 생성하여 가짜 설정 값을 등록__ 합니다. 

```
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.h2.console.enabled=true
spring.session.store-type=jdbc

# Test OAuth
spring.security.oauth2.client.registration.google.client-id=test
spring.security.oauth2.client.registration.google.client-secret=test
spring.security.oauth2.client.registration.google.scope=profile,email
```

## 문제 2. 302 Status Code
두 번째로 "Posts_등록된다" 테스트 로그를 확인해 보면 응답 결과로 200(정상)이 아닌 302(리다이렉션 응답) Status Code가 와서 실패 했습니다. __이는 스프링 시큐리티 설정 때문에 인증되지 않은 사용자의 요청은 이동시키기 때문입니다.__ 그래서 이런 API 요청은 임의로 인증된 사용자를 추가하여 API만 테스트해 볼 수 있게 하겠습니다.

build.gradle에 __스프링 시큐리티 테스트를 위한 여러 도구를 지원__ 하는 의존성을 추가해줍니다.

```
testCompile('org.springframework.security:spring-security-test') // chap5.7
```

그리고 __PostsApiControllerTest__ 의 2개의 테스트 메소드에 다음과 같이 __임의의 사용자 인증을 추가__ 합니다.

```
package com.doop.book.springboot.web;
...
public class PostsApiControllerTest {
    ...
    @Test
    @WithMockUser(roles = "USER") // (1)
    public void Posts_등록된다() throws Exception {
        ...
    }

    @Test
    @WithMockUser(roles = "USER")
    public void Posts_수정된다() throws Exception {
        ...
    }
}
```

* (1) `@WithMockUser(roles = "USER")`
  + 인증된 모의(가짜) 사용자를 만들어서 사용합니다.
  + roles에 권한을 추가할 수 있습니다.
  + 즉, 이 어노테이션으로 인해 ROLE_USER 권한을 가진 사용자가 API를 요청하는 것과 동일한 효과를 가지게 됩니다.

이렇게 수정을 하면 실제로 작동하진 않습니다. __@WithMockUser가 MockMvc에서만 작동하기 때문입니다.__ 현재 PostsApiControllerTestsms @SpringBootTest로만 되어있으며 MockMvc를 전혀 사용하지 않습니다. 그래서 @SpringBootTest에서 MockMvc를 사용할 수 있게 합니다.

```
package com.doop.book.springboot.web;

...
import org.springframework.http.MediaType;
import com.doop.book.springboot.domain.posts.Posts;
import com.doop.book.springboot.domain.posts.PostsRepository;
import com.doop.book.springboot.web.dto.PostsSaveRequestDto;
import com.doop.book.springboot.web.dto.PostsUpdateRequestDto;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MockMvcBuilder;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.security.test.web.servlet.setup.SecurityMockMvcConfigurers.springSecurity;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {
    ...

    @Autowired
    private WebApplicationContext context;

    private MockMvc mvc;

    @Before // chap5.7 (1)
    public void setup() {
        mvc = MockMvcBuilders.webAppContextSetup(context)
                .apply(springSecurity())
                .build();
    }
    ...

    @Test
    @WithMockUser(roles = "USER")
    public void Posts_등록된다() throws Exception {
        ...
        // when
        mvc.perform(post(url) // chap5.7 (2)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());


        // then
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }

    @Test
    @WithMockUser(roles = "USER")
    public void Posts_수정된다() throws Exception {
        ...

        // when
        mvc.perform(put(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());
        // then
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }
}
```

* (1) `@Before`
  + 매번 테스트가 시작되기 전에 MockMvc 인스턴스를 생성합니다.
* (2) `mvc.perform`
  + 생성된 MockMvc를 통해 API를 테스트 합니다.
  + 본문 영역은 문자열로 표현하기 위해 ObjectMapper를 통해 문자열 JSON으로 변환합니다.

## 문제 3. @WebMvcTest에서 CustomOAuth2UserService을 찾을 수 없음
문제 1번과 동일한 문제가 발생했습니다. 그러나 1번과는 조금 다른점이 잇습니다. 문제 3번은 @WebMvcTest를 사용한다는 점입니다. 1번을 통해 스프링 시큐리티 설정은 잘 작동했지만, __@WebMvcTest는 CustomOAuth2UserService를 스캔하지 않기__ 때문입니다. __@WebMvcTest는 WebSecurityConfigurerAdapter, WebMvcConfigurer를 비롯한 @ControllerAdvice, @Controller를 읽습니다.__ 즉, __@Repository, @Service, @Component는 스캔 대상이 아닙니다.__ 그러니 SecurityConfig는 읽었지만, SecurityConfig를 생성하기 위해 필요한 CustomOAuth2UserService는 읽을수가 없어 앞에서와 같은 에러가 발생한 것입니다. 이 문제를 해결하기 위해 __스캔 대상에서 SecurityConfig를 제거__ 합니다.

* HelloControllerTest

```
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class,
        excludeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,
                classes = SecurityConfig.class)
                }
            )
public class HelloControllerTest {
    ...
```

그리고 __@WithMockUser를 사용해서 가짜로 인증된 사용자를 생성__ 합니다.

```
...
public class HelloControllerTest {
    ...
    @WithMockUser(roles = "USER")
    @Test
    public void hello가_리턴된다() throws Exception {
        ...
```

이렇게 설정을하고 다시 테스트 해보면 __@EnableJpaAuditing로 인한 추가 에러가 발생__ 합니다. @EnableJpaAuditing를 사용하기 위해선 최소 하나의 __@Entity 클래스가 필요__ 합니다. @WebMvcTest이다 보니 당연히 없습니다. 

@EnableJpaAuditing가 @SpringBootApplication와 함께 있다보니 @WebMvcTest에서도 스캔하게 되었습니다. 그래서 @EnableJpaAuditing과 @SpringBootApplication 둘을 분리하겠습니다.

1. Application.java에서 __@EnableJpaAuditing를 제거__ 합니다.
2. config패키지에 JpaConfig를 생성하여 __@EnableJpaAuditing를 추가__ 합니다

* JpaConfig

```
package com.doop.book.springboot.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@Configuration
@EnableJpaAuditing // JPA Auditing 활성화
public class JpaConfig {
}
```

다시 테스트를 수행하면 모든 테스트가 통과함을 볼 수 있습니다. 


# Related Posts
* [Chap 1.스프링 부트 시작하기](https://doorisopen.github.io/spring/2020/02/24/spring-freelec-springboot-chap1.html)
* [Chap 2.테스트 코드 작성하기](https://doorisopen.github.io/spring/2020/02/24/spring-freelec-springboot-chap2.html)
* [Chap 3.스프링 부트에서 JPA사용하기](https://doorisopen.github.io/spring/2020/02/26/spring-freelec-springboot-chap3.html)
* [Chap 4.Mustache로 화면 구성하기](https://doorisopen.github.io/spring/2020/03/03/spring-freelec-springboot-chap4.html)
* [Chap 5.스프링 시큐리티와 OAuth2.0 - Now](#)
* [Chap 6.AWS EC2 서버 환경 구축하기](https://doorisopen.github.io/spring/2020/03/10/spring-freelec-springboot-chap6.html)
* [Chap 7.AWS RDS 생성하기](https://doorisopen.github.io/spring/2020/03/11/spring-freelec-springboot-chap7.html)
* [Chap 8.EC2 서버에 프로젝트 배포하기](https://doorisopen.github.io/spring/2020/03/12/spring-freelec-springboot-chap8.html)
* [Chap 9.Travis CI 배포 자동화](https://doorisopen.github.io/spring/2020/03/13/spring-freelec-springboot-chap9.html)
* [Chap 10.Nginx를 이용한 무중단 배포](https://doorisopen.github.io/spring/2020/03/18/spring-freelec-springboot-chap10.html)

# References
* 스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)