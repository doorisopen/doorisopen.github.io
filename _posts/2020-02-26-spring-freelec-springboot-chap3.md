---
layout: post
title:  "[Spring Boot] Chap 3.스프링 부트에서 JPA사용하기"
date:   2020-02-26 21:29:40
author: me
categories: Spring
tags: Spring Framework Boot
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

[스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)](https://jojoldu.tistory.com/463) 책에서 공부한 내용을 정리한 게시글입니다.

<br />

### 목차
>> 1. 프로젝트에 Spring Data Jpa 적용하기 & 테스트 코드 작성
>> 2. 등록/수정/조회 API 만들기 & 테스트 코드 작성
>> 3. JPA Audtiting을 이용한 등록/수정 시간 자동화



<br />
<br />


<hr />

## JPA란 
현대의 웹 애플리케이션에서 __관계형 데이터베이스(RDB, Relational Database)__ 는 빠질 수 없는 요소입니다. 그러다 보니 __객체를 관계형 데이터 베이스에서 관리하는 것__ 이 무엇보다 중요합니다.

* __관계형 데이터 베이스__ 는 __어떻게 데이터를 저장__ 할지에 초점이 맞춰진 기술입니다.
* __객체지향 프로그래밍 언어__ 는 메시지를 기반으로 __기능과 속성을 한 곳에서 관리__ 하는 기술입니다.

이 처럼 __관계형 데이터베이스__ 와 __객체지향 프로그래밍 언어__ 의 패러다임이 서로 다른데, 객체를 데이터베이스에 저장하려고 하니 여러 문제가 발생한다. 이를 `패러다임 불일치`라고 한다.

* __객체지향 프로그래밍에서 부뫄 되는 객체를 가져오기__

```
User user = findUser();
Group group = user.getGroup();
```

객체지향 프로그래밍의 경우 User-Group은 부모-자식 관계임을 알 수 있다.

* __데이터베이스 추가된 코드__

```
User user = userDao.findUser();
Group group = groupDao.findGroup(user.getGroupId());
```

데이터베이스가 추가된 경우엔 User 따로, Group 따로 조회하게 된다. 그렇다면 __User와 Group이 어떤 관계__ 인지 알 수 있을까?

상속, 1:N 등 객체 모델링을 데이터베이스로는 구현할 수 없다. 따라서, 웹 애플리케이션 개발은 __데이터 베이스 모델링에만 집중__ 하게 된다. 

JPA는 서로 지향하는 바가 다른 2개의 영역(객체지향 - 관계형 데이터베이스)을 __중간에서 패러다임 일치__ 를 시켜 주기 위한 기술이다. 

즉, __개발자는 객체지향적으로 프로그래밍을 하고, JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성__ 해서 실행한다. 개발자는 항상 객체지향적으로 코드를 표현할 수 있으니 __더는 SQL에 종속적인 개발을 하지 않아도 된다.__

# Spring Data Jpa
JPA는 인터페이스로서 자바 표준명세서 이다. 인터페이스인 JPA를 사용하기 위해서는 구현체가 필요한데, 대표적으로 Hibernate, Eclipse, Link 등이 있다. 하지만 Spring에서 JPA를 사용할 때는 이 구현체들을 직접 다루진 않고 구현체들을 좀 더 쉽게 사용하고자 추상화시킨 Spring Data JPA라는 모듈을 이용하여 JPA 기술을 다룬다.

JPA <- Hibernate <- Spring Data JPA

Hibernate 와 Spring Data JPA 은 서로 큰 차이가 없다. 그러나 __Spring Data JPA가 등장한 이유는 크게 두 가지__ 가 있다.

1. __구현체 교체의 용이성__
2. __저장소 교체의 용이성__

첫 번째, `구현체 교체의 용이성`이란 __Hibernate 외에 다른 구현체로 쉽게 교체하기 위함__ 이다. Spring Data JPA 내부에서 구현체 매핑을 지원해주기 때문에 교체가 용이하다.

두 번째, `저장소 교체의 용이성`이란 __관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위함__ 이다. 서비스 초기에는 관계형 데이터베이스로 모든 기능을 처리했지만, 트래픽이 많아지면서 RDB로는 감당이 안될 수 가 있다. 이때 MongoDB로 교체가 필요하다면 개발자는 __Spring Data JPA에서 Spring Data MongoDB로 의존성만 교체__ 하면 된다.

이는 Spring Data의 하위 프로젝트들은 기본적인 __CRUD의 인터페이스가 같기__ 때문이다. 따라서 Hibernate 보다는 Spring Data를 권장한다.

## Spring Data Jpa 적용하기
build.gradle

```
...(중략)
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter-data-jpa') // chap3 (1)
    compile('com.h2database:h2') // chap3 (2)
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

* (1) `org.springframework.boot:spring-boot-starter-data-jpa`
  + 스프링 부트용 Spring Data Jpa 추상화 라이브러리 입니다.
  + 스프링 부트 버전에 맞춰 자동으로 JPA관련 라이브러리들의 버전을 관리해준다.
* (2) `com.h2database:h2`
  + 인메모리 관계형 데이터베이스입니다.
  + 별도의 설치가 필요 없이 프로젝트 의존성만으로 관리할 수 있습니다.
  + 메모리에서 실행되기 때문에 애플리케이션을 재시작할 때마다 초기화된다는 점을 이용하여 테스트 용도로 많이 사용됩니다.

다음은 __com.doop.book.springboot__ 패키지 아래 __domain 패키지__ 를 만들어준다. __domain 패키지__ 는 __도메인을 담을 패키지__ 이다.
도메인이란 게시글, 댓글, 회원, 정산, 결제 등 소프트웨어에 대한 요구사항 혹은 문제 영역이라고 생각하면된다. 기존에 MyBatis와 같은 쿼리 매퍼를 사용했다면 dao 패키기를 떠올리겠지만, dao 패키지와는 조금 결이 다르다. 그간 xml에 쿼리를 담고, 클래스는 오로지 쿼리의 결과만 담던 일들이 모두 도메인 클래스라고 불리는 곳에서 해결된다.

__domain 패키지__ 아래에 __posts 패키지와 Posts 클래스__ 를 만든다. 그리고 아래와 같이 코드를 작성한다.

```
package com.doop.book.springboot.domain.posts;

import com.doop.book.springboot.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter // (6)
@NoArgsConstructor // (5)
@Entity // (1)
public class Posts {

    @Id // (2)
    @GeneratedValue(strategy = GenerationType.IDENTITY) // (3)
    private Long id;

    @Column(length = 500, nullable = false) // (4)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder // (7)
    public Posts(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
```

* (1) `@Entity` 
  + 테이블과 링크될 클래스임을 나타냅니다.
  + 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍(_)으로 테이블 이름을 매칭합니다.
  + ex) SalesManager.java -> sales_manager table
* (2) `@Id`
  + 해당 테이블의 PK필드를 나타냅니다.
* (3) `@GeneratedValue`
  + PK의 생성 규칙을 나타냅니다.
  + 스프링 부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment가 됩니다.
  + 스프링 부트 2.0버전과 1.5 버전의 차이는 추후 새로운 게시글에 추가
* (4) `@Column`
  + 테이블의 칼럼을 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 됩니다.
  + 사용하는 이유는, 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용합니다.
  + 문자열의 경우 VARCHAR(255)가 기본값인데, 사이즈를 500으로 늘리고 싶거나 (ex: title), 타입을 TEXT로 변경하고 싶거나(ex: content) 등의 경우에 사용됩니다.
* (5) `@NoArgsConstructor`
  + 기본 생성자 자동 추가
  + public Posts() {}와 같은 효과
* (6) `@Getter`
  + 클래스 내 모든 필드의 Getter 메소드를 자동생성
* (7) `@Builder`
  + 해당 클래스의 빌더 패턴 클래스를 생성
  + 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함

_※참고※_

__Entity의 PK는 Long 타입의 Auto_increament를 추천합니다.__ 이유는 주민등록번호와 같이 비즈니스상 유니크 키나, 여러 키를 조합한 복합키로 PK를 잡을 경우

1. FK를 맺을 때 다른 테이블에서 복한키 전부를 갖고 있거나, 중간 테이블을 하나 더 둬야 하는 상황이 발생합니다.
2. 인덱스에 좋은 영향을 끼치지 못합니다.
3. 유니크한 조건이 변경될 경우 PK 전체를 수정해야 하는 일이 발생합니다.

따라서, 주민등록번호, 복합키 등은 유니크 키로 별도로 추가하는 것을 추천합니다.

_※참고※_

__Entity 클래스에서는 절대 Setter 메소드를 만들지 않습니다.__ 이유는 해당 클래스의 인스턴스 값들이 언제 어디서 변해야 하는지 코드상으로 명확하게 구분할 수가 없어, 차후 기능 변경 시 정말 복잡해지기 때문입니다.

대신, 해당 필드의 값 변경이 필요하면 명확히 그 목적과 의도를 나타낼 수 있는 메소드를 추가해야만 합니다.

__Setter가 없는 상황에서 어떻게 값을 채워 DB에 삽입__ 해야할까???

기본적인 구조는 __생성자를 통해__ 최종값을 채운 후 DB에 삽입 하는 것 이며, 값 변경이 필요한 경우 __해당 이벤트에 맞는 public 메소드를 호출하여 변경__ 하는 것을 전제로 합니다.

또 다른 방법으로는 __@Builder를 통해 제공되는 빌더 클래스__ 를 사용하는 것이다. 생성자나 빌더나 생성 시점에 값을 채워주는 역할은 똑같습니다. 다만, 생성자의 경우 지금 채워야 할 필드가 무엇인지 명확히 지정할 수 가 없습니다.

예를 들어 다음과 같은 코드가 있을때 __new Example(b, a)__

```
public Example(String a, String b) {
    this.a = a;
    this.b = b;
}
```

a와 b의 위치를 변경해도 코드를 실행하기 전까지는 문제를 찾을 수가 없다. 하지만 빌더를 사용하게 되면 다음과 같이 __어느 필드에 어떤 값을 채워야 할지 명확하게 인지__ 할 수 있다.

```
Example.builder()
    .a(a)
    .b(b)
    .build();
```

그리고 __com.doop.book.springboot.domain.posts__ 위치에 Posts 클래스로 DB를 접근하게 해줄 JpaRepository를 생성해준다.

```
package com.doop.book.springboot.domain.posts;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PostsRepository extends JpaRepository<Posts, Long> {

}
```

보통 ibatis나 MyBatis 등에서 Dao라고 불리는 __DB layer 접근자__ 입니다. __JPA에선 Repository라고 부르며 인터페이스로 생성__ 합니다.

그리고 `JpaRepository<Entity 클래스, PK타입>`를 상속하면 기본적인 CRUD 메소드가 자동으로 생성됩니다. 또한 __@Repository를 추가할 필요 없다.__

_※주의할 점※_

__Entity 클래스와 기본 Entity Repository는 함께 위치__ 해야 한다. __Entity 클래스는 기본 Repository 없이는 제대로 역할을 할 수가 없습니다.__

## 테스트 코드 작성
test 디렉토리에 __domain.posts 패키지를 생성 후 PostsRepositoryTest 클래스를 생성__ 한다.

```
package com.doop.book.springboot.web.domain.posts;


import com.doop.book.springboot.domain.posts.Posts;
import com.doop.book.springboot.domain.posts.PostsRepository;

import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.time.LocalDateTime;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest {

    @Autowired
    PostsRepository postsRepository;

    @After // (1)
    public void cleanup() {
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기() {
        // given
        String title = "테스트 게시글";
        String content = "테스트 본문";

        postsRepository.save(Posts.builder()
                            .title(title)
                            .content(content)
                            .author("audauddl2@gmail.com")
                            .build()); // (2)

        // when
        List<Posts> postsList = postsRepository.findAll(); // (3)
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }
}
```

* (1) `@After`
  + Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정
  + 보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기 위해 사용합니다.
  + 여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2에 데이터가 그대로 남아 있어 다음 테스트 실행 시 테스트가 실패할 수 있습니다.
* (2) `postsRepository.save`
  + 테이블 posts에 insert/update 쿼리를 실행합니다
  + id 값이 있다면 update가, 없다면 insert 쿼리가 실행됩니다.
* (3) `postsRepository.findAll`
  + 테이블 posts에 있는 모든 데이터를 조회해오는 메소드입니다.

`@SpringBootTest`를 사용할 경우 __H2 데이터베이스를 자동으로 실행__ 해 준다.

## 로그 출력 설정하기(application.properties)
만약 실제로 실행된 쿼리의 형태를 보고싶다면 __src/main/resources 디렉토리 아래 application.properties 파일을 생성__ 하고 __spring.jpa.show_sql=true__ 를 작성 후 실행하면 된다.

단 출력되는 형태는 H2 문법이 적용된 로그이다 H2가 아닌 MySQL 버전으로 출력 하고싶다면 __spring.jpa.properties.hibernate.dialect= org.hibernate.dialect.MySQL5InnoDBDialect__ 코드를 추가하면 된다.

# 등록/수정/조회 API 만들기
API를 만들기 위해 총 __3개의 클래스__ 가 필요하다.

* Request 데이터를 받을 `Dto`
* API 요청을 받을 `Controller`
* 트랜잭션, 도메인 기능 간의 순서를 보장하는 `Service`

Service의 경우 비지니스 로직 처리를 담당하는 것으로 오해하는 경우가 많다 그러나 __Service는 트랜잭션, 도메인 간 순서 보장의 역할만 한다.__

아래의 __Spring 웹 계층__ 각각을 소개하면 다음과 같습니다. 

<a href="{{ site.2020_spring_img }}/freelec-springboot-chap3-1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2020_spring_img }}/freelec-springboot-chap3-1.JPG" title="Check out the image">
</a>

* __Web Layer__
  + 흔히 사용하는 컨트롤러(@Controller)와 JSP/Freemarker 등의 뷰 템플릿 영역입니다.
  + 이외에도 필터(@Filter), 인터셉터, 컨트롤러 어드바이스(@ControllerAdvice)등 외부 요청과 응답에 대한 전반적인 영역을 이야기합니다.
* __Service Layer__ 
  + @Service에 사용되는 서비스 영역입니다.
  + 일반적으로 Controller와 Dao의 중간 영역에서 사용됩니다.
  + @Transactional이 사용되어야 하는 영역이기도 합니다.
* __Repository Layer__
  + Database와 같이 데이터 저장소에 접근하는 영역입니다.
  + 기존에 개발하셨던 분들이라면 Dao(Data Access Object)영역으로 이해하시면 쉬울 것입니다.
* __Dtos__
  + Dto(Data Transfer Object)는 __계층 간에 데이터 교환을 위한 객체__ 를 이야기하며 __Dtos__ 는 __이들의 영역__ 을 얘기합니다.
  + 예를 들어 뷰 템플릿 엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체 등이 이들을 이야기합니다.
* __Domain Model__
  + 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것을 도메인 모델이라고 합니다.
  + 이를테면 택시 앱이라고 하면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있습니다.
  + @Entity를 사용해보신 분들은 @Entity가 사용된 영역 역시 도메인 모델이라고 이해하면 됩니다.
  + 다만, 무조건 데이터베이스의 테이블과 관계가 있어야만 하는 것은 아닙니다.
  + VO처럼 값 객체들도 이 영역에 해당하기 때문입니다.

위의 5가지 계층에서 비지니스 처리를 담당하는 곳은 __Domain__ 입니다. 그리고 기존에 서비스로 처리하던 방식을 __트랜잭션 스크립트__ 라고 합니다. 또한, __서비스__ 는 __트랜잭션과 도메인 간의 순서만 보장해 줍니다.__

__web 패키지__ 에, PostsApiController / __web.dto 패키지__ 에 PostsSaveRequestDto / __service 패키지__ 에 PostsService를 생성합니다.

* __PostsApiController__
```
// PostsApiController.class
package com.doop.book.springboot.web;

import com.doop.book.springboot.service.posts.PostsService;
import com.doop.book.springboot.web.dto.PostsSaveRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RequiredArgsConstructor
@RestController
public class PostsApiController {

    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto) {
        return postsService.save(requestDto);
    }
}
```

* __PostsService__
```
// PostsService
package com.doop.book.springboot.service.posts;

import com.doop.book.springboot.domain.posts.PostsRepository;
import com.doop.book.springboot.web.dto.PostsSaveRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import javax.transaction.Transactional;

@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto) {
        return postsRepository.save(requestDto.toEntity()).getId();
    }

    @Transactional
    public Long update(Long id, PostsUpdateRequestDto requestDto) {
        Posts posts = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id="+ id));

        posts.update(requestDto.getTitle(), requestDto.getContent());
        return id;
    }

    public PostsResponseDto findById(Long id) {
        Posts entity = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));
        return new PostsResponseDto(entity);
    }
}

