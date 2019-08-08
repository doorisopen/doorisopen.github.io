---
layout: post
title:  "[Spring]1.개발 환경 설정"
date:   2019-08-08 10:36:59
author: me
categories: Spring
tags: Spring Framework web 
cover:  "/assets/instacode.png"
---

### 프로젝트 생성 하기
* New -> Spring Legacy Project -> Spring MVC Project 
<a href="{{ site.spring_img }}/spring_new_project.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_new_project.JPG" title="Check out the image">
</a>


<hr />


### 프로젝트 초기 설정하기

* __Properties:__ Project 마우스 오른쪽 클릭 
<a href="{{ site.spring_img }}/spring_project_setting.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_project_setting.JPG" title="Check out the image">
</a>

1. Properties -> Resource -> Text file encoding -> Other(UTF-8)
2. Properties -> Java Build Path -> JRE System Library [JavaSE-1.8]
3. Properties -> Project Facets -> Java -> 1.8

<hr />


### Eclipse-Github 연동하기

* STS의 경우 Git이 설치되어있기 때문에 별도의 설치가 필요없다. 만약 없을 경우 Egit을 마켓에서 설치하면된다.

* Window -> Perspective -> Open Perspective -> Other -> Git 을 선택한다.
* 기존에 깃 저장소가 있으면 아래와 같이 뜬다. __Add an existing local Git repository__ 선택
<a href="{{ site.spring_img }}/github_setting_0.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/github_setting_0.JPG" title="Check out the image">
</a>

* 개인 Github URI와 아이디 패스워드를 입력해준다. port는 입력하지 않아도 된다.
<a href="{{ site.spring_img }}/github_setting_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/github_setting_1.JPG" title="Check out the image">
</a>

* Github URI는 프로젝트를 업로드할 github 저장소의 Clone or download 버튼을 눌러 URL를 복사후 붙여넣으면 된다.
<a href="{{ site.spring_img }}/github_setting_1_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/github_setting_1_2.JPG" title="Check out the image">
</a>

* 기본 Branch master를 선택하고 넘어간다.
<a href="{{ site.spring_img }}/github_setting_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/github_setting_2.JPG" title="Check out the image">
</a>

* Git이 연동될 Local 저장소를 지정해준다.
  + 해당 프로젝트를 Commit 하게 될경우 Local 저장소에 Update가 되고 push 하게 될 경우 원격저장소로 Update가 된다.
<a href="{{ site.spring_img }}/github_setting_3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/github_setting_3.JPG" title="Check out the image">
</a>

* 위의 과정으로 Git 세팅은 끝이 났다. 다음으로는 소스를 업로드하는 방법이다.

* 업로드 하려는 프로젝트에 오른쪽 클릭하여 __Team -> Share Project__ 선택하면 아래와 같은 화면이 뜬다.
* 저장소를 선택하면 로컬 저장소 목록이 뜬다. 프로젝트의 로컬 저장소를 선택한다. Finish를 누르면 로컬 저장소와 함께 동기화 된다. 실제로 깃에 업로드 된것은 아니며 로컬 저장소에 Commit 후 Update가 가능하다.
<a href="{{ site.spring_img }}/github_setting_4.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/github_setting_4.JPG" title="Check out the image">
</a>

* 프로젝트에 Add to Index를 선택하면 프로젝트 파일에 있는 물음표가 사라진다.
<a href="{{ site.spring_img }}/github_setting_5.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/github_setting_5.JPG" title="Check out the image">
</a>

* 그리고 Commit 을 누른다.
<a href="{{ site.spring_img }}/github_setting_6.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/github_setting_6.JPG" title="Check out the image">
</a>

* Commit 을 누르면 아래와 같이 Commit Message를 입력해준다. 그리고 Commit and Push 버튼을 누른다. 그러면 로컬 저장소에 Commit이 되고 바로 원격 저장소로 push가 된다.
  + 만약 Commit 버튼을 누른다면 로컬 저장소에만 Commit 된다. 
<a href="{{ site.spring_img }}/github_setting_7.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/github_setting_7.JPG" title="Check out the image">
</a>

* Commit and Push 버튼을 누르고 기다리면 아래와 같은 창이 뜨는데 이는 github에 push 한 결과 정보를 확인 할 수 있다.
<a href="{{ site.spring_img }}/github_setting_8.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/github_setting_8.JPG" title="Check out the image">
</a>


### 관련된 Post


<hr />


### References
> * <a href=""><a>
