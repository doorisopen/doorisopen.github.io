---
layout: post
title:  "[Spring] 11 - 게시판 만들기① (CRUD JUnit Test)"
date:   2019-09-03 10:07:40
author: me
categories: Spring
tags: Spring Framework web Board CRUD JUnit
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

>> 해당 게시글의 전체 소스코드는 이곳 <a href="https://github.com/doorisopen/SpringSpring/tree/a35c247f378afe65b935df99520d57f00d2d4d7c"><strong>Github</strong></a>를 참고해 주세요

<br />

### 목차
>> 1. __게시판 CRUD 코드 작성 후 JUnit Test 하기__

<br />
<br />


<hr />

### Board Table 생성하기(MySQL)

```
CREATE TABLE myspring.board (
	 boardIdx int not null auto_increment,
     boardTitle varchar(255) not null,
     boardContent text not null,
     writer varchar(255) not null,
     writeDate datetime not null default now(),
     modifier varchar(255),
     modifyDate datetime,
     important int not null default 0,
     enabled int not null default 1,
     PRIMARY KEY(boardIdx)
);
```

### VO(Value Object) 빈 클래스 생성하기
VO(Value Object)는 __DTO(Data Transfer Object)__ 로 바꿔 말할 수 있다 그러나 VO는 DTO와 다르게 read only 속성을 가진다
* __DTO(Data Transfer Object)는 계층간 데이터 교환을 위한 자바 빈즈를 말한다__ 
  + 여기서 말하는 계층간의 Controller, View, Business, Persistence 계층을 말함

* __VO DTO 비교__
  + __VO :__ 사용 되는 값이 객체로 표현 되며, 값 변경이 없는 경우를 말한다.
  + __DTO :__ 데이터의 전송을 위한 객체이며, 비지니스 로직까지 담아서 사용하기 한다.

```
package org.doorisopen.myspring.Board.Domain;

public class BoardVO {
	
	private int boardIdx;        // 게시글 Index
	private String boardTitle;   // 게시글 제목
	private String boardContent; // 게시글 내용
	private String writer;       // 게시글 작성자
	private String writeDate;    // 게시글 작성 일시
	private String modifier;     // 게시글 수정자
	private String modifyDate;   // 게시글 수정 일시
	private int important;       // 게시글 중요도 0||1
	private int enabled;         // 게시글 사용 여부 0||1
	
	
	public int getBoardIdx() {
		return boardIdx;
	}

	public void setBoardIdx(int boardIdx) {
		this.boardIdx = boardIdx;
	}

	public String getBoardTitle() {
		return boardTitle;
	}

	public void setBoardTitle(String boardTitle) {
		this.boardTitle = boardTitle;
	}

	public String getBoardContent() {
		return boardContent;
	}

	public void setBoardContent(String boardContent) {
		this.boardContent = boardContent;
	}

	public String getWriter() {
		return writer;
	}

	public void setWriter(String writer) {
		this.writer = writer;
	}

	public String getWriteDate() {
		return writeDate;
	}

	public void setWriteDate(String writeDate) {
		this.writeDate = writeDate;
	}

	public String getModifier() {
		return modifier;
	}

	public void setModifier(String modifier) {
		this.modifier = modifier;
	}

	public String getModifyDate() {
		return modifyDate;
	}
	
	public void setModifyDate(String modifyDate) {
		this.modifyDate = modifyDate;
	}

	public int getImportant() {
		return important;
	}

	public void setImportant(int important) {
		this.important = important;
	}

	public int getEnabled() {
		return enabled;
	}

	public void setEnabled(int enabled) {
		this.enabled = enabled;
	}


	// DI XML Test by doorisopen
	public String toString() {
		return "BoardVO [boardIdx="+boardIdx+" boardTitle="+boardTitle+" boardContent="+boardContent+" writer="+writer+"]";
	}
}

```

<hr />


### Service 생성하기
Business 계층을 의미한다

* __BoardService.java__

```
package org.doorisopen.myspring.Board.Service;

import java.util.List;

import org.doorisopen.myspring.Board.Domain.BoardVO;

public interface BoardService {
	
	public int BoardCreate(BoardVO vo) throws Exception;
	public List<BoardVO> BoardRead(BoardVO vo) throws Exception;
	public BoardVO BoardDetail(int boardIdx) throws Exception;
	public int BoardUpdate(BoardVO vo) throws Exception;
	public int BoardDelete(int boardIdx) throws Exception;
}
```