```

Controller와 Service 코드를 살펴보면 @Autowired가 없는 것을 볼 수 있다. 스프링에서 Bean 주입 방식에는 3가지가 있다.

* @Autowired
* setter
* 생성자

이 중에서 __생성자로 주입받는 방식__ 을 가장 권장한다. 위 코드에서는 생성자로 Bean을 주입 하는 부분이 없는데 __@RequiredArgsConstructor__ 가 생성자로 주입받는 부분을 해결해 준것이다. 

__@RequiredArgsConstructor__ 는 final로 선언된 모든 필드를 인자값으로 하는 생성자를 롬복의 __@RequiredArgsConstructor__ 가 대신 생성해 준 것입니다.

### 생성자가아닌 @RequiredArgsConstructor 를 사용한 이유??
해당 클래스의 의존성 관계가 변경될 때마다 생성자 코드를 계속해서 수정하는 번거로움을 해결하기 위함입니다.

* __PostsSaveRequestDto__
```
// PostsSaveRequestDto
package com.doop.book.springboot.web.dto;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class PostsSaveRequestDto {
    private String title;
    private String content;
    private String author;

    @Builder
    public PostsSaveRequestDto(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public Posts toEntity() {
        return Posts.builder()
                .title(title)
                .content(content)
                .author(author)
                .build();
    }
}
```

Controller와 Service에서 사용할 Dto 클래스를 생성하고 위의 코드를 작성했으면 코드를 살펴보자.

__Dto 클래스__ 가 __Entity 클래스__ 와 거의 유사한 형태이지만 Entity 클래스를 __Request/Response 클래스로 사용해서는 안 됩니다.__


### Dto와 Entity 클래스를 분리하는 이유
__Entity 클래스__ 는 __데이터베이스와 맞닿은 핵심 클래스__ 입니다. Entity 클래스를 기준으로 테이블이 생성되고, 스키마가 변경됩니다. 화면 변경은 아주 사소한 기능 변경인데, 이를 위해 테이블과 연결된 Entity클래스를 변경하는 것은 너무 큰 변경입니다.

또한, 수많은 서비스 클래스나 비즈니스 로직들이 Entity 클래스를 기준으로 동작합니다. 

__Dto(Request/Response)__ 는 View를 위한 클래스라 정말 자주 변경이 필요합니다.

따라서, Entity 클래스와 Controller에서 쓸 Dto는 분리해서 사용해야 합니다.

## 테스트 코드 작성

```
package com.doop.book.springboot.web;

import com.doop.book.springboot.domain.posts.Posts;
import com.doop.book.springboot.domain.posts.PostsRepository;
import com.doop.book.springboot.web.dto.PostsSaveRequestDto;
import com.doop.book.springboot.web.dto.PostsUpdateRequestDto;
import javafx.geometry.Pos;
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
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @After
    public void tearDown() throws Exception {
        postsRepository.deleteAll();
    }

    @Test
    public void Posts_등록된다() throws Exception {
        // given
        String title = "title";
        String content = "content";
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                .title(title)
                .content(content)
                .author("author")
                .build();

        String url = "http://localhost:" + port + "/api/v1/posts";

        // when
        ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

        // then
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }
}
```

## 수정/조회 API
PostsApiController, PostsResponseDto, PostsUpdateRequestDto, Posts, PostsService 수정 필요
* __PostsApiController__
```
package com.doop.book.springboot.web;
...(중략)
@RequiredArgsConstructor
@RestController
public class PostsApiController {

