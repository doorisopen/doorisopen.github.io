---
layout: post
title:  "[Spring Boot] Chap 10.Nginx를 이용한 무중단 배포"
date:   2020-03-18 00:30:59
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
>> 1. 무중단 배포(Nginx)
>> 2. EC2 서버에 Nginx 설치 -> 서비스 시작
>> 3. EC2 보안 그룹 추가 : 80 포트
>> 4. 구글, 네이버 리디렉션 URI 추가
>> 5. 프로젝트와 Nginx 연동
>> 6. 무중단 배포 스크립트 작성

<br />
<br />


<hr />

> __chap8 ~ 10__ 은 __프로젝트 배포와 관련된 내용__ 을 포스팅 할 예정입니다.
> 
> 전체적인 __흐름을 단계별로 설명__ 하겠습니다. 각 단계는 __배포 방식을 개선하는 것으로__ 총 3단계로 진행됩니다.
> 
> * __Chap 8: Step 1.__ 스크립트를 실행하여 __수동__ 으로 프로젝트 Test & build 하기
>   + 프로젝트 설정 - MariaDB 드라이버 등록(build.gradle)
>   + RDB에 프로젝트에 사용되는 테이블 생성
>   + 외부 Security 파일 등록
>   + 배포 스크립트 작성
>   + EC2 설정 - RDS 접속 정보 설정
>   + EC2에서 소셜 로그인
> * __Chap 9: Step 2.__ 깃허브에 Push 하면 __자동__ Test & Build & Deploy
>   + Github와 Travis CI 연동
>   + 프로젝트 Travis CI(.travis.yml) 설정
>   + Travis CI와 AWS S3 연동
>     - AWS Key(IAM, identity and Access Management) 발급
>     - Travis CI에 IAM키 등록
>     - AWS S3 버킷 생성
>     - Travis CI의 빌드내용(Jar)을 S3에 올리기 위해 프로젝트(.travis.yml)에 설정 추가
>   + Travis CI와 AWS S3, CodeDeploy 연동하기
>     - EC2와 CodeDeploy 연동
>     - CodeDeploy 연동을 위해 __EC2에서 사용할 IAM 역할 생성__
>     - EC2 서버에 CodeDeploy 에이전트 설치
>     - CodeDeploy -> EC2 접근을 위해 __CodeDeploy에서 사용할 IAM 역할 생성__
>     - CodeDeploy 생성
>     - CodeDeploy 관련 설정을 appspec.yml에 추가
>     - Travis CI 설정 파일(.travis.yml)에 CodeDeploy 내용을 추가
>   + 배포 자동화 구성(스크립트 파일(.sh) 작성)
>     - 배포를 위한 스크립트(Jar, appspec.yml)가 아닌 것을 제외하기 위해 .travis.yml 내용 수정
>     - Codedeploy 명령을 담당할 appspec.yml 파일 수정
>   + CodeDeploy 로그 확인
> * __Chap 10: Step 3.__ Nginx __무중단__ 배포
>   + EC2 서버에 Nginx 설치 -> 서비스 시작
>   + EC2 보안 그룹 추가 : 80 포트
>   + 구글, 네이버 리디렉션 URI 추가
>   + 프로젝트와 Nginx 연동
>   + 무중단 배포 스크립트 작성
>     - 8001, 8002 어느 포트를 사용할지 판단하는 API 작성(/profile-real: TravisCI 배포 자동화를 위한 profile 입니다.)
>     - 무중단 배포를 위한 profile 2개(real1, real2) 추가(application-real1,2.properties 파일 생성)
>     - EC2 서버의 Nginx 설정 수정
>     - 배포 장소 변경/ 배포 스크립트 사용할 수 있도록 appspec.yml 내용 수정
>     - 프로젝트에 배포 스크립트 작성(5개, profile, start, stop, health, switch)


chap 9(step 2)는 Travis CI를 활용하여 배포 자동화 환경을 구축해 보았습니다. 그러나 __배포하는 동안 스프링 부트 프로젝트는 종료 상태가 되어 서비스를 이용할 수 없다__ 는 문제점이 있었습니다. 새로운 Jar가 실행되기 전까진 기존 Jar를 종료시켜 놓기 때문에 서비스가 중단됩니다. 

SpringBoot 시리즈의 마지막인 __chap 10(step 3)__ 에서는 __서비스 중단 없는 배포 방법__ 으로 개선해보겠습니다.

# 무중단 배포(Nginx)
무중단 배포 방식에는 몇 가지가 있습니다.

