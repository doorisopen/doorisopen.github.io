---
layout: post
title:  "[Spring Boot] Chap 2.테스트 코드 작성하기"
date:   2020-02-24 23:52:40
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
>> 1. TDD와 단위 테스트
>> 2. 테스트 코드 작성
>> 3. 자바 롬복(lombok)



<br />
<br />


<hr />


# TDD와 단위 테스트
## TDD(Test Driven Development)
`TDD`는 __테스트가 주도하는 개발__ 을 의미한다. TDD에는 3단계가 있다.
* __Red__ : 항상 실패하는 테스트를 먼저 작성한다.
* __Green__ : 테스트가 통과하는 프로덕션 코드를 작성한다.
* __Refactor__ : 테스트가 통과하면 프로덕션 코드를 리팩토링한다.

## 단위 테스트(Unit Test)
`단위 테스트`는 TDD의 첫 번째 단계인 __기능 단위의 테스트 코드를 작성__ 하는 것을 이야기 한다.

### 단위 테스트 코드 작성시 이점
* 단위 테스트는 개발단계 초기에 문제를 발견하게 도와준다.
* 단위 테스트는 개발자가 나중에 코드를 리팩토링하거나 라이브러리 업그레이드 등에서 기존 기능이 올바르게 작동하는지 확인할 수 있다.(예, 회귀테스트)
* 단위 테스트는 기능에 대한 불확실성을 감소시킬 수 있다.
* 단위 테스트는 시스템에 대한 실세 문서를 제공한다. 즉, 단위 테스트 자체가 문서로 사용할 수 있다.

# 테스트 코드 작성

Java 디렉토리 아래 __com.doop.book.springboot 패키지__ 를 생성하고 Application 클래스를 생성 후 아래와 같은 코드를 작성한다

```
package com.doop.book.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

코드를 다 작성했으면 __Application 클래스__ 는 프로젝트의 __메인 클래스__ 가 된다. 그리고 메인 클래스의 코드를 설명 하면

`@SpringBootApplication`의 선언으로 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정된다. 특히 __@SpringBootApplication이 있는 위치부터 설정을 읽어__ 가기 때문에 이 클래스는 항상 __프로젝트의 최상단에 위치해야만 한다.__

main 메소드에서 `SpringApplication.run(Application.class, args);`로 인해 내장 WAS를 실행한다.

### 내장 WAS를 사용하는 이유
__언제어디서나 같은 환경에서 스프링 부트를 배포 할 수 있기 때문이다.__

만약 외부 WAS를 사용한다면 모든 서버가 WAS의 종류와 버전, 설정을 동일하게 해줘야 한다. 서버의 개수가 적을 땐 문제 없지만 서버가 많을 때는 그만큼 많은 시간이 소요된다.

__com.doop.book.springboot 패키지__ 아래 __web__ 패키지를 생성하고 __HelloController 클래스__ 를 생성한다.

```
package com.doop.book.springboot.web;

import com.doop.book.springboot.web.dto.HelloResponseDto;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController// (1)
public class HelloController {
    @GetMapping("hello") // (2)
    public String hello() {
        return "hello";
    }
}
```

* (1) `@RestController`는 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 준다.
  + 예전에는 @ResponseBody를 각 메소드마다 선언했던 것을 한번에 사용할 수 있게 해준다고 생각하면 된다.
* (2) `@GetMapping` Http Method인 Get의 요청을 받을 수 있는 API를 만들어 준다.
  + 예전에는 `@RequestMapping(method = RequestMethod.GET)`으로 사용되었다. 이제는 이 프로젝트는 /hello로 요청이 오면 문자열 hello를 반환하는 기능을 가지게 되었다.

작성한 코드가 제대로 작동하는지 테스트 하기 위해 __src/test/java__ 디렉토리에 __com.doop.book.springboot.web__ 패키지를 생성하고 그 안에 __테스트 클래스__ 인 __HelloControllerTest.class__ 를 생성한다.

```
package com.doop.book.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class) // (1)
@WebMvcTest(controllers = HelloController.class) // (2)
public class HelloControllerTest {

