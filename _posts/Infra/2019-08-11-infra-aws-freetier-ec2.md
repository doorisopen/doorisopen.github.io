---
layout: post
title:  "[AWS Free Tier] 1 - EC2 인스턴스 생성"
date:   2019-08-11 14:02:59
author: me
categories: Infra
tags: AWS EC2 web Server free tier
cover:  "/assets/images/Aws/awscover.png"
---

<br />
<br />

### AWS란?
* 아마존에서 제공하는 클라우드 서버를 말합니다. 아마존 웹 서비스는 쉽고 빠른 확정성과 비용절감이 가능한 플랫폼 클라우드로써 인프라 관리에 도움을 줍니다


<hr />


### Cloud Computing란?
* 기존 형태의 컴퓨팅리소스를 네트워크 기반 서비스로 제공하는 것을 말합니다.


<hr />


### 1. EC2 인스턴스 생성

* AWS에 로그인하고 왼쪽 상단에 서비스 중 EC2를 선택한다.
<a href="{{ site.aws_img }}/freetier_ec2_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_ec2_1.JPG" title="Check out the image">
</a>

* 인스턴스를 생성하고자 하기 떄문에 아래에 보이는 파란색 인스턴스 시작을 누른다.
<a href="{{ site.aws_img }}/freetier_ec2_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_ec2_2.JPG" title="Check out the image">
</a>


#### 1 단계 : AMI(Amazon Machine Images) 선택
* Amazon Machine Images 는 EC2 인스턴스를 생성하기 위한 기본 파일 입니다.
* 아마존에서 제공하는 AMI 중에서 인스턴스에 설치 될 운영체제를 선택해주시면 됩니다.
<a href="{{ site.aws_img }}/freetier_ec2_3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_ec2_3.JPG" title="Check out the image">
</a>

#### 2 단계 : 인스턴스 유형 선택
* 인스턴스의 유형을 선택하는 창이다. AWS 프리티어를 사용한다면 선택권이 없다.
* 아래에 __검토 및 시작__ 을 누르면 자동으로 검토를 해주고 마지막 설정 페이지로 이동한다.
<a href="{{ site.aws_img }}/freetier_ec2_4.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_ec2_4.JPG" title="Check out the image">
</a>

* 인스턴스를 검토하고 아래의 __시작하기__ 버튼을 누른다.
<a href="{{ site.aws_img }}/freetier_ec2_5.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_ec2_5.JPG" title="Check out the image">
</a>

* 시작하기 버튼을 누르면 아래와 같은 키 생성 창이 뜬다. 이때 __새 키 페어 생성__ 을 선택하고 아래 __키 페어 이름__ 을 입력해주고 키 페어 다운로드 버튼을 누르면 __*.pem 파일이__ 다운로드 된다.
  + 키 페어 이름은 본인이 원하는 이름으로 입력해주면된다. ex) hello ect..
>> *.pem 파일은 안전한 위치에 저장해야한다. 관리를 잘못한다면 과금 폭탄을 맞을 수 도 있으니 조심해야한다.
<a href="{{ site.aws_img }}/freetier_ec2_6.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_ec2_6.JPG" title="Check out the image">
</a>

* 인스턴스 시작 버튼을 누르면 다음과 같은 결과 화면을 볼 수 있다.
<a href="{{ site.aws_img }}/freetier_ec2_7.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_ec2_7.JPG" title="Check out the image">
</a>

이렇게 되면 AWS Free Tier EC2 인스턴스를 생성 완료한거다.

