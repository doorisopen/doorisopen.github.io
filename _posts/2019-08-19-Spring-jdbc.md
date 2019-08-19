---
layout: post
title:  "[Spring] 4 - JDBC"
date:   2019-08-19 10:40:59
author: me
categories: Spring
tags: Spring Framework web JDBC JUnit
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

>> 해당 게시글의 전체 소스코드는 이곳 <a href="https://github.com/doorisopen/SpringSpring/tree/277d82107a895c9f636da9ff2b56344129ff19dd"><strong>Github</strong></a>를 참고해 주세요

<br />

### 목차
>> 1. __데이터 액세스 층에 대해 이해__
>> 2. __스프링 JDBC를 이해__
>> 3. __JUnit 을 이용한 스프링 유닛 테스트를 이해__

<br />
<br />


<hr />


## 데이터 액세스 층
### 데이터 액세스 층의 역할
* __데이터 액세스 처리를 비즈니스 로직 층에서 분리__ 하는 것

### DAO(Data Access Object)
* __데이터 액세스 처리에 특화된 오브젝트__
* Sun microsystems (현재 Oracle)가 제창한 J2EE 패턴 중 하나인 DAP 패턴 용어

#### DAO(Data Access Object) 패턴
* DAO 패턴은 __데이터 취득과 변경에 데이터 처리를 DAO 오브젝트로 분리하는 패턴__


#### 자바의 데이터 액세스 기술
* JDBC, Hybernate, MyBatis, JPA
<a href="{{ site.spring_img }}/spring_jdbc_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_jdbc_1.JPG" title="Check out the image">
</a>

### 데이터 소스(Data Source)
* 데이터 액세스 기술 종류와 상관없이 __데이터베이스 접속을 관리해주는 인터페이스__
* 업무용 어플리케이션은 커넥션 풀에 의해 커넥션 오브젝트를 재사용
<a href="{{ site.spring_img }}/spring_jdbc_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_jdbc_2.JPG" title="Check out the image">
</a>

#### 데이터 소스 구현
* Pom.xml 라이브러리 추가
<a href="{{ site.spring_img }}/spring_jdbc_3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_jdbc_3.JPG" title="Check out the image">
</a>

* 서드 파티가 제공하는 데이터 소스
  + __Apache Commons DBCP__
  + 의존 관계 설정
<a href="{{ site.spring_img }}/spring_jdbc_4.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_jdbc_4.JPG" title="Check out the image">
</a>


<hr />


## JDBC
### JDBC 이용의 문제점
* __대량의 소스 코드__ 를 기술
* 다양한 __에러 원인을 파악하기 위한 코딩__ 필요
* 데이터베이스 제품마다 에러 코드가 달라서 코드의 일관성 유지가 어려움

## 스프링 JDBC
* __JDBC를 래핑한 API를 제공해 소스 코드를 단순화__
* JDBC를 직접 사용할 때 발생하는 __장황한 코드를 은닉__
  + 커넥션 연결 종료
  + SQL 문의 실행
  + SQL 문 실행 결과 행에 대한 반복 처리
  + 예외 처리
* 스프링 JDBC기 제공하는 중요 탬플릿
  + __JdbcTemplate__
  + NamedParameterTemplate

#### JdbcTemplate 클래스 제공 메서드
<a href="{{ site.spring_img }}/spring_jdbc_5.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_jdbc_5.JPG" title="Check out the image">
</a>

#### JdbcTemplate 클래스의 오브젝트 생성과 인젝션
  + 탬플릿 클래스를 XML 파일에 Bean으로 정의
<a href="{{ site.spring_img }}/spring_jdbc_6.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_jdbc_6.JPG" title="Check out the image">
</a>

### 스프링 JDBC - SELECT 

#### 취득 결과가 __레코드 건수 또는 특정 컬럼만 취득할 경우__
* __queryForObject 메서드 사용 예제 1__
  + __제1인수__ : SQL 문자열
  + __제2인수__ : 반환형 클래스 오브젝트 (int)

```
JdbcTemplate jdbcTemplate = ctx.getBean(JdbcTemplate.class);
int count = jdbcTemplate.queryForObject("SELECT COUNT(*) FROM STUDENT", Integer.class);
```

* __queryForObject 메서드 사용 예제 2__
  + __제1인수__ : SQL 문자열, 제2인수 : 반환형 클래스 오브젝트(String)
  + __제3인수__ : 파라미터 값

```
String name = jdbcTemplate.queryForObject("SELECT username FROM STUDENT WHERE id= ?", String.class, id);
```

#### 취득 결과가 한 레코드 값을 취득할 경우

* __queryForMap 메서드__ 
  + 한 레코드 값을 Map(컬럼 이름을 키로 값을 저장) 데이터로 저장

```
Map<String, Object> student = jdbcTemplate.queryForMap("SELECT * FROM STUDENT WHERE id= ?", id);
String name = (String)student.get("username"); 
```

