---
layout: post
title:  "[Spring] 8 - Rest 아키텍처"
date:   2019-08-27 22:08:40
author: me
categories: Spring
tags: Spring Framework web MVC Controller
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

>> 해당 게시글의 전체 소스코드는 이곳 <a href=""><strong>Github</strong></a>를 참고해 주세요

<br />

### 목차
>> 1. __Rest 아키텍처 이해__
>> 2. __REST API 이해__
>> 3. __RESTful 이해__

<br />
<br />


<hr />

## REST(Representational State Transfer) 란?
* 자원을 __이름(자원의 표현)__ 으로 구분하여 해당 __자원의 상태(정보)__ 를 주고 받는 모든 것을 의미한다
  + 즉, 자원(resource)의 표현(representation) 에 의한 상태 전달
    - __자원(resource)의 표현(representation)__ <br/>
      __자원:__ 해당 소프트웨어가 관리하는 모든 것 <br/>
      -> Ex) 문서, 그림, 데이터, 해당 소프트웨어 자체 등 <br/>
      __자원의 표현:__ 그 자원을 표현하기 위한 이름 <br/>
      -> Ex) DB의 학생 정보가 자원일 때, 'students'를 자원의 표현으로 정한다 <br/>
    - __상태(정보) 전달__ <br/>
      데이터가 요청되어지는 시점에서 자원의 상태(정보)를 전달한다 <br/>
      __JSON__ 혹은 __XML를__ 통해 데이터를 주고 받는 것이 일반적이다 <br/>
<a href="{{ site.spring_img }}/spring_rest_3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_rest_3.JPG" title="Check out the image">
</a>   


* 클라이언트와 서버 사이에 __데이터 연동 애플리케이션을 위한__ 아키텍처 스타일
* __웹 상의 정보를 리소스로 파악하고 그 식별자로서 URI를 할당해 고유하게 특정함__
* HTTP 프로토콜을 사용해 리소스에 접근
  + __HTTP URI(Uniform Resource Identifier)를__ 통해 자원(Resource)을 명시하고, __HTTP Method(POST, GET, PUT, DELETE)를 통해__ 해당 자원에 대한 CRUD Operation을 적용하는 것을 의미한다
  + 즉, REST는 __자원 기반의 구조(ROA, Resource Oriented Architecture)__ 설계의 중심에 Resource가 있고 HTTP Method를 통해 Resource를 처리하도록 설계된 아키텍쳐를 의미한다

#### __HTTP Method__
<a href="{{ site.spring_img }}/spring_rest_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_rest_1.JPG" title="Check out the image">
</a>


#### __REST에 의한 리소스 접근 예시__
<a href="{{ site.spring_img }}/spring_rest_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_rest_2.JPG" title="Check out the image">
</a>


#### REST의 장단점
* __장점__
  + HTTP 프로토콜의 인프라를 그대로 사용하므로 REST API 사용을 위한 별도의 인프라를 구출할 필요가 없다.
  + HTTP 프로토콜의 표준을 최대한 활용하여 여러 추가적인 장점을 함께 가져갈 수 있게 해준다.
  + HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능하다.
  + Hypermedia API의 기본을 충실히 지키면서 범용성을 보장한다.
  + REST API 메시지가 의도하는 바를 명확하게 나타내므로 의도하는 바를 쉽게 파악할 수 있다.
  + 여러가지 서비스 디자인에서 생길 수 있는 문제를 최소화한다.
  + 서버와 클라이언트의 역할을 명확하게 분리한다.
* __단점__
  + 표준이 존재하지 않는다.
  + 사용할 수 있는 메소드가 4가지 밖에 없다.
  + HTTP Method 형태가 제한적이다.
  + 브라우저를 통해 테스트할 일이 많은 서비스라면 쉽게 고칠 수 있는 URL보다 Header 값이 왠지 더 어렵게 느껴진다.
  + 구형 브라우저가 아직 제대로 지원해주지 못하는 부분이 존재한다.
  + PUT, DELETE를 사용하지 못하는 점
  + pushState를 지원하지 않는 점


