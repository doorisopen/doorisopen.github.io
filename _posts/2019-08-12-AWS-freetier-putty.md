---
layout: post
title:  "[AWS Free Tier] 1.EC2 인스턴스 생성"
date:   2019-08-11 14:02:59
author: me
categories: Aws
tags: AWS EC2 web Server free tier
cover:  "/assets/images/Aws/awscover.png"
---

<br />
<br />

>> 이번에는 Putty를 통해 우리가 만든 EC2 인스턴스에 접속을 해볼거다. <br>
>> 인스턴스 생성이 안되었다면 <a href="https://doorisopen.github.io/aws/2019/08/11/AWS-freetier-ec2.html">이곳</a> 를 참고하면된다.


<hr />


### 1. Putty 다운로드
* Putty는 <a href="https://putty.ko.softonic.com/">이곳</a> 홈페이지를 통해서 다운로드를 받아주세요


<hr />


### 2. Puttygen 실행
> Puttygen은 <a href="https://www.puttygen.com/">이곳</a> 홈페이지를 통해서 다운로드를 받아주세요

* Puttygen을 통해서 ppk파일을 우선 만들어야합니다.
* 상단에 위치한 Conversions 를 누르시면 Import key 가 하단메뉴로 나타난다 그것을 클릭해준다. 
<a href="{{ site.aws_img }}/freetier_putty_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_putty_1.JPG" title="Check out the image">
</a>

* 그리고, 이전에 __EC2 인스턴스__ 를 생성할때 받았던 __pem 비밀번호 파일__ 을 열어주시면 된다. 
* 아래 사진과 같이 __Key passphrase(비밀번호)__ 를 입력해준다. 그 후 __Save Private Key__ 를 눌러서 저장한다.

> __Key passphrase(비밀번호)__ 의 경우 Putty를 통해 서버에 접속시 필요하기 때문에 잘 기억해둔다.

<a href="{{ site.aws_img }}/freetier_putty_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_putty_2.JPG" title="Check out the image">
</a>

> __private key의 저장 위치를 꼭 기억해야한다. 혹여나 실수로 github와 같은 곳에 유출되지 않도록 조심해야한다.__

<hr />


### 3. Putty 실행 및 접속설정

* PPK파일을 만들었으니 이제 실제로 __EC2 인스턴스__ 에 접속 할 수 있다.
* Putty를 실행 해주시면 아래와 같은 창이 뜬다.
* __Host Name__ 의 경우 __Amazon Linux AMI__ 로 선택 했기 때문에 __ec2-user@'EC2의 public IP'__ 를 적어주면 된다.
  * __예)__
    인스턴스에 public IP가 12.123.456.789 라면<br>
    __ec2-user@12.123.456.789__ 로 접속하시면 된다<br>
    여기서 바로 Open을 누르시면 안되고 방금 만들었던 PPK 파일을 설정 해야한다.
<a href="{{ site.aws_img }}/freetier_putty_3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_putty_3.JPG" title="Check out the image">
</a>

#### __참고(Host Name 설정)__
* __Amazon Linux 2__ 또는 __Amazon Linux AMI__ 의 경우 사용자 이름은 __ec2-user__ 입니다.
* __Centos AMI__ 의 경우 사용자 이름은 __centos__ 입니다.
* __Debian AMI__ 의 경우 사용자 이름은 __admin 또는 root__ 입니다.
* __Fedora AMI__ 의 경우 사용자 이름은 __ec2-user 또는 fedora__ 입니다.
* __RHEL AMI__ 의 경우 사용자 이름은 __ec2-user 또는 root__ 입니다.
* __SUSE AMI__ 의 경우 사용자 이름은 __ec2-user 또는 root__ 입니다.
* __Ubuntu AMI__ 의 경우 사용자 이름은 __ubuntu__ 입니다.


#### __참고(public ip 설정)__
* EC2 인스턴스의 public ip의 경우 아래의 사진을 참고 하면된다. 
<a href="{{ site.aws_img }}/freetier_putty_3_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_putty_3_1.JPG" title="Check out the image">
</a>


* 마지막으로 Putty로 EC2 서버에 접속 하기 위해서 우선 방금전 생성한 .ppk 파일을 등록해야한다.
* 왼쪽 __Category__ 에서 __SSH -> Auth__ 를 선택한다.
* Browse를 눌러 .ppk 파일을 등록한다. 그리고 __open 버튼__ 을 눌러 접속한다.
<a href="{{ site.aws_img }}/freetier_putty_4.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_putty_4.JPG" title="Check out the image">
</a>

* Open버튼을 누르면 아래와 같은 화면이 뜬다. 
* 아까전에 .ppk파일을 만들때 입력한 __Key passphrase__ 를 입력해준다.
  + 비밀번호를 작성해도 아무것도 보이지 않지만 제대로 작성되고 있으므로 당황하지 않는다.
<a href="{{ site.aws_img }}/freetier_putty_5.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_putty_5.JPG" title="Check out the image">
</a>

* 비밀번호를 작성하고 엔터를 누르면 아래와 같은 화면이 보이면 __ec2 서버__ 에 접속을 성공한것이다.
<a href="{{ site.aws_img }}/freetier_putty_6.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_putty_6.JPG" title="Check out the image">
</a>