* __BoardServiceImpl.java__

```
package org.doorisopen.myspring.Board.Service;

import java.util.List;

import org.doorisopen.myspring.Board.Domain.BoardVO;
import org.doorisopen.myspring.Board.Persistence.BoardDAO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BoardServiceImpl implements BoardService{

	@Autowired
	private BoardDAO dao;

	
	/* 게시글 작성
	 * 
	 * 
	 */
	@Override
	public int BoardCreate(BoardVO vo) throws Exception {
		// TODO Auto-generated method stub
		return dao.BoardCreate(vo);
	}

	/* 게시글 리스트
	 * 
	 * 
	 */
	@Override
	public List<BoardVO> BoardRead(BoardVO vo) throws Exception {
		// TODO Auto-generated method stub
		return dao.BoardRead(vo);
	}
	
	/* 게시글 상세
	 * 
	 * 
	 */
	@Override
	public BoardVO BoardDetail(int boardIdx) throws Exception {
		// TODO Auto-generated method stub
		return dao.BoardDetail(boardIdx);
	}

	/* 게시글 수정
	 * 
	 * 
	 */
	@Override
	public int BoardUpdate(BoardVO vo) throws Exception {
		// TODO Auto-generated method stub
		return dao.BoardUpdate(vo);
	}

	/* 게시글 삭제
	 * 
	 * 
	 */
	@Override
	public int BoardDelete(int boardIdx) throws Exception {
		// TODO Auto-generated method stub
		return dao.BoardDelete(boardIdx);
	}	
}
```

<hr />

### DAO(Data Access Object) 생성하기
__DAO(Data Access Object)__ 는 DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트를 말한다

* __BoardDAO.java__

```
package org.doorisopen.myspring.Board.Persistence;

import java.util.List;
import org.doorisopen.myspring.Board.Domain.BoardVO;

public interface BoardDAO {
	
	public int BoardCreate(BoardVO vo) throws Exception;
	public List<BoardVO> BoardRead(BoardVO vo) throws Exception;
	public BoardVO BoardDetail(int boardIdx) throws Exception;
	public int BoardUpdate(BoardVO vo) throws Exception;
	public int BoardDelete(int boardIdx) throws Exception;
}
```

* __BoardDAOImpl.java__

```
package org.doorisopen.myspring.Board.Persistence;

import java.util.ArrayList;
import java.util.List;

import org.apache.ibatis.session.SqlSession;
import org.doorisopen.myspring.Board.Domain.BoardVO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class BoardDAOImpl implements BoardDAO{

	@Autowired
	private SqlSession sqlSession;
	
	private static final String namespace ="org.doorisopen.myspring.Board.BoardMapper";
	
	/* 게시글 작성
	 * 
	 * 
	 */
	@Override
	public int BoardCreate(BoardVO vo) throws Exception {
		// TODO Auto-generated method stub
		return sqlSession.insert(namespace + ".BoardCreate", vo);
	}

	/* 게시글 리스트
	 * 
	 * 
	 */
	@Override
	public List<BoardVO> BoardRead(BoardVO vo) throws Exception {
		// TODO Auto-generated method stub
		List<BoardVO> BoardRead = new ArrayList<BoardVO>();
		BoardRead = sqlSession.selectList(namespace + ".BoardRead", vo);
		return BoardRead;
	}
	
	/* 게시글 상세
	 * 
	 * 
	 */
	@Override
	public BoardVO BoardDetail(int boardIdx) throws Exception {
		// TODO Auto-generated method stub
		BoardVO vo = sqlSession.selectOne(namespace + ".BoardDetail", boardIdx);
		
		return vo;
	}

	/* 게시글 수정
	 * 
	 * 
	 */
	@Override
	public int BoardUpdate(BoardVO vo) throws Exception {
		// TODO Auto-generated method stub
		return sqlSession.update(namespace + ".BoardUpdate", vo);
	}

	/* 게시글 삭제
	 * 
	 * 
	 */
	@Override
	public int BoardDelete(int boardIdx) throws Exception {
		// TODO Auto-generated method stub
		return sqlSession.delete(namespace + ".BoardDelete", boardIdx);
	}
}
```

<hr />

