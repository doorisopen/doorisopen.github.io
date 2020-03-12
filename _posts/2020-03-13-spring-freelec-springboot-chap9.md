---
layout: post
title:  "[Spring Boot] Chap 9.Travis CI 배포 자동화"
date:   2020-03-13 00:30:59
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
>> 1. CI & CD 란
>> 2. Github와 Travis CI 연동
>> 3. 프로젝트에 Travis CI(.travis.yml) 설정
>> 4. Travis CI와 AWS S3 연동
>> 5. EC2와 CodeDeploy 연동
>> 6. 배포 자동화 구성(스크립트 파일(.sh) 작성)

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
>   + Travis CI와 AWS S3, CodeDeploy 연동하기
>     - EC2와 CodeDeploy 연동
>     - CodeDeploy 연동을 위해 __EC2에서 사용할 IAM 역할 생성__
>     - EC2 서버에 CodeDeploy 에이전트 설치
>     - CodeDeploy -> EC2 접근을 위해 __CodeDeploy에서 사용할 IAM 역할 생성__
>     - CodeDeploy 생성
>     - CodeDeploy 관련 설정을 appspec.yml에 추가
>     - Travis CI 설정 파일(.travis.yml)에 CodeDeploy 내용을 추가
>   + 배포 자동화 구성(스크립트 파일(.sh) 작성)
>     - 배포를 위한 스크립트(Jar, appspec.yml)가 아닌 것을 제외하기 위해 .travis.yml 내용 수정
>     - Codedeploy 명령을 담당할 appspec.yml 파일 수정
>   + CodeDeploy 로그 확인
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


* __Chap 9: Step 2.__ 깃허브에 Push 하면 __자동__ Test & Build & Deploy

# CI & CD 란
chap 8 에서는 배포 스크립트를 개발자가 직접 실행함으로써 발생하는 불편이 있었습니다. 그래서 CI, CD 환경을 구축하여 이 과정을 개선해보겠습니다.

__CI & CD 란__ 코드 버전 관리를 하는 VCS 시스템(git, svn 등)에 __PUSH가 되면 자동으로 테스트와 빌드가 수행되어 안정적인 배포 파일을 만드는 과정을 CI(Continuous Integration-지속적 통합)__ 라고 하며, 이 __빌드 경과를 자동으로 운영 서버에 무중단 배포까지 진행되는 과정을 CD(Continuous Deployment-지속적인 배포)__ 라고 합니다.

# Github와 Travis CI 연동
Travis CI는 깃허브에서 제공하는 무료 CI 서비스입니다. 젠킨스와 같은 CI 도구도 있지만, 젠킨스는 설치형이기 때문에 이를 위한 EC2 인스턴스가 하나 더 필요합니다.

