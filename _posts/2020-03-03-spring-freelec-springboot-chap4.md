---
layout: post
title:  "[Spring Boot] Chap 4.Mustache로 화면 구성하기"
date:   2020-03-03 00:44:40
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
>> 1. Mustache란?
>> 2. Mustache 검증 하기(Test Code)
>> 3. Mustache로 CRUD 페이지 만들기



<br />
<br />


<hr />

들어가기 앞서 웹 개발에 있어 `템플릿 엔진`이라는 것이 있습니다. 이것은 __지정된 템플릿 양식과 데이터__ 가 합쳐져 HTML 문서를 출력하는 소프트웨어를 이야기 합니다. 예를 들면 Spring 이나 서블릿 같은 경우는 JSP, Freemarker 등 그리고 React, Vue 들이 __지정된 템플릿과 데이터__ 를 이용하여 HTML을 생성하는 `템플릿 엔진` 입니다.

그러나, JSP 등 과 같은 전자의 경우 __서버 템플릿 엔진__ 이라 부르며, React와 같은 후자의 경우 __클라이언트 템플릿 엔진__ 이라 불립니다.

또한, 알아야 할 점은 __프론트엔드의 자바스크립트(NodeJs X)__ 가 작동하는 영역과 __JSP__ 가 작동하는 영역이 다르다는 점입니다. __JSP(+Spring)__ 를 비롯한 서버 템플릿 엔진은 __서버에서 구동__ 이 됩니다. 서버 템플릿 엔진을 이용한 화면 생성은 서버에서 Java코드로 문자열을 만든 뒤 이 문자열을 HTML로 변환하여 브라우저로 전달합니다. 반면에 __자바스크립트__ 는 __브라우저 위에서 작동__ 합니다. 브라우저에서 작동될 때는 서버 템플릿 엔진의 손을 벗어나 제어할 수가 없습니다.

React.js나 Vue.js를 이용한 SPA(Single Page Application)는 __브라우저에서 화면을 생성__ 합니다. 즉, __서버에서 이미 코드가 벗어난 경우__ 입니다.

# Mustache란
__머스테치(Mustache)__ 는 JSP와 같이 HTML을 만들어 주는 템플릿 엔진입니다. 또한 수많은 언어를 지원하는 가장 심플한 템플릿 엔진입니다.(루비, 자바스크립트, 파이썬, PHP, 자바, 펄, Go, ASP 등 지원)

## 템플릿 엔진들의 단점, Mustache의 장점
### JSP, Velocity
스프링 부트에서는 권장하지 않는 템플릿 엔진입니다.

### Freemarker
템플릿 엔진으로는 너무 과하게 많은 기능을 지원합니다. 높은 자유도로 인해 숙련도가 낮을수록 Freemarker 안에 비즈니스 로직이 추가될 확률이 높습니다.

### Thymeleaf
스프링 진영에서 적극적으로 밀고 있지만 문법이 어렵습니다. HTML 태그에 속성으로 템플릿 기능을 사용하는 방식에 진입장벽이 높게 느껴집니다. Vue.js 사용 경험이 있어 태그 속성 방식이 익숙한 분이라면 선택해도 됩니다.

### Mustache
* 문법이 다른 템플릿 엔진보다 심플합니다.
* 로직 코드를 사용할 수 없어 View의 역할과 서버의 역할을 명확하게 분리됩니다.
* Mustache.js와 Mustache.java 2가지가 다 있어, 하나의 문법으로 클라이언트/서버 템플릿을 모두 사용 가능합니다.


그러면 Mustache를 사용하기 위해서 __Mustache 플러그인을 설치__ `(Action(Ctrl+Shift+A) > plugins 검색 > Mustache검색 후 설치)` 하고 인텔리제이를 재시작 해줍니다. 

재시작 후 build.gradle에 머스테치 스타터 의존성을 등록해 줍니다.

* build.gradle

```
dependencies {
    ...(중략)
    compile('org.springframework.boot:spring-boot-starter-mustache') // chap4
}
```

__머스테치의 파일 위치__ 는 기본적으로 `src/main/resources/templates` 입니다. 이 위치에 머스테치 파일을 두면 스프링 부트에서 자동으로 로딩합니다.

그리고 이 경로에 __index.mustache__ 를 생성합니다.
* index.mustache

```
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <h1>스프링 부트로 시작하는 웹 서비스</h1>
</body>
</html>
```