* AWS에서 블루 그린(Blue-Green)무중단 배포
* 도커를 이용한 웹서비스 무중단 배포
* L4 스위치를 이용한 무중단 배포(L4는 매우 고가의 장비라 대형 인터넷 기업에서 사용함)

우리가 무중단 배포를 진행할 방법은 __엔진엑스(Nginx)__ 를 이용한 무중단 배포입니다. __엔진엑스(Nginx)__ 는

* 웹 서버, 리버스 프록시, 캐싱, 로드 밸런싱, 미디어스트리밍 등을 위한 오픈소스 소프트웨어입니다.
* 이전 아파치(Apache)가 대세였던 자리를 완전히 빼앗은 가장 유명한 웹 서버이자 오픈소스입니다.
* 무중단 배포를 구현하는데 가장 저렴하고 쉬운 방법입니다.
  + 사내 비용 지원이 많다면 AWS 블루 그린 배포 방식을 선택하면됩니다.
* 엔진엑스 기능 중 __리버스 프록시__ 를 통해 무중단 배포 환경을 구축할 수 있습니다.
  + __리버스 프록시란__ 엔진엑스가 __외부의 요청을 받아 백앤드 서버로 요청을 전달하는 행위__ 입니다.
  + 리버스 프록시 서버(엔진엑스)는 요청을 전달하고, 실제 요청에 대한 처리는 뒷단의 웹 애플리케이션 서버들이 처리합니다.
 
이 방식은 꼭 AWS와 같은 클라우드 인프라가 구축되어 있지 않아도 사용할 수 있는 범용적인 방법입니다. 즉, 개인 서버 혹은 사내 서버에서도 동일한 방식으로 구축할 수 있으므로 사용처가 많습니다.


## 무중단 배포 구조
하나의 EC2 혹은 리눅스 서버에 엔진엑스 1대와 스프링 부트 Jar를 2대를 사용하는 것입니다.

* 엔진엑스 80(Http), 443(Https) 포트를 할당합니다.
* 스프링 부트1은 8081포트로 실행합니다.
* 스프링 부트2는 8082포트로 실행합니다.

<a href="{{ site.2020_spring_img }}/freelec-springboot-chap10-1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2020_spring_img }}/freelec-springboot-chap10-1.JPG" title="Check out the image">
</a>

1. 사용자는 서비스 주소로 접속합니다(80 혹은 443 포트)
2. 엔진엑스는 사용자의 요청을 받아 현재 연결된 스프링 부트로 요청을 전달합니다.
3. 스프링 부트2는 엔진엑스와 연결된 상태가 아니니 요청받지 못합니다.

1.1 버전으로 신규 배포가 필요하면, 엔진엑스와 연결되지 않은 스프링부트2(8082 포트)로 배포합니다.

<a href="{{ site.2020_spring_img }}/freelec-springboot-chap10-2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2020_spring_img }}/freelec-springboot-chap10-2.JPG" title="Check out the image">
</a>

1. 배포하는 동안에도 서비스는 중단되지 않습니다.
2. 배포가 끝나고 정상적으로 스프링 부트2가 구동 중인지 확인합니다.
3. 스프링 부트2가 정상 구동 중이면 nginx reload 명령어를 통해 8081 대신에 8082를 바라보도록 합니다.
4. nginx reload는 0.1초 이내에 완료됩니다.

이후 1.2 버전 배포가 필요하면 이번에는 스프링 부트1로 배포합니다.

<a href="{{ site.2020_spring_img }}/freelec-springboot-chap10-3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2020_spring_img }}/freelec-springboot-chap10-3.JPG" title="Check out the image">
</a>

1. 현재는 엔진엑스와 연결된 것이 스프링 부트2입니다.
2. 스프링 부트1의 배포가 끝났다면 엔진엑스가 스프링 부트1을 바라보도록 변경하고 nginx reload를 실행합니다.
3. 이후 요청부터는 엔진엑스가 스프링 부트 1로 요청을 전달합니다.

<a href="{{ site.2020_spring_img }}/freelec-springboot-chap10-4.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2020_spring_img }}/freelec-springboot-chap10-4.JPG" title="Check out the image">
</a>

# EC2 서버에 Nginx 설치 -> 서비스 시작
EC2 서버에 Nginx를 설치&서비스 실행 합니다.

```
sudo yum install nginx

sudo service nginx start
```

