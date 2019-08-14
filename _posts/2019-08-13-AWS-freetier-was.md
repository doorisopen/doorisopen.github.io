---
layout: post
title:  "[AWS Free Tier] 3 - Java, Tomcat 설치하기"
date:   2019-08-13 11:50:59
author: me
categories: Aws
tags: AWS EC2 web Server free tier Java Tomcat
cover:  "/assets/images/Aws/awscover.png"
---

<br />
<br />

* 해당 포스트는 아래의 과정까지 진행되어야 한다.
>> <a href="https://doorisopen.github.io/aws/2019/08/11/AWS-freetier-ec2.html">EC2 인스턴스 생성 하기</a>
<br>
>> <a href="https://doorisopen.github.io/aws/2019/08/12/AWS-freetier-putty.html">Putty로 접속 하기</a>


<br />
<br />


<hr />

## 개발 환경 설정하기
### Java & Apache Tomcat 설치하기

* Putty를 이용하여 EC2 서버에 접속하면 아래와 같은 화면을 볼 수 있다
<a href="{{ site.aws_img }}/freetier_was_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_was_1.JPG" title="Check out the image">
</a>

#### 1. Java 설치
1. __java -version 명령어__ 입력
2. 해당 명령어를 통해 현재 자바가 있는지 확인 그리고 버전을 확인해준다
<a href="{{ site.aws_img }}/freetier_was_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_was_2.JPG" title="Check out the image">
</a>

3. 구 버전의 자바가 설치 되어있으므로 삭제 후 최신 버전으로 설치한다
4. __rpm -qa | grep java 명령어__ 입력하여 설치되어 있는 자바 패키지를 나타낸다
<a href="{{ site.aws_img }}/freetier_was_3.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_was_3.JPG" title="Check out the image">
</a>

5. __sudo yum remove java-1.7.0-openjdk-1.7.0.211-2.6.17.1.79.amznl.x86_64 명령어__ 입력하여 삭제한다
<a href="{{ site.aws_img }}/freetier_was_4.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_was_4.JPG" title="Check out the image">
</a>

6. __sudo yum install java-1.8.0-openjdk.x86_64 명령어__ 입력하여 __java-1.8.0__ 설치
7. __java -version 명령어__ 입력하여 설치가 제대로 되었는지 확인한다



#### 2. Tomcat 설치

* __yum list | grep tomcat 명령어__ 를 통해 yum 으로 설치할 수 있는 목록을 확인할 수 있다.<br/>
 그러나, 이는 Tomcat8 버전 까지 지원 해주기 때문에 최신 버전을 원하는 사람은 별로 도움이 되지 않는다
 
* 최신 버전의 Tomcat을 원하지 않는다면 아래의 단계를 수행하면된다
1. __sudo yum install tomcat8 명령어__ 입력
2. __sudo yum install tomcat8-admin-webapps 명령어__ 입력
3. __sudo yum install tomcat8-webapps 명령어__ 입력
4. __sudo yum install tomcat8-docs-webapp 명령어__ 입력
5. __sudo service tomcat8 start 명령어__ 입력하여 톰켓을 실행 

> 만약 위의 방법으로 Tomcat을 설치 했다면 __'/usr/share/tomcat8'__ 경로에 있으니 참고할 것
* __tomcat 서비스 시작 명령어__ : sudo service tomcat8 start



#### 3. Tomcat 9 설치
1. __cd opt__ 로 이동하여 __mkdir server 명령어__ 입력 하여 server 폴더 생성
2. __cd server__ server 폴더로 이동
* __sudo wget http://apache.tt.co.kr/tomcat/tomcat-9/v9.0.22/bin/apache-tomcat-9.0.22.tar.gz__ 명령어 입력 하여 Tomcat 9.0.22 버전 설치한다.
<a href="{{ site.aws_img }}/freetier_was_5.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_was_5.JPG" title="Check out the image">
</a>

* __sudo tar xzf apache-tomcat-9.0.22.tar.gz 명령어__ 입력하여 압축풀기
<a href="{{ site.aws_img }}/freetier_was_6.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_was_6.JPG" title="Check out the image">
</a>

* __tomcat9 서비스 실행 명령어__
  + __cd apache-tomcat-9.0.22__ 명령어 입력하여 tomcat 폴더로 이동
  + __./bin/startup.sh 명령어__ 입력 후 서비스 실행
<a href="{{ site.aws_img }}/freetier_was_7.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_was_7.JPG" title="Check out the image">
</a>

  + 서비스 종료할때는 __./bin/shutdown.sh 명령어__ 를 입력하여 서비스 종료한다. 


* 주소창에 __public ip:8080__ 입력 후 아래와 같은 화면이 뜨면 성공
<a href="{{ site.aws_img }}/freetier_was_8.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.aws_img }}/freetier_was_8.JPG" title="Check out the image">
</a>

<br />

>> __tomcat 시작을 위한 init 스크립트를 생성하는 방법은 추후에 추가 예정...__

<br />


<hr />


### References
> * <a href="https://ellieya.tistory.com/157">[LINUX] 리눅스에 톰켓 서비스 등록하기<a>
> * <a href="http://progtrend.blogspot.com/2018/07/aws-amazon-linux-2-tomcat-9.html">(AWS) Amazon Linux 2에 Tomcat 9 설치하기<a>
> * <a href="https://goddaehee.tistory.com/74?category=250744">CentOS7 톰캣 설치<a>
> * <a href="https://hostpresto.com/community/tutorials/how-to-install-apache-tomcat-9-on-centos-7/">How to Install Apache Tomcat 9 on CentOS 7<a>
> * <a href="http://dev.crois.net/2019/02/10/centos7-tomcat-9-install/">[CentOS7] Tomcat 9 install<a>
> * <a href="https://sethlee.tistory.com/2">centos 7.0에서 was(tomcat9) 설치하기!<a>