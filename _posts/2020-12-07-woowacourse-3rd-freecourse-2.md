---
layout: post
title: "우아한테크코스 3기 지원후기(프리코스 2주차)"
date: 2020-12-07 00:50:59
author: me
categories: Woowacourse
tags: woowacourse
cover: "/assets/images/2020/woowacourse/woowacourse-cover.png"
---


해당 내용은 2020년도 10월 우아한테크코스(이하, 우테코)를 어떤 생각을 하며 지원했는지에 대한 이야기를 다룹니다.


12월 2일 2주차 미션이 시작되었습니다!

2주차 미션은 1기, 2기와 동일하게 **"자동차 경주 게임"** 이 미션으로 주어졌습니다.

<a href="{{ site.2020_woowacourse_img }}/woowacourse-freecourse-week-2.jpg" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2020_woowacourse_img }}/woowacourse-freecourse-week-2.jpg" title="Check out the image">
</a>

2주차는 1주차 요구사항에 아래 요구사항이 추가 됐습니다.

* 예외 상황 시 에러 문구를 출력해야 한다. 단, 에러 문구는 [ERROR] 로 시작해야 한다.
* 함수(또는 메소드)의 길이가 15라인을 넘어가지 않도록 구현한다.
* 함수(또는 메소드)가 한 가지 일만 잘 하도록 구현한다.
* else 예약어를 쓰지 않는다.
  + 힌트: if 조건절에서 값을 return하는 방식으로 구현하면 else를 사용하지 않아도 된다.
  + else를 쓰지 말라고 하니 switch/case로 구현하는 경우가 있는데 switch/case도 허용하지 않는다.


## 미션 수행 계획 및 목표
* 패키지를 구성하여 입력, 처리, 출력을 구분한다.
* 함수(메소드)와 클래스를 분리 한다.

## 고민한 부분
* 1주차와 동일한 고민... 클래스의 역할 범위를 어디까지?? 그리고 어떻게 분리할 것인지??
* 중복된 이름의 참여자를 어떻게 처리할것인가??

### 클래스 분리하기
처음 구현 기능 목록을 작성할때 아래와 같이 입력, 처리, 출력을 나눠서 작성하였습니다.

* InputController: 입력과 입력 검증 로직이 들어간 클래스
* OutputController: 출력 로직이 들어간 클래스
* RacingCarGame: 자동차 경주 게임 로직이 들어간 클래스
* Car: 자동차 관련 로직이 들어간 클래스
* Cars: 경주에 참여한 자동차들과 관련된 로직이 들어간 클래스

그러나 **"입력과 입력을 검증하는 부분을 분리해야 옳은 설계인가?"**에 대해서 고민을 하였습니다.

고민한 결과 입력과 검증을 분리하는 것으로 결정하였고 다음과 같이 분리하였습니다.

<a href="{{ site.2020_woowacourse_img }}/woowacourse-freecourse-week-2-package.jpg" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2020_woowacourse_img }}/woowacourse-freecourse-week-2-package.jpg" title="Check out the image">
</a>

* InputController: 입력 로직이 들어간 클래스
* **InputValidation**: 입력 검증 로직이 들어간 클래스
* OutputController: 출력 로직이 들어간 클래스
* RacingCarGame: 자동차 경주 게임 로직이 들어간 클래스
* Car: 자동차 관련 로직이 들어간 클래스
* Cars: 경주에 참여한 자동차들과 관련된 로직이 들어간 클래스

### 중복된 이름의 참여자 처리하기
입력 값에 대한 예외 중 **"중복된 이름을 가진 참여자"** 를 어떻게 처리할 것인지 고민하였습니다.

이에 대한 고민에 대한 2가지 대처 방법을 생각했습니다.

* 방법1: 중복된 이름을 허용하지 않는다.
* 방법2: 중복된 이름을 가진 참여자의 이름 뒤에 번호를 부여 한다.("java", "java1",...)

방법1은 정말 간단하게 아에 중복을 허용하지 않는 방법입니다. 그러나 저는 충분히 중복된 이름을 가진 참여자가 존재할 가능성이 높다고 생각하여 방법2를 선택하였습니다.

방법2의 경우 다음과 같은 추가 예외 상황을 고려해야 했습니다.

1. 중복된 이름 뒤에 번호를 부여한다
2. 이름의 길이가 5가 넘을 경우를 검증한다

위의 예외 상황을 고려하여 아래와 같이 구현했습니다.

```java
/*
 * InputValidation.java
 */
private static HashMap<String, Integer> register = new HashMap<>();

...(생략)

public static List<String> validateCarNameIsNotDuplicate(List<String> participants) {
    boolean isCarNameNotDuplicate = participants.stream()
            .distinct()
            .count() == participants.size();
    if (!isCarNameNotDuplicate) {
        return checkNameDuplicate(participants);
    }
    return participants;
}

private static List<String> checkNameDuplicate(List<String> participants) {
    return participants.stream()
            .map(InputValidation::nameGenerator)
            .collect(Collectors.toList());
}

private static String nameGenerator(String name) {
    int order = 0;
    if (register.containsKey(name)) {
        order = register.get(name);
        register.put(name, order+1);
        register.put(name+order, 1);
        name += order;
    }
    if (!register.containsKey(name)) {
        register.put(name, 1);
    }
    return name;
}
```

이름의 출현 횟수를 `register` 라는 Map에 저장하여 계속적으로 업데이트를 해주었고 이미 참여 기록이 있는지 확인하고 `nameGenerator` 메서드에서 Map에 저장된 출현 횟수를 이름 뒤에 추가하여 반환합니다 그리고 리스트에 저장합니다.

* `validateCarNameIsNotDuplicate`: 이름의 중복이 있는지 검증하는 메소드
* `checkNameDuplicate`: 중복된 이름의 변환한 결과를 리스트로 반환하는 메소드
* `nameGenerator`: 중복된 이름 뒤에 번호를 부여하는 메소드

## 미션을 통해 느낀점 or 배운점
2주차는 좀 더 기능 단위 커밋에 익숙해 질 뿐 아니라 자바 컨벤션과 객체지향 프로그래밍에 대해 좀 더 익숙해 질 수 있었습니다.

또한, 클래스의 역할과 범위에 대해서 고민하다보니 클래스를 어떻게 분리하는 것이 좋을지 아이디어가 떠올랐습니다.

그러나 클래스의 역할을 어디까지, 어느 범위까지 한정할지에 대한 명확한 확신에 부족함을 느꼈습니다. 이러한 점은 아직 객체지향에 미숙함을 느낄 수 있었고 객체지향을 더욱 단련해야 함을 느꼈습니다.

* [미션 Repository Link](https://github.com/doorisopen/java-racingcar-precourse/tree/doorisopen)