### Mapper 작성하기 
* __Path: src/main/resources/mappers/BoardMapper.xml__
Mapper는 DB에 접근하여 데이터를 조회, 조작을 위한 쿼리문을 작성하기 위한 파일이다

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  
<mapper namespace="org.doorisopen.myspring.Board.BoardMapper">

	<!-- 게시글 등록 -->
	<insert id="BoardCreate" parameterType="BoardVO">
	INSERT INTO
	board 
	(
		boardTitle,
		boardContent,
		writer,
		writeDate
	) VALUES (
		#{boardTitle},
		#{boardContent},
		#{writer},
		now()
	)
	</insert>
	
    <!-- 게시글 리스트 -->
    <select id="BoardRead" resultType="BoardVO">
 	 <![CDATA[
		SELECT
			*
		FROM
			board
		WHERE
			enabled = '1'
 	 ]]> 
 	</select>
    
    <!-- 게시글 조회 -->
    <select id="BoardDetail" resultType="BoardVO">
    	SELECT
    		*
    	FROM
    		board
    	WHERE
    		boardIdx = #{boardIdx}
 	</select>
 	
    <!-- 게시글 수정 -->
    <update id="BoardUpdate">
    	UPDATE
    		board
    	SET
    		boardTitle = #{boardTitle}
    		, boardContent = #{boardContent}
			, modifier = #{modifier}
			, modifyDate = now()
		WHERE
			boardIdx = #{boardIdx}
 	</update>
 	
    <!-- 게시글 삭제 -->
    <delete id="BoardDelete">
    	UPDATE
    		board
    	SET
    		enabled = '0'
    	WHERE
    		boardIdx = #{boardIdx}
 	</delete>
 	
</mapper>
```

<hr />

### mybatis-config.xml 설정하기
* __Path: src/main/resources/mybatis-config.xml__
MyBatis의 SqlSessionFactory 빈을 등록할 때 설정파일로 등록한 mybatis-config.xml에 아래와 같이 도메인을 등록하여 Mapper파일과 매핑이 가능하게 해준다

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
  
<configuration>
  <typeAliases>
        <typeAlias type="org.doorisopen.myspring.Member.Domain.MemberVO" alias="MemberVO"/> 
        <typeAlias type="org.doorisopen.myspring.Board.Domain.BoardVO" alias="BoardVO"/>    
  </typeAliases>   
</configuration>
```

### Test Case 작성하기

```
package org.doorisopen.myspring.Test.Board;

import org.doorisopen.myspring.Board.Domain.BoardVO;
import org.doorisopen.myspring.Board.Service.BoardService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.ui.Model;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "file:src/main/webapp/WEB-INF/spring/root-context.xml" })
public class BoardCRUDTest {
	
	@Autowired
	BoardService service;
	
	
	/* TEST 1. 게시글 등록 
	 * 
	 *
	 */
	// Test 가 필요한곳에 아래의 태그 입력한다.
	@Test
	public void boardCreateTest() {
		System.out.println("This is boardCreateTest...");
		BoardVO vo = new BoardVO();
		
		vo.setBoardTitle("First Board Test");
		vo.setBoardContent("Hello This is Spring Board Create Test");
		vo.setWriter("gangnam");
		
		try {
			service.BoardCreate(vo);
			System.out.println("Success");
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	/* TEST 2. 게시글 리스트 
	 * 
	 *
	 */
	public void boardReadTest() {
		System.out.println("This is boardReadTest...");
		BoardVO vo = new BoardVO();
		
		try {
			service.BoardRead(vo);
			System.out.println(service.BoardRead(vo));
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	/* TEST 3. 게시글 수정 
	 * 
	 *
	 */
	public void boardUpdateTest() {
		System.out.println("This is boardUpdateTest...");
		BoardVO vo = new BoardVO();
		
		int boardIdx = 2;
		vo.setBoardIdx(boardIdx);
		vo.setBoardTitle("Board Update Test");
		vo.setBoardContent("Goodbye This is Spring Board Update Test");
		vo.setModifier("admin");
		
		try {
			service.BoardUpdate(vo);
			System.out.println(service.BoardRead(vo));
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	/* TEST 4. 게시글 삭제 
	 * 
	 *
	 */
	public void boardDeleteTest() {
		System.out.println("This is boardDeleteTest...");
		BoardVO vo = new BoardVO();
		
		int boardIdx = 2;
		
		try {
			service.BoardDelete(boardIdx);
			System.out.println(service.BoardRead(vo));
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

}
```

<hr />


### References
* 대학교 웹 프레임워크 수업 정리한 내용 입니다.