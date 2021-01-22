---
layout: post
title: "[가지마켓] #3 페이징 처리 및 배포 설정"
date: 2021-01-22 00:50:59
author: me
categories: Project
tags: gagi
cover: "/assets/instacode.png"
---


> 중고 거래 플랫폼 **가지마켓 프로젝트** 의 진행상황 및 이슈를 공유합니다.<br/>
> 프로젝트 저장소는 [이곳](https://github.com/GagiMarket/gagi)에 확인할 수 있습니다.

## 목차
* 상품 조회 페이징 처리
* 상품 검색 API 개발
  + API Spec
  + 문제점
* 상품 API 리펙토링(버전 추가)
* 회원 요구사항 분석 
  + 회원과 상품 
* 서버 구축 및 배포 설정
  + profiles 구분하기(local, real)
  + CORS 설정


## 상품 조회 페이징 처리
기존 상품 조회 API의 경우 아래와 같이 **등록된 상품 전체를 List 형태로 담아서 반환**하였습니다. 

```java
@GetMapping
public ResponseEntity<List<ItemResponseDto>> findItemList() {
    List<ItemResponseDto> responses = itemService.findItemList().stream()
            .map(ItemResponseDto::new)
            .collect(Collectors.toList());
    return ResponseEntity
                .ok()
                .body(responses);
```

그러나 등록된 상품의 수가 많아짐에 따라 API 호출시 페이지 로딩 속도가 느려지고(모바일의 경우 데이터 소비가 많아짐) DB에 부하가 걸리는 문제를 해결하기 위해서 페이징 처리를 하였습니다.

```java
@GetMapping
public ResponseEntity<Page<ItemResponseDto>> findItems(Pageable pageable) {
    Page<ItemResponseDto> findItems = itemService.findItems(pageable)
            .map(ItemResponseDto::new);
    return ResponseEntity
            .ok()
            .body(findItems);
}
```

조회시 다음 파라미터를 넣으면 API를 사용하는 사용자가 원하는데로 사용이 가능합니다.

|Name|Type|Description|default|Required|example|
|:---:|:---:|:---:|:---:|:---:|:---:|
|page|int|조회할 페이지 번호|0|false|/api/v1.0/items?page=0|
|size|int|한 페이지 당 조회 개수|20|false|/api/v1.0/items?size=2|
|sort|String|정렬 기준|asc|false|/api/v1.0/items?sort=itemId,desc|

다음 요청에 대한 Response는 다음과 같습니다

```http
GET http://localhost:8080/api/v1.0/items
200 OK
```

```json
{
    "content": [
        {
            "itemId": 1,
            "itemName": "에어팟 프로",
            "itemDescription": "애플 이어폰",
            "itemCategory": "전자기기",
            "itemPrice": 340000,
            "itemLocation": "가로수길",
            "registerDate": "2021-01-22T11:12:23.257554",
            "updateDate": "2021-01-22T11:12:23.257554"
        },
        {
            "itemId": 2,
            "itemName": "맥북 프로 16인치",
            "itemDescription": "애플 노트북",
            "itemCategory": "전자기기",
            "itemPrice": 3000000,
            "itemLocation": "강남",
            "registerDate": "2021-01-22T11:12:23.257554",
            "updateDate": "2021-01-22T11:12:23.257554"
        }
    ],
    "pageable": {

    },
    "totalPages": 1,
    "totalElements": 1,
    "last": true,
    "size": 20,
    "number": 0,
    "sort": {

    },
    "numberOfElements": 1,
    "first": true,
    "empty": false
}
```

그러나 필자가 사용한 페이징 방법은 `org.springframework.data.domain.Page` 이것인데 이것의 경우 **count 쿼리를 결과에 포함하는 페이징 방법**입니다. 그래서 조회시 count 쿼리가 강제적으로 실행이 되는 문제가 있습니다. 이는 데이터가 많아짐에 따라 전체 데이터를 count하는 쿼리는 성능상 무리를 줄 수 있고 반복되는 count 쿼리는 비효율적입니다. 이후에 count 쿼리를 분리하여 최적화 해보겠습니다.

## 상품 검색 API 개발
상품을 **이름으로 검색하는 요구사항이 추가**되어 상품 검색 API를 추가 구현하였습니다.

이름으로 검색할때 단건의 데이터가 아니라 리스트 형태로 조회되기 때문에 페이징 조건을 넣어서 조회가 가능합니다.

#### Request
```http
GET /api/v1.0/items/search
Host: localhost:8080
```

#### Parameter
|Name|Type|Description|Required|
|:---:|:---:|:---:|:---:|
|itemName|String|상품 이름|true|

#### Parameter(Pagination)
|Name|Type|Description|default|Required|example|
|:---:|:---:|:---:|:---:|:---:|:---:|
|page|int|조회할 페이지 번호|0|false|/api/v1.0/items?page=0|
|size|int|한 페이지 당 조회 개수|20|false|/api/v1.0/items?size=2|
|sort|String|정렬 기준|asc|false|/api/v1.0/items?sort=itemId,desc|

#### Example
```http
GET http://localhost:8080/api/v1.0/items/search?itemName=프로
```

#### Response
```http
200 OK
```

```json
{
    "content": [
        {
            "itemId": 1,
            "itemName": "에어팟 프로",
            "itemDescription": "애플 이어폰",
            "itemCategory": "전자기기",
            "itemPrice": 340000,
            "itemLocation": "가로수길",
            "registerDate": "2021-01-22T11:12:23.257554",
            "updateDate": "2021-01-22T11:12:23.257554"
        },
        {
            "itemId": 2,
            "itemName": "맥북 프로 16인치",
            "itemDescription": "애플 노트북",
            "itemCategory": "전자기기",
            "itemPrice": 3000000,
            "itemLocation": "강남",
            "registerDate": "2021-01-22T11:12:23.257554",
            "updateDate": "2021-01-22T11:12:23.257554"
        }
    ],
    "pageable": {

    },
    "totalPages": 1,
    "totalElements": 1,
    "last": true,
    "size": 20,
    "number": 0,
    "sort": {

    },
    "numberOfElements": 1,
    "first": true,
    "empty": false
}
```

#### 문제점

사실 검색 API에는 문제가 있습니다. 아래와 같이 **검색 조건과 페이징 처리가 섞여 버릴때 검색 조건을 구분하기가 어려움**이 있다는 것입니다.

```http
/api/v1.0/items?search=프로&page=0&size=5
```

이러한 경우에 API 설계에 따라서 Query 조건은 하나의 Query String 으로 정의하는 것이 좋습니다.

```http
/api/v1.0/items?q=itemName%3D프로&page=0&size=5
```

위와 같이 검색 조건을 URLEncode를 사용해서 "q=itemName%3D프로"(q=itemName=3프로) 처럼 표현하고 Deleminator를 , 등을 사용하게 되면 검색 조건은 다른 Query 스트링과 분리됩니다. (검색 조건은 서버에서 토큰 단위로 파싱합니다)

이 문제 또한 추후 리펙토링 과정을 통해 해결해보겠습니다.

> 상품 API에 대한 자세한 스펙은 [Wiki - 상품(item) API](https://github.com/GagiMarket/gagi/wiki/%EC%83%81%ED%92%88(item)-API)을 확인해 주세요

## 상품 API 리펙토링(버전 추가)
API 설계에 따라 API 버전을 추가 했습니다.

```http
http://localhost:8080/api/v1.0/items
```

이와 같은 형태로 관리하는 이유는 서비스의 배포 모델과 관련이 있습니다.

자바 애플리케이션의 경우, `{service-name}.v1.0.war` 와 같이 war로 배포하여 버전별로 배포 바이너리를 관리합니다.

앞단에 서비스 명을 별도로 URL로 떼어 놓는 것은 향후 **서비스가 확장되었을 경우에 해당 서비스만 별도의 서버로 분리해서 배포**하는 경우를 생각할 수 있기 떄문입니다.

외부로 제공되는 URL은 `api.server.com/{service-name}/v1.0/items`로 하나의 서버를 가르키지만, 내부적으로 reverse proxy를 이용해서 이런 URL을 맵핑할 수 있는데, `{service-name}.server.com/v1.0/items` 와 같이 매핑하도록 하면, 외부에 노출되는 URL 변경이 없이 향후 확장되었을때 서버를 물리적으로 분리하기 편리합니다.

## 회원 요구사항 분석
* 회원 등록을 할 수 있다.
  + 중복 회원을 검증할 수 있다.
* 회원은 로그인을 할 수 있다.
* 회원은 로그아웃을 할 수 있다.
* 회원은 회원정보를 수정할 수 있다.
* 회원은 회원탈퇴(삭제)를 할 수 있다.

#### 회원과 상품
기존 상품 도메인에 회원 도메인이 추가되는 것 이므로 회원과 상품간의 요구사항을 분석하였습니다.

* 회원은 여러개의 상품을 등록할 수 있다.
* 회원은 등록한 상품을 조회할 수 있다.
* 회원은 등록한 상품을 수정할 수 있다.
* 회원은 등록한 상품을 삭제할 수 있다.

## 서버 구축 및 배포 설정
서버 구축은 아래와 같이 구축하였습니다.

* AWS (EC2 - Amazon Linux 2, free tier)
  + JDK11
* AWS (RDS - MariaDB)

서버 구축이 완료되었다면 개발한 애플리케이션을 배포하기 전에 약간의 설정을 하였습니다.


#### profiles 구분하기
우선 local 과 real 환경을 구분하는 것입니다. 기존에 `src/main/resources/` 경로에 application.properties 파일을 `application.yml`로 확장자를 바꿉니다. 이유는 `.yml` 파일이 가독성이 더 좋고 local과 real 환경을 구분할때 개인적으로 간편하다고 느꼈기 때문입니다. 

```yml
# src/main/resources/application.yml
spring:
  profiles:
    active: local
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 100
logging:
  level:
    org.hibernamte.SQL: debug

# local
---
spring:
  profiles: local
  jpa:
    hibernate:
      ddl-auto: create-drop
  h2:
    console:
      enabled: true

# real
---
spring:
  profiles: real
  include: real-db 
```

위와 같이 `---` 구분선으로 설정을 구분할 수 있습니다. `.properties` 의 경우 `application-real.properties` 와 같이 파일을 생성하면 profiles가 real인 환경으로 구분할 수 있습니다.

그리고 `src/test/resources/` 테스트 코드의 경우에도 동일한 설정 파일(application.yml)이 필요한데 만약 설정 파일이 없다면 main의 application.yml 옵션을 그대로 가져와서 사용하게 됩니다. 따라서 테스트 코드가 외부의 영향 없이 수행되야 하기 떄문에 아래와 같이 작성하였습니다.

```yml
# src/test/resources/application.yml
# Test
spring:
  profiles:
    active: local
  jpa:
    properties:
      hibernate:
        dialect:

# Local
---
spring:
  profiles: local
  jpa:
    show-sql: true 
```

#### CORS 설정
CORS(Cross-origin resource sharing)는 교차 출처 리소스 공유 라는 의미로 웹 페이지 상의 제한된 리소스를 최초 자원이 **서비스된 도메인 밖의 다른 도메인으로부터 요청할 수 있게 허용**하는 구조를 의미합니다. 

개발한 애플리케이션을 서버에 배포하고 다른 도메인에서 배포한 애플리케이션의 API를 호출하게 되면 **No 'Access-Control-Allow-Origin' header is present on the request resource** 와 같은 에러 메시지를 보게되는게 이는 서로 다른 도메인에서 자원 요청이 거부되었다는 메시지를 확인할 수 있습니다. 

해당 문제를 해결하기 위해 서버쪽에서 아래와 같이 CORS 관련 설정을 통해서 문제를 해결할 수 있습니다.

```java
@CrossOrigin(origins = "*")
@RestController
@RequestMapping(ITEM_API_URI)
public class ItemApiController {
    ...이하 생략
}
```

* `@CrossOrigin(origins = "*")`
  + origins = "*" 은 모든 URL에 대해 허용한다는 의미입니다. 
  + 특정 URL에 대해서 다음과 같이 사용할 수 있습니다 origins = "https://localhost:8080"

## References
* [ec2 배포하기](https://jojoldu.tistory.com/263)
* [cors cross origin](https://popcorn16.tistory.com/154)
* [spring cors](https://spring.io/guides/gs/rest-service-cors/#global-cors-configuration)
* [cors wiki](https://ko.wikipedia.org/wiki/%EA%B5%90%EC%B0%A8_%EC%B6%9C%EC%B2%98_%EB%A6%AC%EC%86%8C%EC%8A%A4_%EA%B3%B5%EC%9C%A0)