    @Autowired // (3)
    private MockMvc mvc; // (4)

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello")) // (5)
                .andExpect(status().isOk()) // (6)
                .andExpect(content().string(hello)); // (7)
    }

}
```

* (1) `@RunWith(SpringRunner.class)`
  + 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다.
  + 여기서는 SpringRunner라는 스프링 실행자를 사용한다.
  + 즉, 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다.
* (2) `@WebMvcTest()`
  + 여러 스프링 테스트 어노테이션 중, Web(Spring MVC)에 집중할 수 있는 어노테이션 이다.
  + 선언할 경우 @Controller, @ControllerAdvice 등을 사용할 수 있다.
  + 단, @Service, @Component, @Repository 등은 사용할 수 없다.
  + 왜냐하면, 여기서는 컨트롤러만 사용하기 때문에 선언한다.
* (3) `@Autowired`
  + 스프링이 관리하는 빈(Bean)을 주입 받는다.
* (4) `MockMvc mvc;`
  + 웹 API를 테스트할 때 사용한다.
  + 스프링 MVC 테스트의 시작점이다.
  + 이 클래스를 통해 HTTP GET, POST 등에 대한 API 테스트를 할 수 있다.
* (5) `mvc.perform(get("/hello"))`
  + MockMvc를 통해 /hello 주소로 HTTP GET 요청을 한다.
  + 체이닝이 지원되어 아래와 같이 여러 검증 기능을 이어서 선언할 수 있다.
* (6) `.andExpect(status().isOk())`
  + mvc.perform의 결과를 검증한다.
  + HTTP Header의 Statu를 검증한다.
  + 상태는 200, 404, 500 등의 상태를 검증한다.
  + 여기선 OK 즉, 200인지 아닌지 검증한다.
* (7) `.andExpect(content().string(hello));`
  + mvc.perform의 결과를 검증한다.
  + 응답 본문의 내용을 검증한다.
  + Controller에서 "hello"를 리턴하기 때문에 이 값이 맞는지 검증한다.


# 자바 롬복(lombok)
__롬복(lombok)__ 은 자바 개발할 때 자주 사용하는 코드 Getter, Setter, 기본 생성자, toString 등을 어노테이션으로 자동 생성해주는 라이브러리이다. (자바 개발자들의 필수 라이브러리이다.)

이클립스의 경우엔 롬복 설치가 번거롭지만, 인텔리제이에선 플러그인 덕분에 쉽게 설정이 가능하다.

우선 롬복을 사용하기 위해서

1. __build.gradle__ 에서 __dependencies영역__ 에 `compile('org.projectlombok:lombok')`를 추가한다.

```
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

2. 롬복 플러그인 설치를 위해 __Action(Ctrl + Shift + A) -> plugins 입력__ 하여 __플러그인 설치 팝업__ 을 띄운다.

3. marketplace -> lombok을 검색하고 install 한다.

4. intelliJ 재 시작

5. 재 시작하면 __Event log에 롬복에 대한 설정이 필요하다__ 라는 팝업이 뜨고 __설정 경로(Setting > build > Compiler > Annotation Processors)를 파란색으로 표기__ 해준다. 

6. 경로로 들어가 __Enable annotation processing__ 을 체크 한다.

## Hello Controller 코드 롬복으로 전환하기
web 패키지에 __dto 패키지를 추가__ 한다. 그리고 dto 패키지에 __HelloResponseDto를 생성__ 한다.

```
package com.doop.book.springboot.web.dto;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter // (1)
@RequiredArgsConstructor // (2)
public class HelloResponseDto {
    private final  String name;
    private final int amount;

}
```

* (1) `@Getter`는 선언된 모든 필드의 get 메소드를 생성해 준다.
* (2) `@RequiredArgsConstructor`
  + 선언된 모든 __final 필드가 포함된 생성자를 생성__ 해 준다.
  + final이 없는 필드는 생성자에 포함되지 않는다.