[Travis CI](https://travis-ci.org/)에 접속해서 깃허브 계정으로 로그인을 한 뒤, 왼쪽 위에 __계정명 > Setting__ 을 클릭합니다. 그리고 해당 프로젝트를 활성화 시킵니다.

# 프로젝트에 Travis CI(.travis.yml) 설정
Travis CI의 상세한 설정은 프로젝트에 존재하는 __.travis.yml 파일__ 로 할 수 있습니다. build.gradle과 같은 위치에 해당 파일을 생성합니다.

```
language: java
jdk:
  - openjdk8

branches: # (1)
  only:
    - master

before_install:
  - chmod +x gradlew

# Travis CI 서버의 Home
cache: # (2)
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

script: "./gradlew clean build" # (3)

# CI 실행 완료 시 메일로 알람
notifications: # (4)
  email:
    recipients:
      - 본인 이메일
```

* (1) `branches`
  + Travis CI를 어느 브랜치가 푸시될 때 수행할지 지정합니다.
  + 현재 옵션은 오직 __master 브랜치에 push__ 될 때만 수행합니다.
* (2) `cache`
  + 그레이들을 통해 의존성을 받게 되면 이를 해당 디렉토리에 캐시하여, __같은 의존성은 다음 배포 때 부터 다시 받지 않도록__ 설정합니다.
* (3) `script`
  + master 브랜치에 푸시되었을 때 수행하는 명령어입니다.
  + 여기서는 프로젝트 내부에 둔 gradlew을 통행 clean & build를 수행합니다.
* (4) `notification`
  + Travis CI 실행 완료 시 자동으로 알람이 가도록 설정합니다.

# Travis CI와 AWS S3 연동
__S3란__ AWS에서 제공하는 __일종의 파일 서버__ 입니다. 이미지 파일을 비롯한 정적 파일들을 관리하거나 지금 진행하는 것처럼 배포 파일들을 관리하는 등의 기능을 지원합니다. 보통 이미지 업로드를 구현한다면 이 S3를 이용하여 구현하는 경우가 많습니다. S3를 비롯한 AWS 서비스와 Travis CI를 연동하게 되면 전체 구조는 다음과 같습니다.

<a href="{{ site.2020_spring_img }}/freelec-springboot-chap9-1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2020_spring_img }}/freelec-springboot-chap9-1.JPG" title="Check out the image">
</a>

첫 번째 단계로 Travis CI와 S3를 연동하겠습니다. 실제 배포는 AWS CodeDeploy라는 서비스를 이용합니다. 하지만, S3 연동이 먼저 필요한 이유는 __Jar 파일을 전달하기 위해서__ 입니다. CodeDeploy는 저장 기능이 없습니다. 그래서 Travis CI가 빌드한 결과물을 받아서 CodeDeploy가 가져갈 수 있도록 보관할 수 있는 공간이 필요합니다. 보통 이럴 때 AWS S3를 이용합니다.

## AWS Key(IAM, identity and Access Management) 발급
일반적으로 __AWS 서비스에 외부 서비스가 접근할 수 없습니다.__ 그러므로 __접근 가능한 권한을 가진 Key__ 를 생성해서 사용해야 합니다. AWS에서는 이러한 인증과 관련된 기능을 제공하는 서비스로 __IAM(Identity and Access Management)__ 이 있습니다.

__IAM__ 은 AWS에서 제공하는 서비스의 접근 방식과 권한을 관리합니다. 이 IAM을 통해 Travis CI가 AWS S3와 CodeDeploy에 접근할 수 있도록 하겠습니다.

첫 번째로 IAM을 발급 받기 위해서 __서비스 > IAM > 왼쪽 사이드바 > 사용자 > 사용자 추가__ 로 이동합니다.

* __사용자 세부 정보 설정 > 사용자 이름__ 를 입력합니다.
* __AWS 엑세스 유형 선택 > 엑세스 유형 > 프로그래밍 방식 엑세스__ 를 체크합니다.
* __사용자 권한 설정 > 기존 정책 직접 연결__ 을 선택합니다.
* 정책 검색 화면에서 __2가지 정책을 추가__ 합니다.
  + s3full 검색&체크
  + CodeDeployFull 검색&체크
* __태크 추가 > 태그 Name 값 지정__ 는 본인이 인지 가능한 정도의 이름으로 만듭니다.

최종 생성이 완료되면 __엑세스 키와 비밀 엑세스 키__ 가 생성됩니다. 이것을 __Travis CI에 등록__ 해 줍니다.

## Travis CI에 IAM키 등록
Travis CI의 설정 화면으로 이동해서 Travis CI의 환경 변수에 
* __AWS_ACCESS_KEY__ (엑세스 키 ID)
* __AWS_ACCESS_SECRET_KEY__ (비밀 엑세스 키)

이 두가지를 변수로 해서 IAM 사용자에서 발급받은 키 값들을 등록합니다. 여기에 등록된 값들은 이제 .travis.yml에서 $AWS_ACCESS_KEY, $AWS_ACCESS_SECRET_KEY란 이름으로 사용할 수 있습니다.

이제 이 키를 사용해서 __Jar를 관리할 S3 버킷을 생성__ 합니다.

## AWS S3 버킷 생성
__AWS의 S3(Simple Storage Service)__ 는 일종의 __파일 서버__ 입니다. 순수하게 파일들을 저장하고 접근 권한을 관리, 검색 등을 지원하는 파일 서버의 역할을 합니다. S3는 보통 게시글을 쓸 때 나오는 첨부파일 등록을 구현할 때 많이 이용합니다. 파일 서버의 역할을 하기 때문인데, Travis CI에서 생성된 __Build 파일을 저장__ 하도록 구성하겠습니다. S3에 저장된 Build 파일은 이후 CodeDeploy에서 배포할 파일로 가져가도록 구성할 예정입니다.

* __서비스 > S3 > 버킷 만들기 > 버킷 이름 짓기__ 이 버킷에 __배포할 Zip 파일이 모여있는 장소__ 임을 의미하도록 짓는 것을 추천합니다.
* __버킷 버전 설정__ 은 별다른 설정 없이 넘어갑니다.
* __버킷의 보안과 권한 설정__ 부분에서는 __모든 퍼블릭 액세스를 차단__ 을 체크합니다.

## Travis CI의 빌드내용(Jar)을 S3에 올리기 위해 프로젝트(.travis.yml)에 설정 추가
S3가 생성되었으니 이제 S3로 배포 파일을 전달해 보겠습니다. __.travis.yml__ 파일에 코드를 추가합니다. 

* .travis.yml

```
script: "./gradlew clean build"

before_deploy: # (1)
  - zip -r freelec-springboot2-webservice * # (2)
  - mkdir -p deploy # (3)
  - mv freelec-springboot2-webservice.zip deploy/freelec-springboot2-webservice.zip # (4)

deploy: # (5)
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY # Travis repo setting에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo setting에 설정된 값
    bucket: freelec-springboot-doop-build # S3 버킷
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private으로
    local_dir: deploy # before_deploy에서 생성한 디렉토리 # (6)
    wait-until-deployed: true
```

* (1) `before_deploy`
  + deploy 명령어가 실행되기 전에 수행됩니다.
  + CodeDeploy는 Jar 파일은 인식하지 못하므로 Jar+기타 설정 파일들을 모아 압축(zip)합니다.
* (2) `zip -r freelec-springboot2-webservice`
  + 현재 위치의 모든 파일을 freelec-springboot2-webservice 이름으로 압축(zip)합니다.
  + 명령어의 마지막 위치는 본인의 프로젝트 이름이어야 합니다.
* (3) `mkdir -p deploy`
  + deploy라는 디렉토리를 Travis CI 가 실행 중인 위치에서 생성합니다.
* (4) `mv freelec-springboot2-webservice.zip deploy/freelec-springboot2-webservice.zip`
  + freelec-springboot2-webservice.zip 파일을 deploy/freelec-springboot2-webservice.zip으로 이동시킵니다.
* (5) `deploy`
  + S3로 파일 업로드 혹은 CodeDeploy로 배포 등 __외부 서비스와 연동될 행위들을 선언__ 합니다.
* (6) `local_dir: deploy`
  + 앞에서 생성한 deploy 디렉토리를 지정합니다.
  + __해당 위치의 파일__ 들만 S3로 전송합니다.

# Travis CI와 AWS S3, CodeDeploy 연동하기
## EC2와 CodeDeploy 연동
__EC2가 CodeDeploy를 연동 받을 수 있게__ EC2에서 사용할 IAM 역할 생성합니다. 

S3와 마찬가지로 IAM검색 합니다. 이번에는 __사용자가 아닌 역할__ 을 선택합니다. __IAM > 역할 만들기__

* __역할__
  + AWS 서비스에만 할당할 수 있는 권한
  + EC2, CodeDeploy, SQS 등
* __사용자__
  + AWS 서비스 외에 사용할 수 있는 권한
  + 로컬 PC, IDC 서버 등

지금 만들 권한은 __EC2에서 사용할 것__ 이므로 역할 만들기에서 서비스 선택를 __AWS 서비스 > EC2__ 으로 차례로 선택합니다. 정책에선 __EC2RoleForA 검색 > AmazonEC2RoleforAWS-CodeDeploy__ 선택 합니다. 태그는 본인이 원하는 이름으로 짓습니다. 마지막으로 역할의 이름을 등록하고 나머지 등록 정보를 최종적으로 확인합니다. 

이렇게 만든 역할을 EC2 서비스에 등록합니다. EC2 인스턴스 목록으로 이동한 뒤, 본인의 __인스턴스를 마우스 오른쪽 클릭 > 인스턴스 설정 > IAM 역할 연결/바꾸기__ 차례로 선택합니다. 그리고 방금 생성한 역할을 선택하고 인스턴스를 재부팅해줍니다.

## EC2 서버에 CodeDeploy 에이전트 설치
재부팅이 완료되었으면 CodeDeploy의 요청을 받을 수 있게 EC2 서버에 에이전트를 하나 설치합니다.

```
aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install .--region ap-northeast-2
...(내려받기 성공 후 메시지)
download: s3://aws-codedeploy-ap-northeast-2/latest/install to ./install

# 아래 명령어를 이어서 수행합니다.
chmod +x ./install // 실행 권한 추가

sudo ./ install auto # install 파일로 설치를 진행

sudo service codedeploy-agent status # 실행 확인

Ths AWS CodeDeploy agent is running as PID xxx # 이 메시지가 나오면 정상입니다.
```

## CodeDeploy -> EC2 접근을 위해 CodeDeploy에서 사용할 IAM 역할 생성
CodeDeploy에서 EC2에 접근하려면 마찬가지로 권한이 필요합니다. AWS의 서비스이니 IAM 역할을 생성합니다. __IAM > 역할 만들기 > AWS 서비스 > CodeDeploy__ 선택합니다. CodeDeploy의 권한은 하나뿐이라서 선택 없이 바로 다음으로 넘어갑니다. 태그 역시 본인이 원하는 이름으로 짓습니다.

## CodeDeploy 생성
CodeDeploy는 AWS의 배포 삼형제 중 하나입니다.

* Code Commit
  + 깃허브와 같은 코드 저장소의 역할을 합니다
  + 프라이빗 기능을 지원한다는 강점이 있지만, 현재 깃허브에서 무료로 프라이빗 지원을 하고 있어서 거의 사용되지 않습니다.
* Code Build
  + Travis CI 와 마찬가지로 빌드용 서비스입니다.
  + 멀티 모듈을 배포해야 하는 경우 사용해 볼만하지만, 규모가 있는 서비스에서는 대부분 __젠킨스/팀시티 등을 이용__ 하니 이것 역시 사용할 일이 거의 없습니다.
* Code Deploy
  + AWS의 배포 서비스입니다.
  + 앞에서 언급한 다른 서비스들은 대체재가 있고, 딱히 대체재보다 나은 점이 없지만, CodeDeploy는 대체재가 없습니다.
  + 오토 스케일링 그룹 배포, 블루 그린 배포, 롤링 배포, EC2 단독 배포 등 많은 기능을 지원합니다.

현재 프로젝트에서는 __Code Commit의 역할은 github가, Code Build의 역할은 Travis CI__ 가 하고있습니다.

__서비스 > CodeDeploy 서비스로 이동__ 해서 __애플리케이션 생성 버튼__ 을 클릭합니다.

* 애플리케이션 이름을 입력합니다.
* 컴퓨팅 플랫폼은 __EC2/온프레미스__ 를 선택합니다.

생성이 완료되면 배포 그룹을 생성하라는 메시지를 볼 수 있습니다. __배포 그룹 생성 버튼을 클릭__ 합니다.

* 배포 그룹 이름을 입력합니다.
* 서비스 역할 선택은 __방금 생성한 IAM 역할을 선택__ 합니다.
* 배포 유형은 __현재 위치를 선택__ 합니다.
  + 배포할 서비스가 2대 이상이라면 블루/그린을 선택하면됩니다.
* 환경 구성에는 __Amazon EC2 인스턴스__ 에 체크합니다.
* 배포 구성을 __CodeDeployDefault.AllAtOnce__ 를 선택하고 로드밸런싱은 __체크 해제__ 합니다.

여기 까지 CodeDeploy 설정은 끝이 났습니다. 이제 Travis CI와 CodeDeploy를 연동해 보겠습니다.

## CodeDeploy 관련 설정을 appspec.yml에 추가
먼저 S3에서 넘겨줄 zip 파일을 저장할 디렉토리를 EC2 서버에 하나 생성합니다.

```
mkdir ~/app/step2 && mkdir ~/app/step2/zip
```

Travis CI의 Build가 끝나면 S3에 zip 파일이 전송되고, 이 zip 파일은 /home/ec2-user/app/step2/zip로 복사되어 압축을 풀 예정입니다.

AWS CodeDeploy 설정을 위해 __프로젝트에 appspec.yml 파일을 추가__ 합니다.

* appspec.yml

```
version: 0.0 # (1)
os: linux
files:
  - source: / # (2)
    destination: /home/ec2-user/app/step2/zip/ # (3)
    overwrite: yes # (4)
```

* (1) `version`
  + CodeDeploy 버전을 이야기합니다.
  + 프로젝트 버전이 아니므로 0.0 외에 다른 버전을 사용하면 오류가 발생합니다.
* (2) `source`
  + CodeDeploy에서 전달해 준 파일 중 destination으로 이동시킬 대상을 지정합니다.
  + 루트 경로(/)를 지정하면 전체 파일을 이야기합니다.
* (3) `destination`
  + source에서 지정된 파일을 받을 위치입니다.
  + 이후 Jar를 실행하는 등은 destination에서 옮긴 파일들로 진행됩니다.
* (4) `overwrite`
  + 기존에 파일들이 있으면 덮어쓸지를 결정합니다.
  + 현재 yes라고 했으니 파일들을 덮어쓰게 됩니다.

## Travis CI 설정 파일(.travis.yml)에 CodeDeploy 내용을 추가

* .travis.yml

```
deploy:
  ...

  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY # Travis repo setting에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo setting에 설정된 값

    bucket: freelec-springboot-doop-build # S3 버킷
    key: freelec-springboot2-webservice.zip # 빌드 파일을 압축해서 전달
    bundle_type: zip # 압축 확장자
    application: freelec-springboot2-webservice # 웹 콘솔에서 등록한 CodeDeploy 애플리케이션

    deployment_group: freelec-springboot2-webservice-group # 웹솔에서 등록한 CodeDeploy 배포 그룹

    region: ap-northeast-2
    wait-until-deployed: true
```

모든 내용을 작성했다면 깃 허브에 커밋/ 푸시를 하고 CodeDeploy 에서 배포가 수행되는 것을 확인 할 수 있습니다. 또한 __cd /home/ec2-user/app/step2/zip__ 경로에서 프로젝트 파일들이 잘 도착했음을 볼 수 있습니다. 


# 배포 자동화 구성(스크립트 파일(.sh) 작성)
앞의 과정으로 __Travis CI와 S3, CodeDeploy가 연동이 완료되었습니다.__ 이제 이것을 기반으로 __실제로 Jar를 배포하여 실행__ 까지 해보겠습니다.

먼저 step 2 환경에서 실행될 deploy 스크립트 파일을 생성하겠습니다. 프로젝트 src 디렐토리와 같은 위치에 __scripts 디렉토리를 생성해서 여기에 스크립트를 생성__ 합니다.

* deploy 스크립트 파일(.sh)

```
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step2
PROJECT_NAME=freelec-springboot2-webservice

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 현재 구동 중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl freelec-springboot2-webservice | grep jar | awk '{print $1}') # (1)

echo "현재 구동 중인 애플리케이션pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
  echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $CURRENT_PID"
  kill -15 $CURRENT_PID
  sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR NAME: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME # (2)

echo "> $JAR_NAME 실행"

nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=real \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 & # (3)
```

* (1) `CURRENT_PID`
  + 현재 수행 중인 스프링 부트 애플리케이션의 프로세스 ID를 찾습니다.
  + 실행 중이면 종료하기 위해서입니다.
  + 스프링 부트 애플리케이션 이름(freelec-springboot2-webservice)으로 된 다른 프로그램들이 있을 수 있어 freelec-springboot2-webservice로 된 __jar(pgrep -fl freelec-springboot2-webservice | grep jar)__ 프로세스를 찾은 뒤 ID를 찾습니다. __(| awk '{print $1}')__
* (2) `chmod +x $JAR_NAME`
  + Jar 파일은 실행 권한이 없는 상태입니다.
  + nohup으로 실행할 수 있게 실행 권한을 부여합니다.
* (3) `$JAR_NAME > $REPOSITORY/nohup.out 2>&1 &`
  + nohup 실행 시 CodeDeploy는 __무한 대기__ 입니다.
  + 이 이슈를 해결하기 위해 nohup.out 파일을 표준 입출력용으로 별도로 사용합니다.
  + 이렇게 하지 않으면 nohup.out 파일이 생기지 않고, CodeDeploy 로그에 표준 입출력이 출력됩니다.
  + nohup이 끝나기 전까지 CodeDeploy도 끝나지 않으니 꼭 이렇게 해야만 합니다.

step 1과 2의 차이점은 __step 1에서 git pull을 통해 직접 빌드했던 부분을 제거__ 했습니다. 그리고 Jar를 실행하는 단계에서 몇가지 코드가 추가되었습니다.

## 배포를 위한 스크립트(Jar, appspec.yml)가 아닌 것을 제외하기 위해 .travis.yml 내용 수정

* .travis.yml

```
script: "./gradlew clean build"

before_deploy:
  - mkdir -p before-deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성 # (1)
  - cp scripts/*.sh before-deploy/ # (2)
  - cp appspec.yml before-deploy/
  - cp build/libs/*.jar before-deploy/
  - cd before-deploy && zip -r before-deploy * # before-deploy로 이동 후 전체 압축 # (3)
  - cd ../ && mkdir -p deploy # 상위 디렉토리로 이동 후 deploy 디렉토리 생성
  - mv before-deploy/before-deploy.zip deploy/freelec-springboot2-webservice.zip # deploy로 zip파일 이동
```

* (1) `Travis CI는 S3로 특정 파일만 업로드가 안됩니다.`
  + 디렉토리 단위로만 업로드할 수 있기 때문에 deploy 디렉토리는 항상 생성합니다.
* (2) `before-deploy에는 zip 파일에 포함시킬 파일들을 저장합니다.`
* (3) `zip -r 명령어를 통해 before-deploy 디렉토리 전체 파일을 압축합니다.`

## Codedeploy 명령을 담당할 appspec.yml 파일 수정

* appspec.yml

```
permissions: # (1)
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks: # (1)
  ApplicationStart:
    - location: start.sh # 엔진엑스와 연결되어 있지 않은 Port로 새 버전의 스프링 부트를 시작합니다.
      timeout: 60
      runas: ec2-user
```

* (1) `permissions`
  + CodeDeploy에서 EC2 서버로 넘겨준 파일들을 모두 ec2-user 권한을 갖도록 합니다.
* (2) `hooks`
  + CodeDeploy 배포 단계에서 실행할 명령어를 지정합니다.
  + ApplicationStart라는 단계에서 deploy 스크립트를 ec2-user 권한으로 실행하게 합니다.
  + timeout: 60으로 스크립트 실행 60초 이상 수행되면 실패가 됩니다(무한정 기다림을 방지하기 위함)

이렇게 모든 설정이 완료 되었습니다 커밋/푸시를 하고 간단하게 view 파일을 수정하고 웹 브라우저에서 확인해보면 변경됨을 확인할 수 있습니다.

# CodeDeploy 로그 확인
CodeDeploy에 관한 대부분의 내용은 

`/opt/codedeploy-agent/deployment-root`에 서 확인할 수 있습니다.

__step 2__ 에서의 문제점은 __배포하는 동안 스프링 부트 프로젝트는 종료 상태가 되어 서비스를 이용할 수 없다__ 는 것입니다. __step 3__ 에서는 __서비스 중단 없는 배포 방법__ 으로 개선해보겠습니다. 
