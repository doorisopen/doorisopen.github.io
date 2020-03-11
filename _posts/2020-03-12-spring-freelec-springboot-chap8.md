---
layout: post
title:  "[Spring Boot] Chap 8.EC2 서버에 프로젝트 배포하기"
date:   2020-03-12 00:30:59
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
>> 1. 프로젝트 설정 - MariaDB 드라이버 등록(build.gradle)
>> 2. RDB에 프로젝트에 사용되는 테이블 생성
>> 3. EC2 서버에 프로젝트 clone 받기
>> 4. EC2 서버에 배포 스크립트 작성
>> 5. EC2 설정 - RDS 접속 정보 설정

<br />
<br />


<hr />

> __chap8 ~ 10__ 은 __프로젝트 배포와 관련된 내용__ 을 포스팅 할 예정입니다.
> 
> 전체적인 __흐름을 단계별로 설명__ 하겠습니다. 각 단계는 __배포 방식을 개선하는 것으로__ 총 3단계로 진행됩니다.
> 
> * __Chap 8: Step 1.__ 스크립트를 실행하여 __수동__ 으로 프로젝트 Test & build 하기
>   + 프로젝트 설정 - MariaDB 드라이버 등록(build.gradle)
>   + RDB에 프로젝트에 사용되는 테이블 생성
>   + 외부 Security 파일 등록
>   + 배포 스크립트 작성
>   + EC2 설정 - RDS 접속 정보 설정
>   + EC2에서 소셜 로그인
> * __Chap 9: Step 2.__ 깃허브에 Push 하면 __자동__ Test & Build & Deploy
>   + Github와 Travis CI 연동
>   + 프로젝트 Travis CI(.travis.yml) 설정
>   + Travis CI와 AWS S3 연동
>     - AWS Key(IAM, identity and Access Management) 발급
>     - Travis CI에 IAM키 등록
>     - AWS S3 버킷 생성
>     - Travis CI의 빌드내용(Jar)을 S3에 올리기 위해 프로젝트(.travis.yml)에 설정 추가
>   + EC2와 CodeDeploy 연동
>     - CodeDeploy 연동을 위해 __EC2에서 사용할 IAM 역할 생성__
>     - EC2 서버에 CodeDeploy 에이전트 설치
>     - CodeDeploy -> EC2 접근을 위해 __CodeDeploy에서 사용할 IAM 역할 생성__
>     - CodeDeploy 생성
>     - CodeDeploy 관련 설정을 appspec.yml에 추가
>     - Travis CI 설정 파일(.travis.yml)에 CodeDeploy 내용을 추가
>   + 배포 자동화 구성(스크립트 파일(.sh) 작성)
>     - 배포를 위한 스크립트(Jar, appspec.yml)가 아닌 것을 제외하기 위해 .travis.yml 내용 수정
>     - Codedeploy 명령을 담당할 appspec.yml 파일 수정
> * __Chap 10: Step 3.__ Nginx __무중단__ 배포
>   + EC2 서버에 Nginx 설치 -> 서비스 시작
>   + EC2 보안 그룹 추가 : 80 포트
>   + 구글, 네이버 리디렉션 URI 추가
>   + 프로젝트와 Nginx 연동
>   + 무중단 배포 스크립트 작성
>     - 8001, 8002 어느 포트를 사용할지 판단하는 API 작성(/profile-real: TravisCI 배포 자동화를 위한 profile 입니다.)
>     - 무중단 배포를 위한 profile 2개(real1, real2) 추가(application-real1,2.properties 파일 생성)
>     - EC2 서버의 Nginx 설정 수정
>     - 배포 장소 변경/ 배포 스크립트 사용할 수 있도록 appspec.yml 내용 수정
>     - 프로젝트에 배포 스크립트 작성(5개, profile, start, stop, health, switch)

# Step 1에서 배포과정 요약
* git clone 혹은 git pull을 통해 새 버전의 프로젝트를 받음
* Gradle이나 Maven을 통해 프로젝트 테스트와 빌드
* EC2 서버에서 해당 프로젝트 실행 및 재실행

# 프로젝트 설정 - MariaDB 드라이버 등록(build.gradle)
먼저 RDS에 생성한 DB 유형인 MariaDB와 맞춰서 프로젝트에 __MariaDB 드라이버를 build.gradle에 등록__ 해 줍니다.

```
compile('org.mariadb.jdbc:mariadb-java-client')
```