    ...(중략)
    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto) {
        return postsService.update(id, requestDto);
    }

    @GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findById (@PathVariable Long id) {
        return postsService.findById(id);
    }
}
```

* __PostsResponseDto__
```
package com.doop.book.springboot.web.dto;

import com.doop.book.springboot.domain.posts.Posts;
import lombok.Getter;

@Getter
public class PostsResponseDto {
    private Long id;
    private String title;
    private String content;
    private String author;

    public PostsResponseDto(Posts entity) {
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.content = entity.getContent();
        this.author = entity.getAuthor();
    }
}
```

__PostsResponseDto__ 는 __Entity의 필드 중 일부만 사용__ 하므로 생성자로 Entity를 받아 필드에 값을 넣습니다. 굳이 모든 필드를 가진 생성자가 필요하진 않으므로 Dto는 Entity를 받아 처리합니다.


* __PostsUpdateRequestDto__
```
package com.doop.book.springboot.web.dto;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class PostsUpdateRequestDto {
    private String title;
    private String content;

    @Builder
    public PostsUpdateRequestDto(String title, String content) {
        this.title = title;
        this.content = content;
    }
}
```

* __Posts__
```
package com.doop.book.springboot.domain.posts;
...(중략)
@Getter
@NoArgsConstructor
@Entity
public class Posts {