# EC2 보안 그룹 추가 : 80 포트
엔진엑스의 포트번호는 기본적으로 80입니다. 해당 포트 번호가 EC2 인스턴스의 보안 그룹에 없으니 __EC2 > 보안 그룹 > EC2 보안 그룹 선택 > 인바운드 편집__ 에서 80 포트를 추가해 줍니다.


# 구글, 네이버 리디렉션 URI 추가
8080이 아닌 80포트로 주소가 변경되니 구글과 네이버 로그인에도 변경된 주소를 등록해야만 합니다. 기존에 등록된 리디렉션 주소에서 8080 부분을 제거하여 추가 등록합니다. 아래와 같이 __3 개__ 가 등록되면 됩니다.

* 승인된 리디렉션 URI(구글 기준 네이버도 같은 방식으로 등록합니다.)
  + http://localhost:8080/login/oauth2/code/google
  + http://아마존퍼블릭URL:8080/login/oauth2/code/google
  + http://아마존퍼블릭URL/login/oauth2/code/google

등록 완료 하였다면 __8080을 제외하고 EC2 도메인으로 접근하면 엔진엑스 웹페이지를 확인하면 성공__ 입니다.

# 프로젝트와 Nginx 연동
엔진엑스가 현재 실행 중인 스프링 부트 프로젝트를 바라볼 수 있도록 __프록시 설정__ 을 하겠습니다.

```
sudo vim /etc/nginx/nginx.conf
================================================
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  localhost;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
                proxy_pass http://localhost:8080; // (1)
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; // (2)
                proxy_set_header Host $http_host;
        }

        # redirect server error pages to the static page /40x.html
        #
        error_page 404 /404.html;
            location = /40x.html {
        }
```

* (1) `proxy_pass`
  + 엔진엑스로 요청이 오면 http://localhost:8080로 전달합니다.
* (2) `proxy_set_header XXX`
  + 실제 요청 데이터를 header의 각 항목에 할당합니다.
  + 예) proxy_set_header X-Real-IP $remote_addr: Request Header의 X-Real-IP에 요청자의 IP를 저장합니다.

위 항목을 추가하고 엔진엑스를 재시작합니다.

# 무중단 배포 스크립트 작성
## 어느 포트를 사용할지 판단하는 API 작성
8001, 8002 어느 포트를 사용할지 판단하는 API는 __ProfileController__ 가 담당하겠습니다. 

```
package com.doop.book.springboot.web;

import lombok.RequiredArgsConstructor;

import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;
import java.util.List;

@RequiredArgsConstructor
@RestController
public class ProfileController {
    private final Environment env;

    @GetMapping("/profile")
    public String profile() {
        List<String> profiles = Arrays.asList(env.getActiveProfiles()); // (1)
        List<String> realProfiles = Arrays.asList("real","real1","real2");
        String defaultProfile = profiles.isEmpty()? "default" : profiles.get(0);

        return profiles.stream()
                .filter(realProfiles::contains)
                .findAny()
                .orElse(defaultProfile);
    }

}
```

* (1) `env.getActiveProfiles()`
  + 현재 실행 중인 ActiveProfile을 모두 가져옵니다.
  + 즉, real, oauth, real-db 등이 활성화되어 있다면(active) 3개가 모두 담겨 있습니다.
  + 여기서 real. real1, real2는 모두 배포에 사용될 profile이라 이 중 하나라도 있으면 그 값을 반환하도록 합니다.
  + 실제로 이번 무중단 배포에서는 real1과 real2만 사용되지만, step2를 다시 사용해 볼 수도 있으니 real도 남겨둡니다.

* ProfileControllerUnit 테스트 코드 작성

```
package com.doop.book.springboot.web;

import org.junit.Test;
import org.springframework.mock.env.MockEnvironment;

import static org.assertj.core.api.Assertions.assertThat;

public class ProfileControllerUnitTest {

    @Test
    public void real_profile이_조회된다() {
        // given
        String expectedProfile = "real";
        MockEnvironment env = new MockEnvironment();
        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("oauth");
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        // when
        String profile = controller.profile();

        // then
        assertThat(profile).isEqualTo(expectedProfile);
    }

    @Test
    public void real_profile이_없으면_첫번째가_조회된다() {
        // given
        String expectedProfile = "oauth";
        MockEnvironment env = new MockEnvironment();

        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        // when
        String profile = controller.profile();

        // then
        assertThat(profile).isEqualTo(expectedProfile);
    }

    @Test
    public void active_profile이_없으면_default가_조회된다() {
        // given
        String expectedProfile = "default";
        MockEnvironment env = new MockEnvironment();
        ProfileController controller = new ProfileController(env);

        // when
        String profile = controller.profile();

        // then
        assertThat(profile).isEqualTo(expectedProfile);
    }
}
```