그리고 서버에서 구동될 환경(스프링의 profile)을 하나 구성합니다. __src/main/resources/ 에 application-real.properties 파일을 추가__ 합니다. 앞에서 이야기한 대로 application-real.properties로 파일을 만들면 profile=real인 환경이 구성된다고 보면 됩니다. 실제 운영될 환경이기 때문에 보안/로그상 이슈가 될 만한 설정들을 모두 제거하며 __RDS환경 profile__ 설정이 추가됩니다.

* application-real.properties

```
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```

# RDB에 프로젝트에 사용되는 테이블 생성

EC2 혹은 IntelliJ에서 RDS 서버에 접속하여 이후 __배포될 프로젝트에서 사용되는 테이블들을 생성__ 해 줍니다. 

```
create table posts (
 id bigint not null auto_increment,
 created_date datetime,
 modified_date datetime,
 author varchar(255),
 content TEXT not null,
 title varchar(500) not null,
 primary key (id)
) engine=InnoDB;

create table user (
 id bigint not null auto_increment,
 created_date datetime,
 modified_date datetime, email varchar(255) not null,
 name varchar(255) not null,
 picture varchar(255),
 role varchar(255) not null,
 primary key (id)
 ) engine=InnoDB;
/* 
* 스프링 세션 테이블은 -> schema-mysql.sql 에서 확인 가능합니다.
* File 검색(win: Ctrl+Shift+N)으로 검색
*/
CREATE TABLE SPRING_SESSION (
	PRIMARY_ID CHAR(36) NOT NULL,
	SESSION_ID CHAR(36) NOT NULL,
	CREATION_TIME BIGINT NOT NULL,
	LAST_ACCESS_TIME BIGINT NOT NULL,
	MAX_INACTIVE_INTERVAL INT NOT NULL,
	EXPIRY_TIME BIGINT NOT NULL,
	PRINCIPAL_NAME VARCHAR(100),
	CONSTRAINT SPRING_SESSION_PK PRIMARY KEY (PRIMARY_ID)
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;

CREATE UNIQUE INDEX SPRING_SESSION_IX1 ON SPRING_SESSION (SESSION_ID);
CREATE INDEX SPRING_SESSION_IX2 ON SPRING_SESSION (EXPIRY_TIME);
CREATE INDEX SPRING_SESSION_IX3 ON SPRING_SESSION (PRINCIPAL_NAME);

CREATE TABLE SPRING_SESSION_ATTRIBUTES (
	SESSION_PRIMARY_ID CHAR(36) NOT NULL,
	ATTRIBUTE_NAME VARCHAR(200) NOT NULL,
	ATTRIBUTE_BYTES BLOB NOT NULL,
	CONSTRAINT SPRING_SESSION_ATTRIBUTES_PK PRIMARY KEY (SESSION_PRIMARY_ID, ATTRIBUTE_NAME),
	CONSTRAINT SPRING_SESSION_ATTRIBUTES_FK FOREIGN KEY (SESSION_PRIMARY_ID) REFERENCES SPRING_SESSION(PRIMARY_ID) ON DELETE CASCADE
) ENGINE=InnoDB ROW_FORMAT=DYNAMIC;
```

# 외부 Security 파일 등록
외부에 Security 파일을 등록을 해주어야 합니다. 이유는 기존에 프로젝트에는 application-oauth.properties가 있어 __clientId와 clientSecret으로 ClientRegistrationRepository를 생성__ 하였지만 해당 파일이 .gitignore로 등록하여 깃 허브에는 올라가지 않기 때문입니다. 애플리케이션을 실행하기 위해 공개된 저장소에 clientId와 clientSecret을 올릴 수는 없으니 __서버에서 직접 이 설정들을 가지고 있게__ 하겠습니다.

```
vim /home/ec2-user/app/application-oauth.properties
```

```
spring.security.oauth2.client.registration.google.client-id=클라이언트ID
spring.security.oauth2.client.registration.google.client-secret=클라이언트 비밀
spring.security.oauth2.client.registration.google.scope=profile,email

# registration
spring.security.oauth2.client.registration.naver.client-id=클라이언트ID
spring.security.oauth2.client.registration.naver.client-secret=클라이언트 비밀
spring.security.oauth2.client.registration.naver.redirect-uri={baseUrl}/{action}/oauth2/code/{registrationId}
spring.security.oauth2.client.registration.naver.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.naver.scope=name,email,profile_image
spring.security.oauth2.client.registration.naver.client-name=Naver

# provider -> https://developers.naver.com/apps/#/myapps/eR05U0GAJw6PSXCVMKSs/overview
spring.security.oauth2.client.provider.naver.authorization-uri=https://nid.naver.com/oauth2.0/authorize
spring.security.oauth2.client.provider.naver.token-uri=https://nid.naver.com/oauth2.0/token
spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
spring.security.oauth2.client.provider.naver.user-name-attribute=response
```