#### __REST가 필요한 이유__
* '애플리케이션 분리 및 통합'
* '다양한 클라이언트의 등장'
* 최근의 서버 프로그램은 다양한 브라우저와 안드로이폰, 아이폰과 같은 모바일 디바이스에서도 통신을 할 수 있어야 한다
* 이러한 멀티 플랫폼에 대한 지원을 위해 서비스 자원에 대한 아키텍처를 세우고 이용하는 방법을 모색한 결과, REST에 관심을 가지게 되었다


#### REST 구성 요소
* __자원(Resource): URI__
  + 모든 자원에 고유한 ID가 존재하고, 이 자원은 Server에 존재한다
  + 자원을 구별하는 ID는 '/groups/:group_id'와 같은 HTTP URI 다
  + Client는 URI를 이용해서 자원을 지정하고 해당 자원의 상태(정보)에 대한 조작을 Server에 요청한다
* __행위(Verb): HTTP Method__
  + HTTP 프로토콜의 Method를 사용한다
  + HTTP 프로토콜은 GET, POST, PUT, DELETE 와 같은 메서드를 제공한다
* __표현(Representation of Resource)__
  + Client가 자원의 상태(정보)에 대한 조작을 요청하면 Server는 이에 적절한 응답(Representation)을 보낸다
  + REST에서 하나의 자원은 JSON, XML, TEXT, RSS 등 여러 형태의 Representation으로 나타내어 질 수 있다
  + JSON 혹은 XML를 통해 데이터를 주고 받는 것이 일반적이다

<hr/>

#### REST 특징
* __Server-Client(서버-클라이언트 구조)__
  + 자원이 있는 쪽이 Server, 자원을 요청하는 쪽이 Client가 된다
    - REST Server: API를 제공하고 비즈니스 로직 처리 및 저장을 책임진다
    - Client: 사용자 인증이나 context(세션, 로그인 정보) 등을 직접 관리하고 책임진다
  + 서로 간 의존성이 줄어든다

* __Stateless(무상태)__
  + HTTP 프로토콜은 Stateless Protocol이므로 REST 역시 무상태성을 갖는다
  + Client의 context를 Server에 저장하지 않는다
    - 즉, 세션과 쿠키와 같은 context 정보를 신경쓰지 않아도 되므로 구현이 단순해진다
  + Server는 각각의 요청을 완전히 별개의 것으로 인식하고 처리한다
    - 각 API 서버는 Client의 요청만을 단순 처리한다
    - 즉, 이전 요청이 다음 요청의 처리에 연관되어서는 안된다
    - 물론 이전 요청이 DB를 수정하여 DB에 의해 바뀌는 것은 허용한다
    - Server의 처리 방식에 일관성을 부여하고 부담이 줄어들며, 서비스의 자유도가 높아진다

* __Cacheable(캐시 처리 가능)__
  + 웹 표준 HTTP 프로토콜을 그대로 사용하므로 웹에서 사용하는 기존의 인프라를 그대로 활용할 수 있다
    - 즉, HTTP가 가진 가장 강력한 특징 중 하나인 캐싱 기능을 적용할 수 있다
    - HTTP 프로토콜 표준에서 사용하는 Last-Modified 태그나 E-Tag를 이용하면 캐싱 구현이 가능하다
  + 대량의 요청을 효율적으로 처리하기 위해 캐시가 요구된다
  + 캐시 사용을 통해 응답시간이 빨라지고 REST Server 트랜잭션이 발생하지 않기 때문에 전체 응답시간, 성능, 서버의 자원 이용률을 향상시킬 수 있다

* __Layered System(계층화)__
  + Client는 REST API Server만 호출한다
  + REST Server는 다중 계층으로 구성될 수 있다
    - API Server는 순수 비즈니스 로직을 수행하고 그 앞단에 보안, 로드밸런싱, 암호화, 사용자 인증 등을 추가하여 구조상의 유연성을 줄 수 있다
    - 또한 로드밸런싱, 공유 캐시 등을 통해 확장성과 보안성을 향상시킬 수 있다
  + PROXY, 게이트웨이 같은 네트워크 기반의 중간 매체를 사용할 수 있다

* __Code-On-Demand(optional)__
  + Server로부터 스크립트를 받아서 Client에서 실행한다
  + 반드시 충족할 필요는 없다

* __Uniform Interface(인터페이스 일관성)__
  + URI로 지정한 Resource에 대한 조작을 통일되고 한정적인 인터페이스로 수행한다
  + HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능하다
    - 특정 언어나 기술에 종속되지 않는다