그리고 Dto에 적용된 롬복이 잘 작동하는지 테스트 하기 위해 __src/test/java__ 디렉토리에서 __com.doop.book.springboot.web__ 패키지 아래 __dto 패키지를 생성__ 하고 __HelloResponseDtoTest.class를 생성__ 한다.

```
package com.doop.book.springboot.web.dto;

import org.junit.Test;
import static org.assertj.core.api.Assertions.assertThat;

public class HelloResponseDtoTest {

    @Test
    public void 롬복_기능_테스트() {
        // given
        String name = "test";
        int amount = 1000;

        // when
        HelloResponseDto dto = new HelloResponseDto(name, amount);

        // then
        assertThat(dto.getName()).isEqualTo(name); // (1), (2)
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
}
```

* (1) `assertThat` (JUnit이 아닌 __assertj의 assertThat을 사용__ 이유는 아래)
  + assertj라는 테스트 검증 라이브러리의 검증 메소드이다.
  + 검증하고 싶은 대상을 메소드 인자로 받는다.
  + 메소드 체이닝이 지원되어 isEqualTo와 같이 메소드를 이어서 사용할 수 있다.
* (2) `isEqualTo`
  + assertj의 동등 비교 메소드이다.
  + assertThat에 있는 값과 isEqualTo의 값을 비교해서 같을 때만 성공한다.


### JUnit과 assertj의 장점
* assertj는 CoreMatchers와 달리 추가적으로 라이브러리가 필요하지 않는다.
  + JUnit의 assertThat을 쓰게 되면 is()와 같이 CoreMatchers 라이브러리가 필요하다.
* 자동완성이 좀 더 확실하게 지원된다.
  + IDE에서는 CoreMatchers와 같은 Matcher 라이브러리의 자동완성 지원이 약하다.


테스트 결과가 성공했다면, __HelloController에 새로 만든 ResponseDto를 사용하도록 코드를 추가__ 한다.

```
package com.doop.book.springboot.web;

import com.doop.book.springboot.web.dto.HelloResponseDto;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController// 1
public class HelloController {
    @GetMapping("hello") // 2
    public String hello() {
        return "hello";
    }

    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name, // (1)
                                     @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount);
    }
}
```

* (1) `@RequestParam()`
  + 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션이다.
  + 여기서는 외부에서 name(@RequestParam("name")) 이란 이름으로 넘긴 파라미터를 메소드 파라미터 name(String name)에 저장하게 된다.

HelloController에 코드를 추가 하고 API를 테스트 하는 코드를 __src/test/java__ 디렉토리의 __HelloControllerTest에 추가__ 한다.

```
package com.doop.book.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }

    @Test
    public void helloDto가_리턴된다() throws Exception {
        String name = "hello";
        int amount = 1000;

        mvc.perform(
                get("/hello/dto")
                    .param("name", name) // ...(1)
                    .param("amount", String.valueOf(amount)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is(name))) // ...(2)
                .andExpect(jsonPath("$.amount", is(amount)));
    }
}
```

* (1) `param`
  + API 테스트할 때 사용될 요청 파라미터를 설정한다.
  + 단, 값은 String만 허용된다.
  + 그래서 숫자/날짜 등의 데이터도 등록할 때는 문자열로 변경해야만 가능하다.
* (2) `jsonPath`
  + JSON 응답값을 필드별로 검증할 수 있는 메소드이다.
  + $를 기준으로 필드명을 명시한다.
  + 여기서는 name과 amount를 검증하니 $.name, $.amount로 검증한다.

# IntelliJ 단축키
* __패키지 가져오기__
  + Win/Linux: Alt + Enter
  + Mac: Option + Enter

# Related Posts
* [Chap 1.스프링 부트 시작하기](https://doorisopen.github.io/spring/2020/02/24/spring-freelec-springboot-chap1.html)
* [Chap 2.테스트 코드 작성하기- Now](#)
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
