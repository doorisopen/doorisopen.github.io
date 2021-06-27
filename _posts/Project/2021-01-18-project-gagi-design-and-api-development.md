---
layout: post
title: "[가지마켓] #2 프로젝트 설계 및 API 개발"
date: 2021-01-18 00:50:59
author: me
categories: Project
tags: gagi
cover: "/assets/instacode.png"
---

> 중고 거래 플랫폼 **가지마켓 프로젝트** 의 진행상황 및 이슈를 공유합니다.<br/>
> 프로젝트 저장소는 [이곳](https://github.com/GagiMarket/gagi)에 확인할 수 있습니다.

## 목차
* 프로젝트 설계(도메인 모델)
* API 설계 방법
* 상품 API 구현
* API 문서화 툴 Swagger 적용하기

## 프로젝트 설계(도메인 모델)
프로젝트를 시작하면서 최종 구현 목표를 두루뭉실하게 설정하였습니다. 이 요구사항을 보면서 어떻게 프로젝트를 한 눈에 파악하기 좋을까 고민하였고, 요구사항이 변경되어도 변화의 가능성이 적은 도메인 모듈들을 도출하고 **도메인 모델**로 표현해보았습니다.

<a href="{{ site.2021_project_img }}/project-gagi-domain-model.png" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2021_project_img }}/project-gagi-domain-model.png" title="Check out the image">
</a>

* **회원과 상품의 관계**: 회원은 여러개의 상품(중고 물품)을 등록할 수 있다.(1:N)
* **회원과 주문의 관계**: 회원은 하나의 상품만 주문할 수 있다.(중고 거래는 1:1 거래를 원칙으로 한다)(1:1)
* **회원과 회원등급의 관계**: 회원은 하나의 등급을 가진다.(1:1)
* **상품과 주문의 관계**: 하나의 상품은 하나의 주문이 발생한다.(1:1)
* **주문과 (할인, 배송, 결제)의 관계**: 하나의 주문에는 할인, 배송, 결제가 한번 발생한다.(1:1)
* **상품과 카테고리의 관계**: 하나의 상품은 하나의 카테고리를 갖는다.(1:1)
* **상품과 (파일, 찜, 태그, 댓글)의 관계**: 하나의 상품은 다수의 파일(이미지), 찜(좋아요), 태그, 댓글(대댓글)을 가질 수 있다.(1:N)

## API 설계 방법
API 개발에 앞서 설계 방법에 대해서 학습하였습니다. 

API 개발이라고 하면 REST API를 생각할 수 있는데 단순히 HTTP Method(GET,POST,PUT,DELETE)를 제공했다고 해서 REST API를 개발했다고 한다면 그것은 잘못된 이해입니다. 그러나 최대한 REST에 대한 속성을 적용해서 RESTFul 한 API를 개발해볼 것 입니다. 아래는 최소한 준수해볼(?) API 설계 규칙을 요약한 것입니다.

* HTTP Method(GET,POST,PUT,DELETE)를 제공한다.
* URI을 보고 직관적으로 이해할 수 있어야한다.
  + 최대 2 depth 정도로 간단하게 만든다.
  + 리소스명은 동사보다는 명사를 사용한다.
    - 리소스명은 소문자를 사용한다.
  + underbar(_) 대신 dash(-)를 사용한다.
  + 마지막에 / 포함하지 않는다.
* 리소스간의 관계를 표현하는 방법.
  + 서브 리소스로 표현하기
  + 서브 리소스에 관계를 명시하기 
* 에러 처리
* 페이징 처리
* 검색
* API 버전 관리

> API 설계 방법에 대한 자세한 내용은 [이곳](https://doorisopen.github.io/developers-library/Web/2020-06-04-web-rest-api-guide) 을 참고해주세요.

## 상품 API 구현
1주차 목표로 **상품 API 구현**하는 목표를 세웠는데, 처음 부터 완벽하게 구현하기 보단 처음엔 틀을 잡고 점차적으로 리펙토링 과정을 통해 완성도를 높여볼 생각입니다. (해당 글에서는 API Spec에 대해서만 소개하고 추후 다른 게시글에서 개발한 코드에 대한 설명을 정리할 예정입니다.)

|HTTP Method|URI|Description|
|:---:|:---:|:---:|
|POST|/api/v1.0/items|상품을 등록한다.|
|GET|/api/v1.0/items|상품 전체를 조회한다.|
|GET|/api/v1.0/items/{itemId}|{itemId} 상품을 조회한다.|
|GET|/api/v1.0/items/search|상품을 검색한다.(상품 이름으로)|
|PUT|/api/v1.0/items/{itemId}|{itemId} 상품을 수정한다.|
|DELETE|/api/v1.0/items/{itemId}|{itemId} 상품을 삭제한다.|


> 상품 API에 대한 자세한 스펙은 [Wiki - 상품(item) API](https://github.com/GagiMarket/gagi/wiki/%EC%83%81%ED%92%88(item)-API)을 확인해 주세요

## API 문서화 툴 Swagger 적용하기
Swagger는 개발자가 REST 웹서비스를 설계, 빌드, 문서화 하는 일을 도와주는 오픈소스 입니다.

Swagger를 사용하는 이유는 API Spec 이 변경 혹은 추가될때 마다 API 문서의 내용을 변경해야하는 번거로운 작업을 하지 않고 API Spec을 문서화를 자동화하기 위해서 적용해 보았습니다.

우선 Spring Boot & Gradle 프로젝트에 Swagger를 적용하기 위해서 `build.gradle`에 해당 라이브러리를 추가해줍니다.

```js
compile group: 'io.springfox', name: 'springfox-swagger2', version: '2.9.2'
compile group: 'io.springfox', name: 'springfox-swagger-ui', version: '2.9.2'
```

그 다음 Swagger 설정을 위한 설정 파일을 만들고 다음과 같이 Bean 등록을 해줍니다.

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any()) // RequestMapping 에 할당된 모든 URL 목록 추출
                .paths(PathSelectors.ant("/api/**")) // "/api/**" 인 URL 들만 필터링
                .build();
    }
}
```

그리고 Swagger 설정에 대한 내용을 `localhost:8080/swagger-ui.html` 주소에서 확인할 수 있습니다.


## References
* [SpringBoot에 Swagger 적용하기](https://jojoldu.tistory.com/31)
* [Spring Guide: baeldung.com](https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)