<hr/>

## REST 컨트롤러를 위한 라이브러리 설정 및 예제
* 리소스 형식을 __JSON(JavaScript Object Notation)__ 으로 사용
* __Jackson-databind__ 를 사용하면 JSON과 자바빈즈를 서로 교환할 수 있음 __(Pom.xml에 내용 추가 필요)__

* __Pom.xml__

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.9</version>
</dependency>
```

### REST 컨트롤러에 사용되는 애노테이션
* __@RestController__
  + __REST API를 제공하는 컨트롤러를 의미__
  + @Controller와 @ResponseBody를 의미를 합침
* __@RequestBody__
  + 컨트롤러 메서드 매개 변수에 @RequestBody가 애노테이션 된 경우, 스프링은 __요청된 HTTP request body를 해당 매개 변수에 바인딩__ 한다
* __@ResponseBody__
  + 컨트롤러 메소드가 @ResponseBody로 어노테이션 된 경우, Spring은 __반환 값을 나가는 HTTP response body에 바인딩한다__ 
  + 스프링은 요청된 메시지의 HTTP 헤더에 있는 Content-Type을 기반으로 HTTP Message converter를 사용하여 반환 값을 HTTP response body로 변환한다
* __ResponseEntity__
  + __전체 HTTP 응답을 나타내며__ statusCode, headers, body 3가지 속성 값을 지정할 수 있다

* __org.doorisopen.myspring.Test.Member.MemberRestTest 내용__

```
// Json GET
	@RequestMapping(value="/json/{id}", method = RequestMethod.GET)
	public ResponseEntity<MemberVO> MemberReadJson(@PathVariable String id) throws Exception {
		
		MemberVO vo = service.readMember(id);
		logger.info(" /member/rest/json/{id} REST-API GET method called. then method executed.");
		HttpHeaders headers = new HttpHeaders();
		headers.setContentType(new MediaType("application", "json", Charset.forName("UTF-8")));
		headers.set("My-Header", "MyHeaderValue");

		return new ResponseEntity<MemberVO>(vo, headers, HttpStatus.OK);
	}
```

* __명령 프롬프트(cmd) 결과__

```
Microsoft Windows [Version 10.0.17134.950]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\WINDOWS\system32>curl http://localhost:8080/myspring/member/rest/json/han
{"id":"han","passwd":"pwd","username":"kevin","snum":"190101","depart":"computer","mobile":"010-1111-2222","email":"a@gmail.com"}
```

<hr/>

## REST API
#### __REST API 예시__
<a href="{{ site.spring_img }}/spring_restapi_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_restapi_1.JPG" title="Check out the image">
</a>

#### __스프링MVC에서 REST API 구현하기__
<a href="{{ site.spring_img }}/spring_restapi_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_restapi_2.JPG" title="Check out the image">
</a>

<hr/>

## RESTful
#### RESTful 이란?
* RESTful은 일반적으로 REST라는 아키텍처를 구현하는 웹 서비스를 나타내기 위해 사용되는 용어이다. 
  + 'REST API'를 제공하는 웹 서비스를 'RESTful'하다고 할 수 있다.
* RESTful은 REST를 REST답게 쓰기 위한 방법으로, 누군가가 공식적으로 발표한 것이 아니다.
  + 즉, REST 원리를 따르는 시스템은 RESTful이란 용어로 지칭된다.
#### RESTful의 목적
* 이해하기 쉽고 사용하기 쉬운 REST API를 만드는 것
* RESTful한 API를 구현하는 근본적인 목적이 성능 향상에 있는 것이 아니라 일관적인 컨벤션을 통한 API의 이해도 및 호환성을 높이는 것이 주 동기이니, 성능이 중요한 상황에서는 굳이 RESTful한 API를 구현할 필요는 없다.
#### RESTful 하지 못한 경우
* __Ex1)__ CRUD 기능을 모두 POST로만 처리하는 API
* __Ex2)__ route에 resource, id 외의 정보가 들어가는 경우(/students/updateName)

<hr />

### References
* 대학교 웹 프레임워크 수업 정리한 내용 입니다.
> * <a href="https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html">https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html<a>