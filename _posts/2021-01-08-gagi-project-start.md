---
layout: post
title: "[가지마켓] #1 팀 프로젝트 시작"
date: 2021-01-08 00:50:59
author: me
categories: Project
tags: gagi
cover: "/assets/instacode.png"
---


> 중고 거래 플랫폼 **가지마켓 프로젝트** 를 소개합니다.<br/>
> 프로젝트 저장소는 [이곳](https://github.com/GagiMarket/gagi)에 확인할 수 있습니다.


## 팀 프로젝트 시작
프로젝트 첫 회의는 아래와 같은 내용을 다뤘습니다.

* 프로젝트 규칙 설정하기
* 프로젝트 최종 구현 목표 설정하기
* 1주차 구현 목표 설정하기
* Git

## 프로젝트 규칙 설정하기
프로젝트를 진행하면서 지켜야할 규칙을 설정하였습니다.

* 기능 구현 목록 작성하기
* git commit convention 준수하기
* code convention 준수하기
* 이슈(issue) 관리하기

## 프로젝트 최종 구현 목표 설정하기
약 8주간 진행되는 프로젝트이기 때문에 다소 작은 프로그램처럼 보일지라도 **도메인의 역할과 구현을 구분하고 학습을 얻어갈 수 있는 목표**를 설정하였습니다.

* 상품 기능
  + 상품 정보 등록, 수정, 삭제, 조회
* 회원 기능
  + 회원 정보 등록, 수정, 삭제
* 주문 기능
  + 주문, 예약, 취소
* 결제 기능(고민중)
  + 결제, 취소
* 할인 기능(고민중)

## 1주차 구현 목표 설정하기
1주차 구현 목표는 **상품(item) 기능**의 API를 설계 및 구현하는 것입니다.

구현할 상품 기능은 아래와 같습니다.

* 상품 정보 등록
  + 상품은 다음 정보를 포함합니다. (상품이름, 설명, 카테고리, 가격, 판매 지역, 이미지(추후 예정))
* 상품 정보 수정
* 상품 정보 삭제
* 상품 정보 조회

## Git
하나의 저장소에서 프로젝트를 관리 해야하기 때문에 기능 구현보다 **Git 학습의 필요성**을 느꼈습고 아래와 같은 내용을 학습하였습니다.

* issue 관리
* 일관된 git commit log 남기기(git convention)
* **git branch 전략**
* **git merge** (Pull Request 후 원본 저장소에 merge 하기)

#### git branch 전략
지금껏 프로젝트를 수행하면서 Git을 사용했지만 branch를 나눠 본적은 없었습니다. 

그러나 협업에 있어서 기능, 개발, 배포, 버그 별로 branch를 나누는 것의 중요성을 깨달아서 다음과 같은 계획을 세워보았습니다.

우선 저장소는 **원본 저장소**와 **개인 원격 저장소** 2가지가 존재할때 다음과 같은 branch가 있습니다.

* **원본 저장소/main:** 제품 출시 하는 메인 브랜치
* **원본 저장소/develop:** 다음 출시 버전을 개발하는 브랜치
* **개인 원격 저장소/main** 
* **개인 원격 저장소/develop**
* **개인 원격 저장소/feature/{issue number}:** 기능 개발을 위한 브랜치

처음에는 main과 develop 브랜치가 존재합니다. develop 브랜치는 main 브랜치에서 생성된 브랜치입니다.

이슈가 생길때 develop 브랜치에서 feature 브랜치를 생성합니다.

그리고 이슈 별 기능을 **개인 원격 저장소/feature/{issue number}** 해당 브랜치에서 개발합니다.

기능 구현 완료시 **원본 저장소/develop** <- **개인 원격 저장소/feature/{issue number}** 로 Pull Request를 생성합니다.

Pull Request에 대한 리뷰가 끝나면 Merge(Squash)를 수행합니다.

#### git merge (Pull Request 후 원본 저장소에 merge 하기)
git merge에는 3가지 전략 **Create a merge commit, Rebase and merge, Squash and merge** 이 있습니다.

이들의 차이점은 **Commit History**가 기록되는 방식이 달라진다는 차이점이 있습니다.

팀 프로젝트를 진행하면서 Pull Request 후 Merge를 수행하게 될텐데 이때 팀원들이 일관된 merge 전략을 가져가야 일관된 커밋 로그를 남길 수 있습니다.

첫 번째 **Create a merge commit** 를 선택한 경우엔 "Merge pull request #1 from branch-name" 과 같은 메시지와 함게 커밋 로그가 남겨집니다.

두 번쨰 **Rebase and merge** 는 PR의 커밋 로그들이 master에 재정렬돼서 merge를 수행합니다. PR에서 작성한 모든 커밋들이 master에서 관리돼야 한다면 이 방법을 추천합니다.

마지막은 **Squash and merge** 인데 저의 경우엔 이 방법은 선택해서 merge할 계획입니다. 이 방법은 PR의 커밋 로그들을 한개로 추려서 master에 merge하는 방법입니다. 그리고 커밋 로그들이 한개로 추려지는데 이때 PR 제목이 커밋 메시지가 됩니다.(즉 PR == Commit)

**Squash and merge 방법을 선택한 이유**는 PR 1개는 1개의 기능을 담당하는 것이 일관성을 갖는다고 생각해서 선택하였습니다. 

## 1주차 회고
1주차는 협업과 관련된 내용을 중심으로 학습했습니다. 

특히 Git 사용법에 대해서 학습하였는데 협업이 원활하게 이루어 지기 위해서(ex: git 충돌(?) 등..) **일관된 커밋 로그, 브랜치 전략** 등의 중요성을 깨달을 수 있었습니다. 다음 이야기는 **상품 API 구현**에 대해서 다뤄보겠습니다.


## Reference
* [Git: 원격 저장소 브랜치 삭제](https://www.lesstif.com/gitbook/git-delete-remote-branch-20776547.html)
* [Git: 브랜치 이름 바꾸기](https://velog.io/@zansol/git%EB%A6%B0%EC%9D%B4-%ED%83%88%EC%B6%9C%EA%B8%B0-branch-%EC%9D%B4%EB%A6%84-%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0-g1jtzk99se)
* [Git: Merge 전략 3가지](https://evan-moon.github.io/2019/08/30/commit-history-merge-strategy/)
* [Git: After PR and Merge](https://brunch.co.kr/@anonymdevoo/9)
* [Git: Merge 이해하기](https://im-developer.tistory.com/182)
* [Git: gitflow-workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
* [Git: 배민의 branch 전략](https://woowabros.github.io/experience/2017/10/30/baemin-mobile-git-branch-strategy.html)