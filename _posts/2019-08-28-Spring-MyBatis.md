---
layout: post
title:  "[Spring] 10 - MyBatis"
date:   2019-08-28 22:55:40
author: me
categories: Spring
tags: Spring Framework web MyBatis
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

>> 해당 게시글의 전체 소스코드는 이곳 <a href="https://github.com/doorisopen/SpringSpring/tree/1dd8527099f303ac3cb42e088f704a5886119e04"><strong>Github</strong></a>를 참고해 주세요

<br />

### 목차
>> 1. __MyBatis의 기본 개념을 이해__
>> 2. __스프링과 MyBatis 연계 구조 이해__
>> 3. __Spring Test 모듈 이해__

<br />
<br />


<hr />


### MyBatis
* __SQL과 자바 객체를 매핑하는 사상에서 개발된 데이터베이스 접근용 프레임워크__
* SQL 기반으로 데이터베이스 접근을 수행하는 기존 방식과 큰 규모의 애플리케이션 개발에서 발생하는 과제를 해결하는 구조

* __장점__
  + __SQL의 체계적인 관리__(설정 파일, 애노테이션)
    - 자바 Mapper 인터페이스를 통해 SQL 설정 파일과 연동
    - 비즈니스 로직에서 Mapper 인터페이스를 통해 SQL문 실행
  + __자바 객체와 SQL 입출력 값의 바인딩__
  + 동적 SQL 조합

* __주요 컴포넌트__
  + MyBatis 라이브러리
    - __MyBatis 설정 파일__
    - __SqlSessionFactoryBuilder :__ MyBatis 설정 파일을 바탕으로 SqlSessionFactory를 생성
    - __SqlSessionFactory :__ sqlSession 생성을 위한 컴포넌트
    - __SqlSession :__ SQL 발행과 트랜잭션 관리
    - __Mapper 인터페이스 :__ 매핑 파일과 SQL에 대응하는 자바 인터페이스
    - __매핑 파일 :__ SQL과 OR 매핑을 XML에 파일 기술

  + 스프링 프레임워크 연동을 위한 __MyBatis-Spring__ 라이브러리
    - __org.mybatis.spring.SqlSessionTemplate :__ sqlSession 인터페이스를 구현

* MyBatis의 핵심 API는 __sqlSession 인터페이스__ 이다
  + __sqlSession을 사용하는 방법__
    - __sqlSession 객체를 DAO 객체에 의존관계 주입으로 사용__
    - DAO 역할을 Mapper 객체를 통해 기능 제공
      - myBatis는 DAO의 구현을 자동으로 생성해주는 Mapper라는 기능을 제공
      - 개발자가 Mapper 인터페이스를 준비해서 Mapper 객체를 생성하도록 빈 정의
<a href="{{ site.spring_img }}/spring_mybatis_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_mybatis_1.JPG" title="Check out the image">
</a>

* __MyBatis 연동을 위한 라이브러리(pom.xml 추가 필요)__
  + __spring-jdbc :__ 스프링이 제공하는 JDBC 래핑 모듈
  + __MyBatis-Spring :__ 마이바티스가 제공하는 프레임워크 간의 연동 라이브러리
  + __MyBatis :__ 마이바티스 프레임워크 모듈
  + __mysql-connector-java :__ 사용할 데이터베이스 JDBC 라이브러리
  + __commons-dbcp :__ 커넥션 풀 지원 라이브러리

* __Pom.xml 내용 일부__

```
    ...
    <!-- DataBase DI by doop -->		
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${org.springframework-version}</version>
    </dependency>
    
    <!-- commons dbcp by doop -->
    <dependency>
        <groupId>commons-dbcp</groupId>
        <artifactId>commons-dbcp</artifactId>
        <version>1.4</version>
    </dependency>
    
    <!-- mysql-connector-java by doop -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.12</version>		
    </dependency>
    
    <!-- MyBatis by doop -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.6</version>
    </dependency>
    
    <!-- MyBatis-Spring by doop -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>1.3.2</version>
    </dependency>
    ...
```

<hr />

### MyBatis 연동 설정하기 (root-context.xml)
* __MyBatis 연동을 위한 스프링 의존 관계 설정__
1. 커넥션 풀을 지원하는 __데이터 소스 빈__ 을 등록한다.
2. 스프링의 __트랜잭션 관리자의 빈을 등록__ 한다 (스프링의 트랜잭션 제어를 사용하기 때문에 MyBatis의 트랜잭션 API는 사용하지 않음)
3. MyBatis의 __SqlSessionFactory 빈을 등록__ 한다
4. MyBatis-Spring의 __SqlSessionTemplate 빈을 등록__ 한다.

* __root-context.xml XML 네임스페이스 설정__
  + __XML 문서들의 엘리먼트와 어트리뷰트의 충돌을 피하기 위해 구분을 지어 주는 방법__