    ...(중략)
    public void update(String title, String content) {
        this.title = title;
        this.content = content;
    }
}
```

* __PostsService__
```
package com.doop.book.springboot.service.posts;
...(중략)
@RequiredArgsConstructor
@Service
public class PostsService {
    
    ...(중략)
    @Transactional
    public Long update(Long id, PostsUpdateRequestDto requestDto) {
        Posts posts = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id="+ id));

        posts.update(requestDto.getTitle(), requestDto.getContent());
        return id;
    }

    public PostsResponseDto findById(Long id) {
        Posts entity = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));
        return new PostsResponseDto(entity);
    }
}
```

위의 update 기능의 코드를 살펴보면 데이터베이스에 __쿼리를 날리는 부분이 없음__ 을 알 수 있다. 이게 가능한 이유는 JPA의 `영속성 컨텍스트` 때문입니다.

`영속석 컨텍스트`란, __엔티티를 영구 저장하는 환경__ 입니다. 일종의 논리적 개념이라고 보면 되며, JPA의 핵심 내용은 __엔티티가 영속성 컨텍스트에 포함되어 있냐 아니냐__ 로 갈립니다.

JPA의 엔티티 매니저가 활성화된 상태(Spring Data Jpa의 기본 값)로 __트랜잭션 안에서 DB에서 데이터를 가져오면__ 이 데이터는 영속성 컨텍스트가 유지된 상태입니다. 이 상태에서 해당 데이터의 값을 변경하면 __트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영__ 합니다. 즉, Entity 객체의 값만 변경하면 별도로 __Update 쿼리를 날릴 필요가 없다__ 는 것이다. 이 개념을 __더티 체킹__ 이라고 합니다.

## 수정 기능 테스트 코드 작성
* __PostsApiControllerTest__
```
package com.doop.book.springboot.web;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    ...(중략)

    @Test
    public void Posts_수정된다() throws Exception {
        // given
        Posts savePosts = postsRepository.save(Posts.builder()
                .title("title")
                .content("content")
                .author("author")
                .build());

        Long updateId = savePosts.getId();
        String expectedTitle = "title2";
        String expectedContent = "content2";

        PostsUpdateRequestDto requestDto = PostsUpdateRequestDto.builder()
                .title(expectedTitle)
                .content(expectedContent)
                .build();

        String url = "http://localhost:" + port + "/api/v1/posts/" + updateId;

        HttpEntity<PostsUpdateRequestDto> requestEntity = new HttpEntity<>(requestDto);

        // when
        ResponseEntity<Long> responseEntity = restTemplate.exchange(url, HttpMethod.PUT, requestEntity, Long.class);

        // then
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }
}
```

## H2 데이터베이스에서 확인하기

1. application.properties에 `spring.h2.console.enabled=true` 코드 추가 한다.
2. 웹 브라우저에서 `http://localhost:8080/h2-console`로 접속하고 다음 단계의 설정을 해준다.
3. __JDBC URL__ : jdbc:h2:mem:testdb
4. __user Name__ : sa 를 입력하고 접속한다.