* __queryForList 메서드__ 
  + 여러 레코드 값을 Map 데이터로 저장

```
List<Map<String, Object>> studentList = jdbcTemplate.queryForMap("SELECT * FROM STUDENT ");
```

#### 도메인으로 변환할 경우
* __queryForObject 메서드__ 와 __query 메서드__ 를 이용한다
* __queryForObject 메서드:__ 한 레코드를 가져올 때
  + __제1인수__ : SELECT 문
  + __제2인수__ : 도메인 자동 변환을 위한 스프링 제공 클래스 BeanPropertyRowMapper
  + __제3인수__ : SELECT 문의 파라미터
  + BeanPropertyRowMapper를 사용할 경우 StudentVO의 프로퍼티 명과 테이블 컬럼 명이 같아야 함. 그렇지 않을 경우는 RowMapper 인터페이스를 구현해서 StudentVO로 변환 처리

```
public StudentVO read(String id) throws Exception {
    StudentVO vo = null;
    try {
        vo = jdbcTemplate.queryForObject("SELECT * FROM STUDENT WHERE ID=?", new BeanPropertyRowMapper<StudentVO>(StudentVO.class), id);
    }
    catch(EmptyResultDataAccessException e) {
        return vo;
    }
    return vo;
}
```

* __query 메서드__ : 여러 레코드를 가져올 때
  + RowMapper 인터페이스를 구현한 익명 클래스를 정의
  + 클래스내 mapRow() 추상 메서드를 정의

```
public List<StudentVO> readList() throws Exception {
    List<StudentVO> studentlist = jdbcTemplate.query("SELECT * FROM STUDENT",
        new RowMapper<StudentVO>() {
            public StudentVO mapRow(ResultSet rs, int rowNum) throws SQLException {
                StudentVO vo = new StudentVO();
                vo.setId(rs.getString("ID"));
                vo.setPasswd(rs.getString("PASSWD"));
                vo.setUsername(rs.getString("USERNAME"));
                vo.setSnum(rs.getString("SNUM"));
                vo.setDepart(rs.getString("DEPART"));
                vo.setMobile(rs.getString("MOBILE"));
                vo.setEmail(rs.getString("EMAIL"));
                return vo;
            }
        }
    );
    return studentlist;
}
```

### 스프링 JDBC - INSERT, UPDATE, DELETE
* __update 메서드__ 만을 사용

#### INSERT

```
StudentVO vo;
jdbcTemplate.update("INSERT INTO STUDENT (ID, PASSWD, USERNAME, SNUM, DEPART, MOBILE, EMAIL)VALUES (?, ?, ?, ?, ?, ?, ?) " , vo.getId(), vo.getPasswd(), vo.getUsername(), vo.getSnum(), vo.getDepart(), vo.getMobile(), vo.getEmail());
```

#### DELETE

```
StudentVO vo;
jdbcTemplate.update( "DELETE FROM STUDENT WHERE ID=? ", vo.getId() );
```

<hr />

## JUnit을 이용한 스프링 유닛 테스트

### __스프링 테스트(Spring Test)__
* __스프링 프레임워크에서 만든 클래스__(@Controller, @Service, @Repository,@Component 등이 붙은 클래스)를 테스트하는 모듈
* __단위 테스트, 통합 테스트를 지원하기 위한 매커니즘이나 편리한 기능을 제공__
  + Junit 테스트 프레임워크를 사용하여 스프링 DI 컨테이너를 동작시키는 기능
  + 트랜잭션 테스트를 상황에 맞게 제어하는 기능
  + 애플리케이션 서버를 사용하지 않고 스프링 MVC 동작을 재현하는 기능
  + RestTemplate을 이용해 HTTP 요청에 대한 임의 응답을 보내는 기능

#### 스프링 테스트와 JUnit 테스팅 프레임워크 설정
* __Pom.xml__

```

<org.spring-framework.version>5.1.3.RELEASE</spring-framework.version>
<junit.version>4.12</junit.version>

...(중략)...

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>${org.spring-framework.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>${junit.version}</version>
    <scope>test</scope>
</dependency>

```

* __JUnit Test__

```
package org.doorisopen.myspring.JUnitTest;

import org.doorisopen.myspring.Member.Domain.MemberVO;
import org.doorisopen.myspring.Member.Service.MemberService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;


@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:/Testcontext.xml")
public class JUnitMember {
	
	
	@Autowired
	MemberService memberService;
	
	@Test
	public void testReadMember( ) throws Exception {
		MemberVO member = memberService.readMember("Anonymous");
		System.out.println(member);
	}
}
```

> 만약!! 아래의 클래스를 import할 수 없는 문제가 발생한다면 <br />
> import org.springframework.test.context.ContextConfiguration; <br />
> import org.springframework.test.context.junit4.SpringJUnit4ClassRunner; <br />
>> __프로젝트 오른쪽 클릭 -> Maven -> Project Update__ 를 해주면 된다.