ProfileController나 Environment 모두 자바 클래스(인터페이스)이기 때문에 쉽게 테스트할 수 있습니다. Environment는 인터페이스라 가짜 구현체인 MockEnvironment(스프링에서 제공)를 사용해서 테스트하면 됩니다. 이렇게 해보면 __생성자 DI가 얼마나 유용한지__ 알 수 있습니다. 만약 @Autowired로 DI 받았다면 이런 테스트 코드를 작성하지 못했습니다. 항상 스프링 테스트를 해야했을 것입니다.

테스트가 통과 했다면 __SecurityConfig 클래스에 /profile를 제외__ 하는 코드를 추가합니다.

```
.antMatchers("/","/css/**","/images/**","/js/**","/h2-console/**","/profile").permitAll()
```

추가했다면 SecurityConfig 설정이 잘 되었는지 테스트 코드를 작성합니다.

```
package com.doop.book.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ProfileControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void profile은_인증없이_호출된다() throws Exception {
        String expected = "default";

        ResponseEntity<String> response = restTemplate.getForEntity("/profile", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).isEqualTo(expected);
    }
}
```

테스트가 성공했다면 깃허브에 커밋/푸시하고 __브라우저에 /profile 에 접속하여 화면에 real이 보이는지 확인__ 합니다. 

## 무중단 배포를 위한 profile 2개(real1, real2) 추가(application-real1,2.properties 파일 생성)
현재 EC2 환경에서 실행되는 profile은 real밖에 없습니다. 해당 profile(real)은 Travis CI 배포 자동화를 위한 profile이니 __무중단 배포를 위한 profile 2개를 추가__ 합니다.

```
# resources/application-real1.properties

server.port=8081
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc

=============================================
# resources/application-real2.properties

server.port=8082
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```

생성하였다면 깃허브에 커밋/푸시합니다.

## EC2 서버의 Nginx 설정 수정
무중단 배포의 핵심은 __엔진엑스 설정__ 입니다. 배포 때마다 엔진엑스의 프록시 설정(스프링 부트로 요청을 흘려보내는)이 순식간에 교체됩니다. 여기서 프록시 설정이 교체될 수 있도록 설정을 추가하겠습니다. __엔진엑스 설정이 모여있는 /etc/nginx/conf.d 에 service-url.inc라는 파일을 하나 생성__ 합니다.

```
sudo vim /etc/nginx/conf.d/service-url.inc > 아래 내용 작성했다면 저장하고 종료합니다. 
================================================
set $service_url http://127.0.0.1:8080;
```

그리고 이 파일을 엔진엑스가 사용할 수 있게 설정합니다.

```
sudo vim /etc/nginx/nginx.conf
================================================
include /etc/nginx/conf.d/service-url.inc; // <--- 내용 추가

location / {
        proxy_pass $service_url; // <--- 내용 수정
        proxy_set_header X-Real-IP $remote_addr;
```

저장하고 종료한뒤(:wq) 엔진액스를 __재시작__ 합니다.

## 배포 장소 변경/ 배포 스크립트 사용할 수 있도록 appspec.yml 내용 수정
먼저 step2와 중복되지 않기 위해 EC2에 step3 디렉토리를 생성합니다.

```
mkdir ~/app/step3 && mkdir ~/app/step3/zip
```

__appspec.yml__ 에서 step3로 배포되도록 수정합니다.

```
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step3/zip/
    overwrite: yes

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  AfterInstall:
    - location: stop.sh # 엔진엑스와 연결되어 있지 않은 스프링 부트를 종료합니다.
      timeout: 60
      runas: ec2-user
  ApplicationStart:
    - location: start.sh # 엔진엑스와 연결되어 있지 않은 Port로 새 버전의 스프링 부트를 시작합니다.
      timeout: 60
      runas: ec2-user
  ValidateService:
    - location: health.sh # 새 스프링 부트가 정상적으로 실행됬는지 확인합니다.
      timeout: 60
      runas: ec2-user
```