이 머스테치에 URL 매핑을 위해 __web 패키기 안에 IndexController__ 를 만들어 줍니다.

* IndexController

```
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class IndexController {

    @GetMapping("/")
    public String index() {
        
        return "index";
    }
}
```

머스테치 스타터 덕분에 컨트롤러에서 무자열을 반환할 때 __앞의 경로와 뒤의 파일 확장자는 자동으로 지정__ 됩니다.

앞의 경로는 __src/main/resources/templates__, 뒤의 파일 확장자는 __.mustache__ 가 붙는 것입니다. 즉 위의 코드에서는 "index"를 반환하므로, __src/main/resources/templates/index.mustache__ 로 전환되어 View Resolver가 처리하게 됩니다.

# Mustache 검증 하기(Test Code)
index Mustache를 검증하기 위해 test 패키지에 __IndexControllerTest 클래스__ 를 생성합니다.

* IndexControllerTest

```
package com.doop.book.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.boot.test.context.SpringBootTest.WebEnvironment.RANDOM_PORT;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = RANDOM_PORT)
public class IndexControllerTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void 메인페이지_로딩() {
        // when
        String body = this.restTemplate.getForObject("/", String.class);

        // then
        assertThat(body).contains("스프링 부트로 시작하는 웹 서비스");
    }
}
```

위의 테스트는 실제로 URL 호출 시 페이지의 내용이 제대로 호출되는지에 대한 테스트 입니다.

# Mustache로 CRUD 페이지 만들기
이전에 __PostsApiController__ 에 개발한 API와 Mustache를 사용하여 CRUD 페이지를 구성해보겠습니다.

추가로 부트스트랩, 제이쿼리 등 __프론트엔드 라이브러리__ 를 사용할 수 있는 방법은 크게 __2가지__ 가 있습니다.

1. 외부 CDN 사용
2. 직접 라이브러리를 받아서 사용

__1번 방법__ 을 사용해서 화면을 구성해 보겠습니다.

먼저 위에서 작성한 __index.mustache__ 를 __레이아웃 방식(공통 영역을 별도의 파일로 분리하여 필요한 곳에서 가져다 쓰는 방식)__ 으로 바꿔 줍니다.

src/main/resources/template 디렉토리에 __layout 디렉토리를 추가로 생성하고 그 안에 header.mustache, footer.mustache__ 파일을 생성합니다.

* header, footer

```
<!-- 여기부터 header -->
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>
<body>
<!-- /.header -->

<!-- 여기부터 footer -->
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
<!-- index.js 추가 -->
<script src="/js/app/index.js"></script>
</body>
</html>
<!-- /.footer -->
```

위 코드를 각각의 파일에 작성하면 된다. 그리고 위의 코드를 살펴보면 css, js의 위치가 서로 다릅니다. __페이지 로딩속도를 높이기 위해 css는 header에, js는 footer에 두었습니다.__ HTML은 위에서부터 코드가 실행되기 때문에 __head가 다 실행되고서야 body가 실행__ 됩니다.

* index.mustache

```
{.{>layout/header}.} <!-- (1) -->
    <h1>스프링 부트로 시작하는 웹 서비스</h1>
{.{>layout/footer}.}
```

* (1) `{.{>layout/header}.}` \{\{>\}\}는 현재 머스테치 파일을 기준으로 다른 파일을 가져옵니다.
  + 참고로 `{.{...}.}` 이렇게 작성되어있는데 __머스테치 문법이 마크다운에서 인식을 못하는건지 해당 내용이 생략__ 되는 현상이 있다. 그래서 `.`를 추가했다 만약 코드를 사용할때는 `.`은 제외하고 사용해야한다.

## 게시글 등록
1. 게시글 등록 Button을 만들기
2. 등록 페이지를 호출하는 URL을 IndexController에 추가
3. 게시글 등록 Page 만들기
4. 게시글 등록 API 기능(js)

### 1. 게시글 등록 Button을 만들기
* index.mustache

```
{.{>layout/header}.}

    <h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <div calss="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
            </div>
        </div>
    </div>
{.{>layout/footer}.}
```
### 2. 등록 페이지를 호출하는 URL을 IndexController에 추가
* IndexController

```
@RequiredArgsConstructor
@Controller
public class IndexController {
...(중략)
@GetMapping("/posts/save")
    public String postsSave() {
        return "posts-save";
    }
}
```

### 3. 게시글 등록 Page 만들기
index.mustache와 같은 위치에 __posts-save.mustache 파일 생성__ 

