---
layout: post
title:  "[Spring] 13 - 게시판 만들기③ (조회수)"
date:   2020-01-26 12:25:40
author: me
categories: Spring
tags: Spring Framework web Board CRUD MVC
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

>> 해당 게시글의 전체 소스코드는 이곳 <a href="https://github.com/doorisopen/SpringSpring/tree/0719d8aa98475ec0fbeb8b9fb6e6f548f3d70558"><strong>Github</strong></a>를 참고해 주세요

<br />

### 목차
>> 1. __게시글 조회수 구현__


<br />
<br />


<hr />

게시글을 클릭 할때 조회수가 증가하는 기능을 구현해보려고 한다

조회수 기능을 구현하는것은 아주 간단하다

* __BoardController__

```
    /* 
	 * 게시글 상세
	 */
	@RequestMapping(value = "/boardDetail", method = RequestMethod.GET)
	public String BoardDetail(Model model, 
                                BoardVO vo, 
                                @ModelAttribute("boardIdx") int boardIdx) throws Exception {
        
        ...(생략)

	}
```
게시글 상세내용을 보여주는 주소인 __"/boardDetail"__ 이 호출될때 조회수를 +1 이 더해지게 구현하면 된다

우선 board 테이블의 __게시글 조회수__ 컬럼을 추가한다

> __ALTER TABLE board ADD column boardViewCnt int not null default 0;__

그리고 Domain 아래의 BoardVO에 __boardViewCnt__ 를 추가해준다 (getter, setter)

### Domain
__/src/main/java/org/doorisopen/myspring/Board/Domain/BoardVO.java__

```
private int boardIdx;
	private String boardTitle;
	private String boardContent;
	private String writer;
	private String writeDate;
	private String modifier;
	private String modifyDate;
	private int important;
	private int enabled;
	private int boardViewCnt; // 게시글 조회수 추가

    ...이하생략...

```
### Persistence

조회수 기능은 __BoardService(Business layer)__ 에서 수행되면된다 

우선 __BoardDAO 인터페이스__ 에 아래의 코르를 작성한다.

__/src/main/java/org/doorisopen/myspring/Board/Persistence/BoardDAO.java__
```
// BoardDAO.java
public interface BoardDAO {
    ...(생략)...
    public void BoardViewCntUpdate(int boardIdx) throws Exception;

}    
```
그리고 BoardDAOImpl.java에 아래의 코드를 추가한다

__/src/main/java/org/doorisopen/myspring/Board/Persistence/BoardDAOImpl.java__
```
@Component
public class BoardDAOImpl implements BoardDAO{

	@Autowired
	private SqlSession sqlSession;
	
	private static final String namespace ="org.doorisopen.myspring.Board.BoardMapper";
    
    ...(생략)...
    
    /* 
	 * 게시글 조회수 증가
	 */
	@Override
	public void BoardViewCntUpdate(int boardIdx) throws Exception {
		// TODO Auto-generated method stub
		sqlSession.update(namespace + ".BoardViewCntUpdate", boardIdx);
	}
}
```

### Mapper(SQL)
__/src/main/resources/mappers/BoardMapper.xml__
```
<!-- 게시글 조회수 증가 -->
 	<update id="BoardViewCntUpdate">
    	UPDATE
    		board
    	SET
    		boardViewCnt = boardViewCnt + 1
		WHERE
			boardIdx = #{boardIdx}
 	</update>
```


### Service
마지막으로 __BoardServiceImpl__ 에서 __BoardDetail__ 에 게시글 조회수 증가하는 코드를 작성하면 끝난다

__/src/main/java/org/doorisopen/myspring/Board/Service/BoardServiceImpl.java__
```
    /* 
	 * 게시글 상세
	 */
	@Override
	public BoardVO BoardDetail(int boardIdx) throws Exception {
		// TODO Auto-generated method stub
		
		// 게시글 조회수 업데이트
		dao.BoardViewCntUpdate(boardIdx); // <---- 추가
		
		return dao.BoardDetail(boardIdx);
	}
```

추가로 BoardRead(List) 에 아래와 같이 작성해주면 게시글 조회수 구현이 끝이 난다

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
		<table border="1">
			<thead>
				<tr>
					<th colspan="2">NO</th>
					<th colspan="10">Title</th>
					<th colspan="5">Writer</th>
					<th colspan="5">WriteDate</th>
					<th colspan="2">View</th>
				</tr>
			</thead>
			<tbody>
			<c:choose>
				<c:when test="${fn:length(boardRead) > 0}">
					<c:forEach items="${boardRead }" var="boardRead" varStatus="rowcnt">
						<tr>
							<td colspan="2">${boardRead.boardIdx}</td>
							<td colspan="10"><a href="/myspring/Board/boardDetail?boardIdx=${boardRead.boardIdx}">${boardRead.boardTitle}</a></td>
							<td colspan="5">${boardRead.writer}</td>
							<td colspan="5">${boardRead.writeDate}</td>
							<td colspan="2">${boardRead.boardViewCnt}</td>
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

> 게시글 조회수 구현 자체는 아주 간단한 내용이다. 그러나 게시글 조회수 중복증가라는 문제점이 있다. 이 문제는 추후 추가 해볼 것이다


### References
* 대학교 웹 프레임워크 수업 정리한 내용 입니다.