## 프로젝트에 배포 스크립트 작성(5개, profile, start, stop, health, switch)
* __stop.sh:__ 기존 엔진엑스에 연결되어 있진 않지만, 실행 중이던 스프링 부트 종료
* __start.sh:__ 배포할 신규 버전 스프링 부트 프로젝트를 stop로 종료한 'profile.sh'로 실행
* __health.sh:__ 'start.sh'로 실행시킨 프로젝트가 정상적으로 실행됐는지 체크
* __switch.sh:__ 엔진엑스가 바라보는 스프링 부트를 최신 버전으로 변경
* __profile.sh:__ 앞선 4개 스크립트 파일에서 공용으로 사용할 'profile'과 포트 체크 로직

위의 5개의 스크립트 파일을 __scripts/ 경로에 생성 하고 코드를 작성한다.__

* __profile.sh__

```
#!/usr/bin/env bash

# bash는 return value가 안되니 *제일 마지막줄에 echo로 해서 결과 출력*후, 클라이언트에서 값을 사용한다

# 쉬고 있는 profile 찾기: real1이 사용중이면 real2가 쉬고 있고, 반대면 real1이 쉬고 있음
function find_idle_profile()
{
    RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile) # (1)

    if [ ${RESPONSE_CODE} -ge 400 ] # 400 보다 크면 (즉, 40x/50x 에러 모두 포함)
    then
        CURRENT_PROFILE=real2
    else
        CURRENT_PROFILE=$(curl -s http://localhost/profile)
    fi

    if [ ${CURRENT_PROFILE} == real1 ]
    then
      IDLE_PROFILE=real2 # (2)
    else
      IDLE_PROFILE=real1
    fi

    echo "${IDLE_PROFILE}" # (3)
}

# 쉬고 있는 profile의 port 찾기
function find_idle_port()
{
    IDLE_PROFILE=$(find_idle_profile)

    if [ ${IDLE_PROFILE} == real1 ]
    then
      echo "8081"
    else
      echo "8082"
    fi
}
```

* (1) `$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)`
  + 현재 엔진엑스가 바라보고 있는 스프링 부트가 정상적으로 수행 중인지 확인합니다.
  + 응답값을 HttpStatus로 받습니다.
  + 정상이면 200, 오류가 발생한다면 400~503 사이로 발생하니 400 이상은 모두 예외로 보고 real2를 현재 profile로 사용합니다.
* (2) `IDLE_PROFILE`
  + 엔진엑스와 연결되지 않은 profile입니다.
  + 스프링 부트 프로젝트를 이 profile로 연결하기 위해 반환합니다.
* (3) `echo "${IDLE_PROFILE}"`
  + bash라는 스크립트는 값을 반환하는 기능이 없습니다.
  + 그래서 제일 마지막 줄에 echo로 결과를 출력 후, 클라이언트에서 그 값을 잡아서 ($(find_idle_profile))사용합니다.
  + 중간에 echo를 사용해선 안 됩니다.


* __stop.sh__

```
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH) # (1)
source ${ABSDIR}/profile.sh # (2)

IDLE_PORT=$(find_idle_port)

echo "> $IDLE_PORT 에서 구동중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

if [ -z ${IDLE_PID} ]
then
  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 ${IDLE_PID}
  sleep 5
fi
```

* (1) `ABSDIR=$(dirname $ABSPATH)`
  + 현재 stop.sh가 속해 있는 경로를 찾습니다.
  + 하단의 코드와 같이 profile.sh의 경로를 찾기 위해 사용됩니다.
* (2) `source ${ABSDIR}/profile.sh`
  + 자바로 보면 일종의 import 구문입니다.
  + 해당 코드로 인해 stop.sh에서도 profile.sh의 여러 function을 사용할 수 있게 됩니다.


* __start.sh__

```
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

REPOSITORY=/home/ec2-user/app/step3
PROJECT_NAME=freelec-springboot2-webservice

echo "> Build 파일 복사"
echo "> cp $REPOSITORY/zip/*.jar $REPOSITORY/"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 새 어플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

IDLE_PROFILE=$(find_idle_profile)

echo "> $JAR_NAME 를 profile=$IDLE_PROFILE 로 실행합니다."
nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-$IDLE_PROFILE.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=$IDLE_PROFILE \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

* (1)기본적인 스크립트는 step2의 deploy.sh와 유사합니다.
* (2)다른 점이라면 IDLE_PROFILE을 통해 properties 파일을 가져오고(application-#IDLE_PROFILE.properties), active profile을 지정하는 것(-Dspring.profiles.active=$IDLE_PROFILE) 뿐입니다.
* (3)여기서도 IDLE_PROFILE을 사용하니 profile.sh을 가져와야 합니다.

* __health.sh__

```
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh

IDLE_PORT=$(find_idle_port)