* posts-save.mustache

```
{.{>layout/header}.}
    <h1>게시글 등록</h1>
    <div class="col-md-12">
        <div calss="col-md-4">
            <form>
                <div class="form-group">
                    <label for="title">제목</label>
                    <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
                </div>
                <div class="form-group">
                    <label for="author">작성자</label>
                    <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
                </div>
                <div class="form-group">
                    <label for="content">내용</label>
                    <textarea class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
                </div>
            </form>
            <a href="/" role="button" class="btn btn-secondary">취소</a>
            <button type="button" class="btn btn-primary" id="btn-save">등록</button>
        </div>
    </div>
{.{>layout/footer}.}
```

### 4. 게시글 등록 API 기능(js)
게시글 등록 API를 호출하는 JS를 만들기 위해 __src/main/resources에 static/js/app 디렉토리를 생성하고 index.js 파일을 생성__ 합니다.

* index.js

```
var main = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
            _this.save();
        });
    },
    save : function () {
        var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };
        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType: 'application/json; charset-utf-8',
            data: JSON.stringify(data)
        }).done(function () {
            alert('글이 등록되었습니다.');
            window.location.href = '/'; // (1)
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
};

main.init();
```

* (1) `window.location.href = '/';` 글 등록이 성공하면 메인페이지(/)로 이동합니다.

index.js의 첫 문장에 __var main = {...}__ 라는 코드를 선언 했습니다. 

#### main 이라는 변수의 속성으로 function을 추가한 이유는 무엇인가?
브라우저의 스코프(scope)는 __공용 공간__ 으로 쓰이기 때문에 __나중에 로딩된 js의 init, save가 먼저 로딩된 js의 function을 덮어쓰게 됩니다.__

여러 사람이 참여하는 프로젝트에서는 __중복된 함수 이름__ 은 자주 발생할 수 있습니다. 모든 function 이름을 확인하면서 만들 수는 없습니다.

그러다 보니 이런 문제를 피하려고 index.js만의 유효범위를 만들어 사용합니다.

방법은 var main이란 객체를 만들어 해당 객체에서 필요한 모든 function을 선언하는 것입니다. 이렇게 하면 index 객체 안에서만 function이 유효하기 때문에 다른 JS와 겹칠 위험이 사라집니다.


__footer.mustache__ 에 index.js를 추가합니다.
* footer.mustache

```
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
<!-- index.js 추가 -->
<script src="/js/app/index.js"></script>
</body>
</html>
```

index.js 호출 코드를 보면 __절대 경로(/)__ 로 시작합니다. 스프링 부트는 기본적으로 __src/main/resources/static__ 에 위치한 자바스크립트, CSS, 이미지 등 정적 파일들을 URL에서 /로 설정됩니다. 또한, 아래와 같이 파일이 위치하면 위치에 맞게 호출이 가능합니다.

* src/main/resources/static/js/... `(http://도메인/js/...)`
* src/main/resources/static/css/... `(http://도메인/css/...)`
* src/main/resources/static/image/... `(http://도메인/image/...)`


## 게시글 조회
1. index.mustache 수정
2. Controller, Service, Repository 수정

### 1. index.mustache 수정

```
{.{>layout/header}.}
    <h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <div calss="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
            </div>
        </div>
    </div>
    <br>
    <!--목록 출력 영역-->
    <table class="table table-horizontal table-bordered">
        <thead class="thead-strong">
        <tr>
            <td>게시글번호</td>
            <td>제목</td>
            <td>작성자</td>
            <td>최종수정일</td>
        </tr>
        </thead>
        <tbody id="tbody">
        {.{#posts}.} <!-- (1) -->
            <tr>
                <td>{.{id}.}</td> <!-- (2) -->
                <td><a href="/posts/update/{.{id}.}">{.{title}.}</a></td>
                <td>{.{author}.}</td>
                <td>{.{modifiedDate}.}</td>
            </tr>
        {.{/posts}.}
        </tbody>
    </table>
{.{>layout/footer}.}
```

* (1) `{.{#posts}.}`
  + posts 라는 List를 순회합니다.
  + Java의 for문과 동일하게 생각하면 됩니다.
* (2) `{.{id}.} 등의 {.{변수명}.}`
  + List에서 뽑아낸 객체의 필드를 사용합니다.

### 2. Controller, Service, Repository 수정
* PostsRepository

