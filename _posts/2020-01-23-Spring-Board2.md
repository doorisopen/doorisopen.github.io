---
layout: post
title:  "[Spring] 12 - 게시판 만들기② (MVC)"
date:   2020-01-25 00:07:40
author: me
categories: Spring
tags: Spring Framework web Board CRUD MVC
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

>> 해당 게시글의 전체 소스코드는 이곳 <a href="https://github.com/doorisopen/SpringSpring/tree/4be848ca9750202f2b27ecaacaf76eec0a8131e2"><strong>Github</strong></a>를 참고해 주세요

<br />

### 목차
>> 1. __Controller 작성__
>> 2. __화면 만들기__


<br />
<br />


<hr />

이전 포스트에 이어서 게시판 결과 화면을 처리 하기 위한 Controller를 작성해보고 View(jsp) 코드를 작성해 볼 것이다. 

### Controller 작성

```
// Path : /src/main/java/org/doorisopen/myspring/Board/Controller/BoardController.java
@Controller
@RequestMapping(value="/Board")
public class BoardController {
	
	private static final Logger logger = LoggerFactory.getLogger(BoardController.class);
	
	@Autowired
	private BoardService service;
	
	/* 
	 * 게시글 작성 화면
	 */
	@RequestMapping(value="/boardCreateView", method = RequestMethod.GET)
	public String BoardCreateView() throws Exception {
		
		
		return "/Board/boardCreate";
	}
	
	/* 
	 * 게시글 작성
	 */
	@RequestMapping(value = "/boardCreate", method = RequestMethod.POST)
	public String BoardCreate(HttpServletRequest request, @ModelAttribute BoardVO vo) throws Exception {
		
		service.BoardCreate(vo);
		
		return "redirect:/Board/boardRead";
	}
	
	/* 
	 * 게시글 리스트
	 */
	@RequestMapping(value = "/boardRead", method = RequestMethod.GET)
	public String BoardRead(Model model,BoardVO vo) throws Exception {

		System.out.println(service.BoardRead(vo));
		
		List<BoardVO> boardRead = service.BoardRead(vo);
		model.addAttribute("boardRead", boardRead);
		
		
		return "/Board/boardRead";
	}
	
	
	/* 
	 * 게시글 상세
	 */
	@RequestMapping(value = "/boardDetail", method = RequestMethod.GET)
	public String BoardDetail(Model model, BoardVO vo, @ModelAttribute("boardIdx") int boardIdx) throws Exception {
		
		vo = service.BoardDetail(boardIdx);
		model.addAttribute("boardDetail", vo);
		return "/Board/boardDetail";
	}
	
	
	/* 
	 * 게시글 수정 화면
	 */
	@RequestMapping(value = "/boardUpdateView", method = RequestMethod.GET)
	public String BoardUpdateView(Model model, BoardVO vo, @ModelAttribute("boardIdx") int boardIdx) throws Exception {
		vo = service.BoardDetail(boardIdx);
		model.addAttribute("boardUpdate", vo);
		return "/Board/boardUpdate";
	}
	
	/* 
	 * 게시글 수정
	 */
	@RequestMapping(value = "/boardUpdate", method = RequestMethod.POST)
	public String BoardModify(@ModelAttribute BoardVO vo, @ModelAttribute("boardIdx") int boardIdx) throws Exception {
		service.BoardUpdate(vo);
		
		return "redirect:/Board/boardDetail";
	}
	
	/*  
	 * 게시글 삭제
	 */
	@RequestMapping(value = "/boardDelete", method = RequestMethod.GET)
	public String BoardDelete(@ModelAttribute("boardIdx") int boardIdx) throws Exception {
		service.BoardDelete(boardIdx);
		return "redirect:/Board/boardRead";
	}

}
```

## 화면 만들기
경로: __/src/main/webapp/WEB-INF/views/Board/__