# EC2 서버에 프로젝트 clone 받기
프로젝트 설정과 DB 테이블을 생성 하였으니 __깃 허브 저장소에 있는 프로젝트를 EC2 서버에 clone__ 합니다. 그 전에 EC2 서버에 git을 설치가 필요합니다. 아래의 명령어를 순차적으로 수행해주세요. 

```
sudo yum install git // git 설치

git --version // 설치 확인

mkdir ~/app && mkdir ~/app/step1 // 디렉토리 생성

cd ~/app/step1

git clone 깃 허브 URL

cd 프로젝트명 // 프로젝트로 이동
```

프로젝트 clone이 완료 되었으면 코드들이 잘 수행되는지 테스트 검증하겠습니다. chap 5. 스프링 시큐리티 내용을 잘 따라왔다면 테스트는 정상 수행이 될 것입니다.

```
./gradlew test // 테스트 수행

// 만약 권한이 없다는 메시지가 뜰때 아래 명령어를 수행해주세요
chmod +x ./gradlew // gradlew 파일에 실행 권한 부여
```

# EC2 서버에 배포 스크립트 작성
EC2 서버에 프로젝트를 가져왔습니다. 이제 본격적으로 배포하는 과정을 설명하겠습니다.

### 배포 란?
작성한 코드를 실제 서버에 반영하는 것을 말합니다. 

__step 1 에서 수행할 배포__ 는 아래의 과정을 포괄하는 의미를 가집니다.

* git clone 혹은 git pull을 통해 새 버전의 프로젝트를 받음
* Gradle이나 Maven을 통해 프로젝트 테스트와 빌드
* EC2 서버에서 해당 프로젝트 실행 및 재실행

위의 과정을 배포할 때마다 개발자가 하나하나 명령어를 실행하는 것은 불편함이 많습니다. 그래서 이를 쉘 스크립트로 작성해 스크립트만 실행하면 앞의 과정이 차례로 진행되도록 하겠습니다. __쉘 스크립트__ 는 리눅스에서 기본적으로 사용할 수 있는 스크립트 파일의 한종류입니다. 노드JS가 .js라는 파일을 통해 서버에서 작동하는 것과 같은 것이라고 생각하면됩니다.

__~/app/step1 에 deploy 라는 이름의 쉘 스크립트 파일을 하나 생성__ 합니다.

```
vim ~/app/step1/deploy.sh
```

* deploy

```
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1 # (1)
PROJECT_NAME=freelec-springboot2-webservice

cd $REPOSITORY/$PROJECT_NAME/ # (2)

echo "> Git Pull" # (3)

git pull

echo "> 프로젝트 Buile 시작"

./gradlew build # (4)

echo "> step 1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/ # (5)

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar) # (6)

echo "> 현재 구동중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then # (7)
    echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1) # (8)

echo "> JAR Name: $JAR_NAME"

# 시큐리티 설정 전 nohup java -jar $REPOSITORY/|$JAR_NAME 2>1 & (9)

nohup java -jar \
        -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
        $REPOSITORY/$JAR_NAME 2>&1 & # 시큐리티 설정 후 (10)
```

* (1) `REPOSITORY=/home/ec2-user/app/step1`
  + 프로젝트디렉토리 주소는 스크립트 내에서 자주 사용하는 값이기 때문에 이를 변수로 저장합니다.
  + 마찬가지로 PROJECT_NAME=freelec-springboot2-webservice도 동일하게 변수로 저장합니다.
  + 쉘에서는 타입 없이 선언하여 저장합니다.
  + 쉘에서는 $ 변수명으로 변수를 사용할 수 있습니다.
* (2) `cd $REPOSITORY/$PROJECT_NAME/`
  + 제일 처음 git clone 받았던 디렉토리로 이동합니다.
  + 바로 위의 쉘 변수 설명을 따라 /home/ec2-user/app/step1/freelec-springboot2-webservice 주소로 이동합니다.
* (3) `git pull`
  + 디렉토리 이동 후, master 브랜치의 최신 내용을 받습니다.
* (4) `./gradlew build`
  + 프로젝트 내부의 gradlew로 build를 수행합니다.
* (5) `cp ./build/libs/*.jar $REPOSITORY/`
  + build의 결과물인 jar 파일을 복사해 jar 파일을 모아둔 위치로 복사합니다.