```
package com.doop.book.springboot.domain.posts;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.util.List;

public interface PostsRepository extends JpaRepository<Posts, Long> {

    @Query("SELECT p FROM Posts p ORDER BY p.id DESC")
    List<Posts> findAllDesc();
}
```

_※참고※_

규모가 있는 프로젝트에서의 데이터 조회는 FK의 조인, 복잡한 조건 등으로 인해 이런 Entity 클래스만으로 처리하기 어려워 조회용 프레임워크를 추가로 사용합니다. 대표적 예로 querydsl, jooq, MyBatis 등이 있습니다. 조회는 위 3가지 프레임워크 중 하나를 통해 조회하고, 등록/수정/삭제 등은 SpringDataJpa를 사용합니다.

* __Querydsl 추천 및 장점__
  + __타입 안정성이 보장됩니다.__
    - 단순한 문자열로 쿼리를 생성하는 것이 아니라, 메소드를 기반으로 쿼리를 생성하기 때문에 오타나 존재하지 않는 컬럼명을 명시할 경우 IDE에서 자동으로 검출됩니다. 이 장점은 Jooq에서도 지원하는 장점이지만, MyBatis에서는 지원하지 않습니다.
  + __국내 많은 회사에서 사용 중입니다.__
  + __레퍼런스가 많습니다.__


* PostsService

```
...
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    ...(중략)
    @Transactional(readOnly = true) // (1)
    public List<PostsListResponseDto> findAllDesc() {
        return postsRepository.findAllDesc().stream()
                .map(PostsListResponseDto::new) // (2)
                .collect(Collectors.toList());
    }
}
```

* (1) `readOnly = true`
  + 트랜잭션 범위는 유지하되, 조회 기능만 남겨두어 __조회 속도가 개선__ 되기 때문에 등록, 수정, 삭제 기능이 전혀 없는 서비스 메소드에서 사용하는 것을 추천합니다.
* (2) `.map(PostsListResponseDto::new)`은 `.map(posts -> new PostsListResponseDto(posts))`와 같습니다.
  + postsRepository 결과로 넘어온 Posts의 Stream을 map을 통해 PostsListResponseDto 변환 -> List로 반환하는 메소드 입니다.


* PostsListResponseDto __(dto 패키지에 생성)__

```
package com.doop.book.springboot.web.dto;

import com.doop.book.springboot.domain.posts.Posts;
import lombok.Getter;

import java.time.LocalDateTime;

@Getter
public class PostsListResponseDto {
    private Long id;
    private String title;
    private String author;
    private LocalDateTime modifiedDate;

    public PostsListResponseDto(Posts entity) {
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.author = entity.getAuthor();
        this.modifiedDate = entity.getModifiedDate();
    }
}
```

* IndexController

```
package com.doop.book.springboot.web;

import com.doop.book.springboot.config.auth.LoginUser;
import com.doop.book.springboot.config.auth.dto.SessionUser;
import com.doop.book.springboot.service.posts.PostsService;
import com.doop.book.springboot.web.dto.PostsResponseDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

import javax.servlet.http.HttpSession;

@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")
    public String index(Model model) { // (1)
        model.addAttribute("posts", postsService.findAllDesc());
        return "index";
    }
}
```

* (1) `Model`
  + 서버 템플릿 엔진에서 사용할 수 있는 객체를 저장할 수 있습니다.
  + 여기서는 postsService.findAllDesc()로 가져온 결과를 posts로 index.mustache에 전달합니다.


## 게시글 수정 및 삭제
1. 게시글 수정 머스테치 파일 생성(+수정 페이지에 삭제 버튼)
2. 게시글 수정/삭제 API 기능(js) 
3. IndexController 수정 URL
4. PostsService, PostsApiController 삭제 기능 추가

### 1. 게시글 수정 머스테치 파일 생성(+수정 페이지에 삭제 버튼)
templates 아래에 __posts-update.mustache 파일을 생성__ 합니다.
* posts-update.mustache

```
{.{>layout/header}.}

    <h1>게시글 수정</h1>
    <div class="col-md-12">
        <div calss="col-md-4">
            <form>
                <div class="form-group">
                    <label for="id">글 번호</label>
                    <input type="text" class="form-control" id="id" value="{.{post.id}.}" readonly>
                </div>
                <div class="form-group">
                    <label for="title">제목</label>
                    <input type="text" class="form-control" id="title" value="{.{post.title}.}">
                </div>
                <div class="form-group">
                    <label for="author">작성자</label>
                    <input type="text" class="form-control" id="author" value="{.{post.author}.}" readonly>
                </div>
                <div class="form-group">
                    <label for="content">내용</label>
                    <textarea class="form-control" id="content">{.{post.content}.}</textarea>
                </div>
            </form>
            <a href="/" role="button" class="btn btn-secondary">취소</a>
            <button type="button" class="btn btn-primary" id="btn-update">수정 완료</button>
            <button type="button" class="btn btn-danger" id="btn-delete">삭제</button>
        </div>
    </div>

{.{>layout/footer}.}
```