# JPA Audtiting을 이용한 등록/수정 시간 자동화
보통 엔티티에는 해당 데이터의 생성시간과 수정시간을 포함합니다. 이는 추후 유지보수에 있어서 중요한 정보이다. 그렇기 때문에 DB에 삽입, 갱신 코드가 반복되고 모든 테이블과 서비스 메소드에 포함되어지면서 코드가 지저분해집니다. 이러한 문제를 해결하기위해 __JPA Auditing__ 를 사용합니다.

## LocalData 사용
Java8부터 LocalDate와 LocalDateTime이 등장했고, 이는 Java의 기본 날짜 타입인 Date의 문제점을 고친 타입입니다.

### Java8 이전 Date/Calendar 클래스의 문제점
1. 불변 객체가 아닙니다. 이는 멀티스레드 환경에서 언제든 문제가 발생할 수 있습니다.
2. Calendar는 월(Month) 값 설계가 잘못되었습니다.(10월:Calendar.OCTOBER의 숫자 값은 '10'이 아닌 '9' 입니다)


__domain 패키지에 BaseTimeEntity 클래스를 생성__ 합니다.

```
package com.doop.book.springboot.domain;

import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDate;
import java.time.LocalDateTime;

@Getter
@MappedSuperclass // (1)
@EntityListeners(AuditingEntityListener.class) // (2)
public class BaseTimeEntity {

    @CreatedDate // (3)
    private LocalDateTime createdDate;

    @LastModifiedDate // (4)
    private LocalDateTime modifiedDate;
}
```

