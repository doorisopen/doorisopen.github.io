---
layout: post
title: "우아한테크코스 3기 지원후기(프리코스 1주차)"
date: 2020-11-30 00:50:59
author: me
categories: Woowacourse
tags: woowacourse
cover: "/assets/images/2020/woowacourse/woowacourse-cover.png"
---

해당 내용은 2020년도 10월 우아한테크코스(이하, 우테코)를 어떤 생각을 하며 지원했는지에 대한 이야기를 다룹니다.


11월 25일 수요일 드디어!! 1주차 프리코스가 시작되었습니다!

1주차 미션은 1기, 2기와 동일하게 **"숫자 야구 게임"** 이 미션으로 주어졌습니다.

<a href="{{ site.2020_woowacourse_img }}/woowacourse-freecourse-week-1.jpg" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2020_woowacourse_img }}/woowacourse-freecourse-week-1.jpg" title="Check out the image">
</a>

이번 1주차 미션의 요구사항을 정리하면 다음과 같습니다.

* **자바 코드 컨벤션** 을 지키면서 프로그래밍한다.
  + 기본적으로 Google Java Style Guide을 원칙으로 한다.
  + 단, 들여쓰기는 '2 spaces'가 아닌 '4 spaces'로 한다.
* indent(인덴트, 들여쓰기) **depth를 3이 넘지 않도록 구현** 한다. 2까지만 허용한다.
  + 예를 들어 while문 안에 if문이 있으면 들여쓰기는 2이다.
  + 힌트: indent(인덴트, 들여쓰기) depth를 줄이는 좋은 방법은 함수(또는 메소드)를 분리하면 된다.
* 3항 연산자를 쓰지 않는다.
* **함수(또는 메소드)가 한 가지 일만 하도록 최대한 작게** 만들어라.
* System.exit 메소드를 사용하지 않는다.
* **비정상적 입력에 대해서는 IllegalArgumentException을 발생** 시킨다.
* 기능을 구현하기 전에 **README.md 파일에 구현할 기능 목록을 정리해 추가** 한다.
* git의 **commit 단위는** 앞 단계에서 README.md 파일에 정리한 **기능 목록 단위로 추가** 한다.
  + AngularJS Commit Message Conventions 참고해 commit log를 남긴다.

## 미션 수행 계획
이번 1주차 미션을 받고 다음과 같은 계획을 세웠습니다.

* 행동(action)단위로 기능 구현 목록을 작성한다.
* 기능 구현 코드 작성 전에 테스트 코드를 먼저 작성한다.
* 클래스의 역할과 책임에 대해 고민하고 구분한다.
* indent 제약을 2가 아닌 1로 제한하여 1이 되는 방법을 찾고 구현한다.


## 고민한 부분

미션을 수행하면서 가장 고민했던 부분은 **"클래스의 역할 범위를 어디까지 이고 어떻게 분리 해야하는가?"** 였습니다.

1주차 미션의 패키지 구조는 다음과 같습니다.

* baseball
  + `Application` : main
  + `BaseBallGame` : 숫자 야구 게임 수행 클래스 
* baseball.domain
  + `Ball` : 숫자 야구 공 도메인
  + `GameInfomation` : 숫자 야구 게임의 Constant 클래스
* utils
  + `BallGenerator` : 컴퓨터, 사용자 공을 생성하는 클래스
  + `InputValidation` : 입력값 검증하는 클래스
  + `RandomUtils` : 랜덤 값을 생성하는 클래스


* `Ball`

```java
public class Ball {
    private BallGenerator ballGenerator;

    public Ball() {
        this.ballGenerator = new BallGenerator();
    }

    public List<Integer> getComputerBall() {
        return ballGenerator.createComputerBall();
    }

    public List<Integer> getUserBall(Scanner sc) {
        return ballGenerator.createUserBall(sc);
    }
}
```


* `BallGenerator`

```java
public class BallGenerator {

    public List<Integer> createComputerBall() {
        List<Integer> balls = new ArrayList<>();
        int num;
        while(balls.size() < NUMBER_LENGTH) {
            num = RandomUtils.nextInt(NUMBER_RANGE_ONE, NUMBER_RANGE_NINE);
            if(!balls.contains(num)) {
                balls.add(num);
            }
        }
        return balls;
    }

    public List<Integer> createUserBall(Scanner sc) {
        System.out.print(INPUT_USER_BALL_MESSAGE);
        String num = sc.next();
        validateNumberLength(num);
        validateIsOnlyDigit(num);
        validateNumberRange(num);
        validateNumberDuplicate(num);
        return parseStringToIntegerList(num);
    }

    private List<Integer> parseStringToIntegerList(String num) {
        return Arrays.stream(num.split(""))
                .map(Integer::parseInt)
                .collect(Collectors.toList());
    }
}
```

처음에 생각한 것은 `Ball` 클래스는 공의 생성까지 담당한다고 생각했습니다. 그러나 고민해 보니 Ball이 생성에 대한 책임을 가져야할까? 라는 의문이 들었고 공 생성하는 로직을 `BallGenerator` 클래스로 분리하였습니다...

솔직히 이 부분은 잘 분리했는지? 더 구분할 수 없는지? 에 대한 의문을 아직도 가지고 있습니다.

최종적으로 각 클래스의 역할에 대해 고민을 정리하면 다음과 같습니다.

* `Ball` 는 공(숫자)을 생성의 역할을 가지지 않고 공 그 자체만을 가진다.
* `BallGenerator` 공(컴퓨터, 사용자 숫자) 생성을 담당한다.
* `BaseBallGame` 숫자 야구 게임의 모든 로직을 담당한다.
  + 게임 시작
  + 게임 결과 계산(스트라이크, 볼 개수)
  + 게임 결과 출력
  + 게임 종료 or 재시작 여부

그리고, 그외 입력값 검증 클래스, 상수 클래스를 따로 분리 하였습니다.


## 미션을 통해 느낀점 or 배운점
* 기능 단위로 커밋을 하는것이 다소 어색함을 주었습니다. 그러나 기능 하나에 집중할 수 있었습니다.
* 테스트 코드를 먼저 작성함으로써 예외 상황을 대비하고 완성도 높은 기능을 구현할 수 있었습니다.
* 클래스의 역할과 책임을 고민하면서 클래스를 분리하였고 좀 더 객체지향적인 코드를 작성할 수 있었습니다.
* java stream을 활용하여 indent를 1로 만드는 방법에 대해서 학습하였습니다.


1주차 미션으로 기능 구현 뿐만 아니라 예외상황 검증, README 작성, Code Convention, Git 사용법 등을 익히고 경험할 수 있었습니다. 2주차 미션에서는 화면과 핵심 로직을 분리해서 좀더 객체지향 스러운 코드를 작성해보려고 합니다.

* [미션 Repository Link](https://github.com/doorisopen/java-baseball-precourse/tree/doorisopen)
* [미션 PR Link](https://github.com/woowacourse/java-baseball-precourse/pull/251)