* `{.{post.id}.}`
  + 머스테치는 객체의 필드 접근 시 점(Dot)으로 구분합니다.
  + 즉, Post 클래스의 id에 대한 접근은 post.id로 사용할 수 있습니다.
* `readonly`
  + input 태그에 읽기 가능만 허용하는 속성입니다.

### 2. 게시글 수정/삭제 API 기능(js) 

* index.js

```
var main = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
            _this.save();
        });

        $('#btn-update').on('click', function () { // chap4.5 (1)
            _this.update();
        });

        $('#btn-delete').on('click', function () {
            _this.delete();
        });
    },
    save : function () {
        var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };
        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType: 'application/json; charset-utf-8',
            data: JSON.stringify(data)
        }).done(function () {
            alert('글이 등록되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    update : function () { // (2)
        var data = {
            title: $('#title').val(),
            content: $('#content').val()
        };

        var id = $('#id').val();

        $.ajax({
            type: 'PUT', // (3)
            url: '/api/v1/posts/'+id, // (4)
            dataType: 'json',
            contentType: 'application/json; charset-utf-8',
            data: JSON.stringify(data)
        }).done(function () {
            alert('글이 수정되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    delete : function () {
        var id = $('#id').val();

        $.ajax({
            type: 'DELETE',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType: 'application/json; charset-utf-8',
        }).done(function () {
            alert('글이 삭제되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    }
};

main.init();
```

* (1) `$('#btn-update').on('click')`
  + btn-update란 id를 가진 HTML 엘리먼트에 click 이벤트가발생할 때 update function을 실행하도록 이벤트를 등록합니다.
* (2) `update : function ()`
  + 신규로 추가될 update function 입니다.
* (3) `type: 'PUT'`
  + 여러 HTTP Method 중 PUT 메소드를 선택합니다.
  + PostApiController에 있는 API에서 이미 @PutMapping으로 선언했기 떄문에 PUT을 사용해야 합니다. 참고로 이는 REST 규약에 맞게 설정된 것입니다.
  + REST에서 CRUD는 다음과 같이 HTTP Method에 매핑 됩니다.
    - __생성(Create)__ - POST
    - __읽기(Read)__ - GET
    - __수정(Update)__ - PUT
    - __삭제(Delete)__ - DELETE
* (4) `url: '/api/v1/posts/'+id`
  + 어느 게시글을 수정할지 URL Path로 구분하기 위해 Path에 id를 추가합니다.

* index.mustache 내용 수정

```
    <tbody id="tbody">
        {.{#posts}.}
            <tr>
                <td>{.{id}.}</td>
                <td><a href="/posts/update/{.{id}.}">{.{title}.}</a></td>
                <td>{.{author}.}</td>
                <td>{.{modifiedDate}.}</td>
            </tr>
        {.{/posts}.}
    </tbody>
```

### 3. IndexController 수정 URL

* IndexController

```
package com.doop.book.springboot.web;

...
public class IndexController {

    ...

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model) {
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post", dto);

        return "posts-update";
    }
}
```

### 4. PostsService, PostsApiController 삭제 기능 추가

* PostsService

```
...
public class PostsService {

    ...

    @Transactional
    public void delete(Long id) {
        Posts posts = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다.id=" + id ));
        postsRepository.delete(posts); // (1)
    }
}
```

* (1) `postsRepository.delete(posts)`
  + JpaRepository에서 이미 delete 메소드를 지원하고 있으니 이를 활용합니다.
  + 엔티티를 파라미터로 삭제할 수도 있고, deletById 메소드를 이용하면 id로 삭제할 수도 있습니다.
  + 존재하는 Posts인지 확인을 위해 엔티티 조회 후 그대로 삭제합니다.


* PostsApiController

```
...
public class PostsApiController {
    ...

    @DeleteMapping("/api/v1/posts/{id}")
    public Long delete(@PathVariable Long id) {
        postsService.delete(id);
        return id;
    }
}
```

# References
* 스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)