echo "> Health Check Start!"
echo "> IDLE_PORT: $IDLE_PORT"
echo "> curl -s http://localhost:$IDLE_PORT/profile "
sleep 10

for RETRY_COUNT in {1..10}
do
  RESPONSE=$(curl -s http://localhost:${IDLE_PORT}/profile)
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)

  if [ ${UP_COUNT} -ge 1 ]
  then # $up_count >= 1 ("real" 문자열이 있는지 검증)
      echo "> Health check 성공"
      switch_proxy
      break
  else
      echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
      echo "> Health check: ${RESPONSE}"
  fi

  if [ ${RETRY_COUNT} -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi

  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
```

* (1)엔진액스와 연결되지 않은 포트로 스프링 부트가 잘 수행되었는지 체크합니다.
* (2)잘 떴는지 확인되어야 엔진엑스 프록시 설정을 변경(switch_proxy)합니다.
* (3)엔진엑스 프록시 설정 변경은 switch.sh에서 수행합니다.

* __switch.sh__

```
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

function switch_proxy() {
    IDLE_PORT=$(find_idle_port)

    echo "> 전환할 Port: $IDLE_PORT"*
    echo "> Port 전환"
    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc # (1), (2)

    echo "> 엔진엑스 Reload"
    sudo service nginx reload
}
```

* (1) `echo "set \$service_url http://127.0.0.1:${IDLE_PORT};`
  + 하나의 문장을 만들어 파이프라인(|)으로 넘겨주기 위해 echo를 사용합니다.
  + 엔진엑스가 변경할 프록시 주소를 생성합니다.
  + 쌍따옴표(")를 사용해야 합니다.
  + 사용하지 않으면 $service_url을 그대로 인식하지 못하고 변수를 찾게 됩니다.
* (2) `| sudo tee /etc/nginx/conf.d/service-url.inc`
  + 앞에서 넘겨준 문장을 service-url.inc에 덮어 씁니다.
* (3) `sudo service nginx reload`
  + 엔진엑스 설정을 다시 불러옵니다.
  + restart와는 다릅니다.
  + restart는 잠시 끊기는 현상이 있지만, reload는 끊김 없이 다시 불러옵니다.
  + 다만, 중요한 설정들은 반영되지 않으므로 restart를 사용해야 합니다.
  + 여기선 외부의 설정 파일인 service-url을 다시 불러오는 거라 reload로 가능합니다.

이렇게 5개의 스크립트를 모두 작성했습니다. 마지막으로 잦은 배포로 __Jar 파일명이 겹치지 않게 build.gradle에 코드를 수정__ 합니다.

```
version '1.0.1-SNAPSHOT-'+new Date().format("yyyyMMddHHmmss")
```

최종적으로 깃허브에 푸시하고 __CodeDeploy 로그로 잘 진행되는지 확인해 봅니다.__

```
# CodeDeploy 로그 확인
tail -f /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log

# 스프링 부트 로그 확인
vim ~/app/step3/nohup.out
```

# Related Posts
* [Chap 1.스프링 부트 시작하기](https://doorisopen.github.io/spring/2020/02/24/spring-freelec-springboot-chap1.html)
* [Chap 2.테스트 코드 작성하기](https://doorisopen.github.io/spring/2020/02/24/spring-freelec-springboot-chap2.html)
* [Chap 3.스프링 부트에서 JPA사용하기](https://doorisopen.github.io/spring/2020/02/26/spring-freelec-springboot-chap3.html)
* [Chap 4.Mustache로 화면 구성하기](https://doorisopen.github.io/spring/2020/03/03/spring-freelec-springboot-chap4.html)
* [Chap 5.스프링 시큐리티와 OAuth2.0](https://doorisopen.github.io/spring/2020/03/03/spring-freelec-springboot-chap5.html)
* [Chap 6.AWS EC2 서버 환경 구축하기](https://doorisopen.github.io/spring/2020/03/10/spring-freelec-springboot-chap6.html)
* [Chap 7.AWS RDS 생성하기](https://doorisopen.github.io/spring/2020/03/11/spring-freelec-springboot-chap7.html)
* [Chap 8.EC2 서버에 프로젝트 배포하기](https://doorisopen.github.io/spring/2020/03/12/spring-freelec-springboot-chap8.html)
* [Chap 9.Travis CI 배포 자동화](https://doorisopen.github.io/spring/2020/03/13/spring-freelec-springboot-chap9.html)
* [Chap 10.Nginx를 이용한 무중단 배포 - Now](#)

# References
* 스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)