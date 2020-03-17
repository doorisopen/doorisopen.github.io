---
layout: post
title:  "[Spring Boot] Chap 7.AWS RDS 생성하기"
date:   2020-03-11 00:30:00
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
>> 1. RDS 인스턴스 생성
>> 2. RDS 운영황경에 맞는 파라미터 설정하기
>> 3. RDS 접속하기


<br />
<br />


<hr />

__RDS(Relational DataBase Service)__ 는 AWS에서 지원하는 클라우드 기반 관계형 데이터베이스로 모니터링, 알람, 백업, HA 구성 등을 지원하는 __관리형 서비스__ 입니다.
또한, 하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같은 잦은 운영 작업을 자동화하여 개발자가 개발에 집중할 수 있게 지원합니다.

# RDS 인스턴스 생성
__AWS 접속 > 서비스 > RDS > 데이터베이스 생성__

* __데이터베이스 생성 방식 선택:__ 표준 생성
* __엔진 옵션__
  + __엔진 유형:__ MariaDB 
  + __버전:__ 10.2.21
* __템플릿:__ 프리 티어
* __설정__ 
  + 본인이 원하는 식별자, 자격 증명 설정(마스터 사용자 이름) 작성
  + __암호는 추후 RDS 접속시 필요한 정보__ 입니다.
* __DB 인스턴스 크기:__ 버스터블 클래스 - db.t2.micro
* __스토리지__
  + __유형:__ 범용(SSD)
  + __할당된 스토리지:__ 20 GiB
* __연결 > 추가 연결 구성__
  + __퍼블릭 엑세스 가능:__ 예
  + __데이터베이스 포트:__ 3306
* __추가 구성__
  + 초기 데이터베이스 이름 작성
  + __DB 파라미터 그룹:__ default.mariadb 10.2
  + __옵션 그룹:__ default:mariadb-10-2

위 내용을 작성하고 RDS 인스턴스를 생성을 클릭한다.

# RDS 운영황경에 맞는 파라미터 설정하기
RDS를 처음 생성하면 몇 가지 설정을 필수로 해야 합니다.

* 타임존
* Character Set
* Max Connection


__RDS > 파라미터 그룹 > 파라미터 그룹 생성 > 파라미터 편집__

`time_zone` 검색 > Asia/Seoul 선택

`character` 검색 > __character 항목은 utf8mb4__ 로 __collation항목은 utf8mb4_general_ci__ 로 변경

* character_set_client - utf8mb4
* character_set_connection - utf8mb4
* character_set_database - utf8mb4
* character_set_filesystem - utf8mb4
* character_set_results - utf8mb4
* collation_connection - utf8mb4_general_ci
* collation_server - utf8mb4_general_ci

### utf8과 utf8mb4 차이
이모지 저장 가능 여부 입니다. utf8은 이모지를 저장할 수 없지만, utf8mb4는 이모지를 저장할 수 있어 보편적으로 utf8mb4를 많이 사용합니다.

`max_connections` 검색 > 150 으로 변경

RDS의 Max Connection은 __인스턴스 사양에 따라 자동__ 으로 정해집니다. 설정을 저장하고 생성된 파라미터 그룹을 데이터베이스에 연결하겠습니다.

__데이터베이스 > DB 식별자 선택 > 선택 > DB 파라미터 그룹 선택 > 수정 사항 요약: 즉시 적용__

__즉시 적용__ 하는 이유는 변경된 내용이 예약된 다음 유지 시간으로 하면 지금 하지 않고, 새벽 시간대에 진행하게 됩니다. 이 수정사항이 반영되는 동안 데이터베이스가 작동하지 않을 수 있으므로 __예약 시간을 걸어__ 두라는 의미입니다. 


# RDS 접속하기
RDS에 접속하기 위해서 __RDS의 보안 그룹에 본인 PC IP를 추가__ 해야합니다. 

__RDS 인스턴스 선택 > 보안 그룹 선택__ 아래 내용 추가

|  <center>유형</center> |  <center>프로토콜</center> |  <center>포트 범위</center> |  <center>소스</center> |
|:--------:|:--------:|:--------:|:--------:|
| MySQL/Aurora | TCP | 3306 | 내 IP | 
| MySQL/Aurora | TCP | 3306 | 사용자 지정: EC2 보안 그룹 ID |

이렇게 하면 __EC2와 RDS 간에 접근이 가능__ 합니다. PC에서 RDS에 접속하기 위한 방법에는 여러 방법이있습니다. 대표적으로 두 가지는 아래와 같습니다.

1. __IntelliJ에 플러그인 설치 후 접근__
2. __Putty EC2 서버에 MySQL를 설치하여 콘솔에서 접근__

위의 방법이 아니여도 Workbench 등 으로 접속할 수 있습니다.

RDS 데이터베이스에 접속 하였다면 현재의 __character_set, collation 설정__ 을 확인합니다.

`show variables like '%c';` 

쿼리 결과를 보면 다른 필드들은 모두 utf8mb4가 잘 적용되었는데 일부가 변경 안됨을 볼 수 있습니다.

```
ALTER DATABASE 데이터베이스명
CHARACTER SET = 'utf8mb4'
COLLATE = 'utf8mb4_general_ci';
```

위의 쿼리를 입력하여 변경해줍니다. 

이것으로 RDS 인스턴스 생성과 환경 설정은 끝이 났습니다. 다음 게시글은 EC2 서버에 프로젝트를 배포하는 방법을 설명하겠습니다.

# Related Posts
* [Chap 1.스프링 부트 시작하기](https://doorisopen.github.io/spring/2020/02/24/spring-freelec-springboot-chap1.html)
* [Chap 2.테스트 코드 작성하기](https://doorisopen.github.io/spring/2020/02/24/spring-freelec-springboot-chap2.html)
* [Chap 3.스프링 부트에서 JPA사용하기](https://doorisopen.github.io/spring/2020/02/26/spring-freelec-springboot-chap3.html)
* [Chap 4.Mustache로 화면 구성하기](https://doorisopen.github.io/spring/2020/03/03/spring-freelec-springboot-chap4.html)
* [Chap 5.스프링 시큐리티와 OAuth2.0](https://doorisopen.github.io/spring/2020/03/03/spring-freelec-springboot-chap5.html)
* [Chap 6.AWS EC2 서버 환경 구축하기](https://doorisopen.github.io/spring/2020/03/10/spring-freelec-springboot-chap6.html)
* [Chap 7.AWS RDS 생성하기 - Now](#)
* [Chap 8.EC2 서버에 프로젝트 배포하기](https://doorisopen.github.io/spring/2020/03/12/spring-freelec-springboot-chap8.html)
* [Chap 9.Travis CI 배포 자동화](https://doorisopen.github.io/spring/2020/03/13/spring-freelec-springboot-chap9.html)
* [Chap 10.Nginx를 이용한 무중단 배포](https://doorisopen.github.io/spring/2020/03/18/spring-freelec-springboot-chap10.html)

# References
* 스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)