* (6) `CURRENT_PID=$(pgrep -f springboot-webservice*.jar)`
  + 기존에 수행 중이던 스프링 부트 애플리케이션을 종료합니다.
  + pgrep은 process id만 추출하는 명령어입니다.
  + -f 옵션은 프로세스 이름으로 찾습니다.
* (7) `if~else~fi`
  + 현재 구동 중인 프로세스가 있는지 없는지 판단해서 기능을 수행합니다.
  + process id 값을 보고 프로세스가 있으면 해당 프로세스를 종료합니다.
* (8) `JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)`
  + 새로 실행할 jar 파일명을 찾습니다.
  + 여러 jar 파일이 생기기 때문에 tail -n로 가장 나중의 jar파일(최신 파일)을 변수에 저장합니다.
* (9) `nohup java -jar $REPOSITORY/|$JAR_NAME 2>1 &`
  + 찾은 jar 파일명으로 해당 jar파일을 nohup으로 실행합니다.
  + 스프링 부트의 장점으로 특별히 외장 톰캣을 설치할 필요가 없습니다.
  + 내장 톰캣을 사용해서 jar 파일만 있으면 바로 웹 애플리케이션 서버를 실행할 수 있습니다.
  + 일반적으로 자바를 실행할 때는 java -jar라는 명령어를 사용하지만, 이렇게 하면 사용자가 터미널 접속을 끊을 때 애플리케이션도 같이 종료됩니다.
  + 애플리케이션 실행자가 터미널을 종료해도 애플리케이션은 계속 구동될 수 있도록 nohup 명령어를 사용합니다.
* (10) `-Dspring.config.location`
  + 스프링 설정 파일 위치를 지정합니다.
  + 기본 옵션들을 담고 있는 application.properties과 OAuth 설정들을 담고 있는 application-oauth.properties의 위치를 지정합니다.
  + __classpath가 붙으면__ jar 안에 있는 resources 디렉토리를 기준으로 경로가 생성됩니다.
  + application-oauth.properties 은 __절대경로를 사용합니다. 외부에 파일이 있기 때문__ 입니다.

# EC2 설정 - RDS 접속 정보 설정
OAuth와 마찬가지로 RDS 접속 정보도 보호해야 할 정보이니 EC2 서버에 직접 설정 파일을 둡니다.

```
vim ~/app/application-real-db.properties
```

* application-real-db.properties

```
spring.jpa.hibernate.ddl-auto=none # (1)
spring.datasource.url=jdbc:mariadb://rds주소:포트명(기본 3306)/database이름
spring.datasource.username=db계정(RDS 인스턴스 생성시 입력한 마스터 계정을 말합니다.)
spring.datasource.password=db계정 비밀번호(RDS 인스턴스 생성시 입력한 마스터 계정 비밀번호을 말합니다.)
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
```

* (1) `spring.jpa.hibernate.ddl-auto=none`
  + JPA로 테이블이 자동 생성되는 옵션을 None(생성하지 않음)으로 지정합니다.
  + RDS에는 실제 운영으로 사용될 테이블이니 절대 스프링 부트에서 새로 만들지 않도록 해야 합니다.
  + 이 옵션을 하지 않으면 자칫 테이블이 모두 새로 생성될 수 있습니다.
  + 주의해야 하는 옵션 입니다.


* deploy 스크립트 파일 수정

```
nohup java -jar \
        -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
        -Dspring.profiles.active=real \
        $REPOSITORY/$JAR_NAME 2>&1 # (1)
```

* (1) `-Dspring.profiles.active=real`
  + application-real.properties를 활성화시킵니다.
  + application-real.properties의 spring.profiles.include=oauth,real-db 옵션 때문에 real-db 역시 함께 활성화 대상에 포함됩니다.


# EC2에서 소셜 로그인
기존 __로컬 환경이 아닌 EC2 환경에서 소셜 로그인을 수행__ 하기 위해서 몇 가지 작업이 필요합니다.

* AWS 보안 그룹 변경 -> __보안 그룹에 8080 포트를 등록합니다.__
* EC2 퍼블릭 DNS를 구글과 네이버 서비스에 추가해줍니다.
  + 구글 -> __승인된 리디렉션 URI__
    - EC2 퍼블릭 DNS:8080/login/oauth2/code/google
  + 네이버 -> __서비스 URL, Callback URL 2개에 등록__
    - EC2 퍼블릭 DNS:8080/login/oauth2/code/naver


> 소셜 로그인 설정을 끝으로 step 1은 끝이 났습니다. step 2는 깃 허브에 푸시를 하면 자동으로 Test & Build & Deploy가 진행되도록 개선하는 작업을 해보겠습니다.

# References
* 스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)