### 1. 게시글 리스트 화면
__boardRead.jsp__
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>DOOP</title>
</head>
<body>
	<!-- div center -->
	<div align=center>
	 <header>게시글 리스트</header>
 
	 	<!-- Board List -->
		<table>
			<thead>
				<tr>
					<th>NO</th>
					<th>Title</th>
					<th>Writer</th>
					<th>WriteDate</th>
				</tr>
			</thead>
			<tbody>
			<c:choose>
				<c:when test="${fn:length(boardRead) > 0}">
					<c:forEach items="${boardRead }" var="boardRead" varStatus="rowcnt">
						<tr>
							<td>${boardRead.boardIdx}</td>
							<td><a href="/myspring/Board/boardDetail?boardIdx=${boardRead.boardIdx}">${boardRead.boardTitle}</a></td>
							<td>${boardRead.writer}</td>
							<td>${boardRead.writeDate}</td>
						</tr>
					</c:forEach>
				</c:when>
				<c:otherwise>
					<tr>
						<td colspan="6">조회된 결과가 없습니다.</td>
					</tr>
				</c:otherwise>
			</c:choose>
			</tbody>
		</table>
		<!-- ./Board List -->
		<div>
			<a href="/myspring/Board/boardCreateView">게시글 등록하기</a>
		</div>
		
	</div>
	<!-- ./div center -->
</body>
</html>
```

### 2. 게시글 작성 화면
__boardCreate.jsp__
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>DOOP</title>
</head>
<body>
	<div align=center>
	 <header>게시글 등록하기</header>
		<form action="/myspring/Board/boardCreate" method="POST">
			<table>
				<tr>
					<th>게시글 제목</th>
					<td><input type="text" name="boardTitle" id="boardTitle" autofocus placeholder="공백없이 입력하세요"/></td>
				</tr>
				<tr>
					<th>게시글 내용</th>
					<td><input type="text" name="boardContent" id="boardContent" placeholder="공백없이 입력하세요"/></td>
				</tr>
				<tr>
					<th>게시글 작성자</th>
					<td><input type="text" name="writer" id="writer" placeholder="공백없이 입력하세요"/></td>
				</tr>
			</table>
			<div>
				<input type="submit" name="submit" value="등록" />
				<input type="reset" name="reset" value="재 작성" />
			</div>
			<div>
				<a href="/myspring/Board/boardRead">게시글 리스트 가기</a>
			</div>
		</form>	
	</div>
</body>
</html>
```

### 3. 게시글 상세 화면
__boardDetail.jsp__
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>DOOP</title>
</head>
<body>
	<div align=center>
	 <header>게시글 상세보기</header>
			<table>
				<tr>
					<th>게시글 제목</th>
					<td><input type="text" id="boardTitle" name="boardTitle" value="${boardDetail.boardTitle}" placeholder="boardTitle" disabled /></td>
				</tr>
				<tr>
					<th>게시글 내용</th>
					<td><input type="text" id="boardContent" name="boardContent" value="${boardDetail.boardContent}" placeholder="boardContent" disabled /></td>
				</tr>
				<tr>
					<th>게시글 작성자</th>
					<td><input type="text" id="writer" name="writer" value="${boardDetail.writer}" placeholder="writer" disabled /></td>
					
				</tr>
			</table>
			<div>
				<a href="/myspring/Board/boardUpdateView?boardIdx=${boardDetail.boardIdx}">수정하기</a>
			</div>
			<div>
				<a href="/myspring/Board/boardRead">게시글 리스트 가기</a>
			</div>
			<div>
				<a href="/myspring/Board/boardDelete?boardIdx=${boardDetail.boardIdx}">게시글 삭제</a>
			</div>
	</div>
</body>
</html>
```

### 4. 게시글 수정 화면
__boardUpdate.jsp__
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>DOOP</title>
</head>
<body>
	<div align=center>
	 <header>게시글 수정하기</header>
	 	<form action="/myspring/Board/boardUpdate?boardIdx=${boardUpdate.boardIdx}" method="POST">
			<table>
				<tr>
					<th>게시글 제목</th>
					<td><input type="text" id="boardTitle" name="boardTitle" value="${boardUpdate.boardTitle}" placeholder="boardTitle" /></td>
				</tr>
				<tr>
					<th>게시글 내용</th>
					<td><input type="text" id="boardContent" name="boardContent" value="${boardUpdate.boardContent}" placeholder="boardContent" /></td>
				</tr>
				<tr>
					<th>게시글 작성자</th>
					<td><input type="text" id="writer" name="writer" value="${boardUpdate.writer}" placeholder="writer" /></td>
				</tr>
			</table>
			<div>
				<input type="submit" name="submit" value="수정" />
				<input type="reset" name="reset" value="재 작성" />
			</div>
		</form>
			<div>
				<a href="/myspring/Board/boardRead">게시글 리스트 가기</a>
			</div>
	</div>
</body>
</html>
```

### References
* 대학교 웹 프레임워크 수업 정리한 내용 입니다.