* (1) `@MappedSuperclass`
  + JPA Entity 클래스들이 BaseTimeEntity을 상속할 경우 필드들(createdDate, modifiedDate)도 칼럼으로 인식하도록 합니다.
* (2) `@EntityListeners`
  + BaseTimeEntity 클래스에 Auditing 기능을 포함시킵니다.
* (3) `@CreatedDate`
  + Entity가 생성되어 저장될 때 시간이 자동 저장됩니다.
* (4) `@LastModifiedDate`
  + 조회한 Entity의 값을 변경할 때 시간이 자동으로 저장됩니다.

그리고 Posts클래스가 BaseTimeEntity를 상속받도록 변경합니다.

```
public class Posts extends BaseTimeEntity {
...(중략)
}
```

## 테스트 코드 작성
* __PostsRepositoryTest__ 코드 추가
```
package com.doop.book.springboot.web.domain.posts;
...(중략)
@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest {

    ...(중략)

    @Test
    public void BaseTimeEntity_등록() {
        // given
        LocalDateTime now = LocalDateTime.of(2020,2,25,0,0,0);
        postsRepository.save(Posts.builder()
                .title("title")
                .content("content")
                .author("author")
                .build());
        // when
        List<Posts> postsList = postsRepository.findAll();

        // then
        Posts posts = postsList.get(0);

        System.out.println(">>>>>>>> createDate="+posts.getCreatedDate()+", modifiedDate="+posts.getModifiedDate());

        assertThat(posts.getCreatedDate()).isAfter(now);
        assertThat(posts.getModifiedDate()).isAfter(now);

    }
}
```

# References
* 스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)