<a href="{{ site.spring_img }}/spring_mybatis_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_mybatis_2.JPG" title="Check out the image">
</a>

```
    <!-- DataSource bean 등록 by doop -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>
		<property name="url" value="jdbc:mysql://127.0.0.1:3306/myspring?useSSL=false&amp;serverTimezone=UTC"></property>
		<property name="username" value="root" />
		<property name="password" value="root" />
		<property name="maxActive" value="5" />
	</bean>
	
	<!-- 트랜잭션 관리자 빈 등록 by doop -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean> 
	
	<!-- MyBatis의 SqlSessionFactory 빈 등록 by doop -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
		<property name="configLocation" value="classpath:/mybatis-config.xml"></property>
      	<property name="mapperLocations" value="classpath:mappers/*Mapper.xml"></property> 
	</bean>
	
	<!-- Mybatis-Spring의 SqlSessionTemplate 빈 등록 -->
	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate" destroy-method="clearCache">
		<constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"></constructor-arg>
	</bean>
```

### mybatis-config.xml
* __MyBatis의 공통적인 설정을 지정하는 XML파일__
<a href="{{ site.spring_img }}/spring_mybatis_3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_mybatis_3.JPG" title="Check out the image">
</a>

* __mybatis-config.xml 내용__

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
  
<configuration>
  <typeAliases>
        <!--
            아래와 같이 사용가능
            <package name="org.doorisopen.myspring.Member.Domain" /> 
        -->
        <typeAlias type="org.doorisopen.myspring.Member.Domain.MemberVO" alias="MemberVO"/> 
        <typeAlias type="org.doorisopen.myspring.Board.Domain.BoardVO" alias="BoardVO"/>
        
  </typeAliases>   
</configuration>
```

* ~Mapper 파일에서 parameterType이나 resultType에 사용하는 클래스 이름에서 "org.doorisopen.myspring.Member.Domain"을 생략할 수 있다 

>> __여기까지 MyBatis 연동 완료__

<hr />

### Spring Test
* __스프링 프레임워크에서 동작하도록 만든 클래스__(@Controller, @Service, @Repository, @Component 등이 붙은 클래스)__를 테스트 하는 모듈__
* __빈의 통합 테스트__
  + Junit 에서 DI 컨테이너를 구동시킨 후, __DI 컨테이너 안에 관리되는 빈을 테스트__
* __스트링 테스트가 지원하는 기능__
  + __JUnit 테스팅 프레임워크를 사용하여 스프링의 DI 컨테이너를 동작시키는 기능__
  + 트랜잭션을 테스트 상황에 맞게 제어하는 기능
  + 애플리케이션 서버를 사용하지 않고 스프링 MVC의 동작을 재현하는 기능
  + 테스트 데이터를 적재하기 위해 SQL을 실행하는 기능
  + RestTemplate를 이용해 HTTP 요청에 대한 임의 응답을 보내는 기능

### Spring Test를 위한 라이브러리 등록
* __Pom.xml__

```
    <!-- Spring Test by doop -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${org.springframework-version}</version>
        <scope>test</scope>
    </dependency>
```

### Test Case 작성
* __테스트 케이스 작성을 위한 주요 애노테이션__
  + __@RunWith :__ 스프링 테스트를 위한 DI 컨테이너(애플리케이션 컨텍스트)를 로딩
  + __@ContextConfiguration :__ DI 컨테이너(애플리케이션 컨텍스트)의 설정 파일 위치 또는 설정 클래스를 지정
* __스프링 TestContext 프레임워크 (Spring TestContext Framework)__
  + __스프링 테스트에서 동작하는 테스트용 프레임워크__
  + 스프링 테스트는 Junit에서 스프링 TestContext 프레임워크을 동작시킬 수 있도록 지원하기 위해 org.springframework.test.context.junit4.SpringJUnit4ClassRunner 를 제공

* __DataSourceTest.java__

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring/root-context.xml"})
public class DataSourceTest {
	
	@Inject
	private DataSource ds;
	
	@Test
	public void ConnectionTest() throws Exception {
		
		try(Connection con = ds.getConnection()) {
			System.out.println(con);
		}
		catch(Exception e) {
			e.printStackTrace();
		}
	}
}
```

* __MyBatisTest.java__

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "file:src/main/webapp/WEB-INF/spring/root-context.xml" })
public class MyBatisTest {
	
	
	@Inject
	private SqlSessionFactory sqlFactory;
	
	@Test
	public void testFactory() throws Exception {
		try(SqlSession session = sqlFactory.openSession()) {
			System.out.println(session);
		}
		catch(Exception e) {
			e.printStackTrace();
		}
	}
}
```

<hr />

### References
* 대학교 웹 프레임워크 수업 정리한 내용 입니다.