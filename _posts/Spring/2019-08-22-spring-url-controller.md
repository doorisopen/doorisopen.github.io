---
layout: post
title:  "[Spring] 6 - Controller 구현 예시"
date:   2019-08-22 15:51:40
author: me
categories: Spring
tags: Spring Framework web MVC Controller
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

>> 해당 게시글의 전체 소스코드는 이곳 <a href="https://github.com/doorisopen/SpringSpring/tree/889c43dd7fa73497ad84d279c9cc1b281d05e5fa"><strong>Github</strong></a>를 참고해 주세요

<br />

### 목차
>> 1. __프리젠테이션 레이어 동작 방식과 컨트롤러 구현 방식을 이해__
>> 2. __HTTP Method GET, POST 동작 방식 이해__

<br />
<br />


<hr />


## Controller 구현 방법
* __MemberController.java__ 파일에 __@Controller__ 지정하면 Component-Scan 기능을 통해 __urlController를 빈__ 으로 등록

```
package org.doorisopen.myspring.Member.Controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;

@Controller
public class MemberController {
	
	private static final Logger logger = LoggerFactory.getLogger(MemberController.class);
	

}
```

### @RequestMapping
* __URL과 컨트롤러 메서드의 매핑을 설정하는 애노테이션__
* 속성 값을 사용해 URL 설정
<a href="{{ site.spring_img }}/spring_controller_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_controller_1.JPG" title="Check out the image">
</a>

#### 컨트롤러 메서드 매개변수
* Model 오브젝트
* @ModelAttribute 매개변수
* @PathVariable 매개변수
* @RequestParam 매개변수
* @MatrixVariable 매개변수
* @RequestBody 매개변수

* __urlControllerTest.java 내용__

```
package org.doorisopen.myspring;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class urlControllerTest {
	
	private static final Logger logger = LoggerFactory.getLogger(urlControllerTest.class);
	
	// Method 1
	// URL 경로 내의 변수 값을 @PathVariable 적용 변수로 전달: 
	// http://localhost:8080/myspring/try/Hello
	@RequestMapping(value="/try/{msg}", method = RequestMethod.GET)
	public String getUserTest( @PathVariable("msg") String msg) {
		logger.info(msg);
		logger.info(" /try URL called. then getUserTest method executed.");
		return "/urlControllerTest/result_a";
	}
	
	// Method 2
	// 요청 파라미터 값을 @RequestParam 적용 변수로 전달
	// http://localhost:8080/myspring/tryA?msg=ThankYou
	// 결과 : Result A: Hello
	// @RequestParam 적용 변수의 값이 출력 안됨
	@RequestMapping(value="/tryA", method = RequestMethod.GET)
	public String getUserTest1( @RequestParam("msg") String msg ) {
		logger.info(msg);
		logger.info(" /tryA URL called. then getUserTest1 method executed.");
		return "/urlControllerTest/result_a";
	}
	
	// Method 3
	// 요청 파라미터 값을 @ModelAttribute 적용 변수로 전달
	// http://localhost:8080/web/tryB?msg=thankyou
	// 결과 : Result A: Hello thankyou
	// @ModelAttribute 적용 변수의 값이 출력 된다
	@RequestMapping(value="/tryB", method = RequestMethod.GET)
	public String getUserTest2( @ModelAttribute("msg") String msg ) {
		logger.info(msg);
		logger.info(" /tryB URL called. then getUserTest2 method executed.");
		return "/urlControllerTest/result_a";
	}
	
	// Method 3
	// http://localhost:8080/web/tryC?msg=thankyouHelloWow
	// 요청 파라미터 값을 @ModelAttribute 적용 변수로 전달
	@RequestMapping(value={"/tryC", "/tryD"}, method = RequestMethod.GET)
	public String getUserTest3( @ModelAttribute("msg") String msg ) {
		logger.info(msg);
		logger.info(" /tryB URL called. then getUserTest3 method executed.");
		return "/urlControllerTest/result_a";
	}
}
```

* __/view/urlControllerTest/result_a.jsp 내용__

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>url Controller Test Page</title>
</head>
<body>
	<h2>url Controller Test Page</h2>
	<br/>
	<span>Result A : Hello ${msg}</span>
</body>
</html>
```


<hr />


## HTTP Method GET, POST 테스트
* __src/test/java/org.doorisopen.myspring.Test.Board/BoardTest.java의 내용__

```
package org.doorisopen.myspring.Test.Board;

import org.doorisopen.myspring.urlControllerTest;
import org.doorisopen.myspring.Board.Domain.BoardVO;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
public class BoardTest {
	
	private static final Logger logger = LoggerFactory.getLogger(urlControllerTest.class);
	
	// GET, POST URL TEST
	
	// GET Test
	// http://localhost:8080/myspring/BoardPage
	@RequestMapping(value="/BoardPage", method = RequestMethod.GET)
	public String BoardCreatePage() {

		logger.info(" /BoardPage URL GET method called. then BoardCreatePage method executed.");
		return "/urlControllerTest/BoardCreate";
	}
	
	// POST Test
	// http://localhost:8080/myspring/BoardCreate
	@RequestMapping(value="/BoardCreate", method = RequestMethod.POST)
	public String BoardCreate(@ModelAttribute("board") BoardVO vo) {
		logger.info(vo.toString());
		logger.info(" /BoardCreate URL POST method called. then BoardCreate method executed.");
		return "/urlControllerTest/BoardResult";
	}
	
}
```

* __/view/urlControllerTest/BoardCreate.jsp 내용__

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Board Create</title>
</head>
<body>
	<div align=center>
	 <header>게시글 등록하기</header>
		<form action="/myspring/BoardCreate" method="POST">
			<table>
				<tr>
					<th>게시글 제목</th>
					<td><input type="text" name="boardTitle" autofocus placeholder="공백없이 입력하세요"/></td>
				</tr>
				<tr>
					<th>게시글 내용</th>
					<td><input type="text" name="boardContent" placeholder="공백없이 입력하세요"/></td>
				</tr>
				<tr>
					<th>게시글 작성자</th>
					<td><input type="text" name="writer" placeholder="공백없이 입력하세요"/></td>
				</tr>
			</table>
			<div>
				<input type="submit" name="submit" value="등록" />
				<input type="reset" name="reset" value="재 작성" />
			</div>
		</form>	
	</div>
</body>
</html>
```

* __/view/urlControllerTest/BoardResult.jsp 내용__

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
   
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Board Result</title>
</head>
<body>
	<div align=center>
		<header>게시글 등록 결과</header>
		<c:if test="${board != null }">
			<table>
				<tr>
					<th>게시글 제목</th>
					<td><c:out value="${board.boardTitle}"/></td>
				</tr>
				<tr>
					<th>게시글 내용</th>
					<td><c:out value="${board.boardContent}"/></td>
				</tr>
				<tr>
					<th>게시글 작성자</th>
					<td><c:out value="${board.writer}"/></td>
				</tr>
			</table>
		
		
		</c:if>
	</div>
</body>
</html>
```

* __GET 결과__
<a href="{{ site.spring_img }}/spring_controller_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_controller_2.JPG" title="Check out the image">
</a>

* __POST 결과__
<a href="{{ site.spring_img }}/spring_controller_3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_controller_3.JPG" title="Check out the image">
</a>

<hr />

### References
* 대학교 웹 프레임워크 수업 정리한 내용 입니다.