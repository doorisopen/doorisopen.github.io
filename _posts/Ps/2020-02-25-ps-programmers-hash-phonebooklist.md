---
layout: post
title:  "[Programmers] 해시(hash) 전화번호 목록"
date:   2020-02-25 16:22:59
author: me
categories: Ps
tags:	Problem Solving Ps Algorithm C++
cover:  "/assets/instacode.png"
---



해시 문제 출처 [전화번호 목록 - 문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42577)이 곳 입니다.


## 문제 설명
전화번호부에 적힌 전화번호 중, 한 번호가 다른 번호의 접두어인 경우가 있는지 확인하려 합니다.
전화번호가 다음과 같을 경우, 구조대 전화번호는 영석이의 전화번호의 접두사입니다.

* 구조대 : 119
* 박준영 : 97 674 223
* 지영석 : 11 9552 4421
전화번호부에 적힌 전화번호를 담은 배열 phone_book 이 solution 함수의 매개변수로 주어질 때, 어떤 번호가 다른 번호의 접두어인 경우가 있으면 false를 그렇지 않으면 true를 return 하도록 solution 함수를 작성해주세요.

## 제한 사항
* phone_book의 길이는 1 이상 1,000,000 이하입니다.
* 각 전화번호의 길이는 1 이상 20 이하입니다.

## 접근 방법

1. phone_book의 사이즈 만큼 반복 ex. phone_book -> [119, 97674223, 1195524421] 일때 __사이즈는 3__: `for(int i = 0; i < phone_book.size(); i++)`
2. `string prefix = phone_book[i]` 값을 포함하는 문자열이 있는지 탐색 (i != j)
3. `phone_book[j].find(prefix)`는 __phone_book[j]에서 prefix를 포함하면 문자열의 첫 번째 인덱스를 반환 없으면 string::npos를 반환__ 한다.
4. 만약 발견되면 false 리턴한다

* `.find()` : 찾는 문자열의 첫번째 인덱스를 반환하는 함수
* `string::npos` : 찾는 문자열이 없는 경우에 `string::npos`를 반환


## 소스코드(C++)
```
#include <string>
#include <vector>

using namespace std;

bool solution(vector<string> phone_book) {
    for(int i = 0; i < phone_book.size(); i++) {
        string prefix = phone_book[i];
        for(int j = 0; j < phone_book.size(); j++) {
            if(i != j && phone_book[j].find(prefix) == 0 && phone_book[j].find(prefix) != string::npos) return false;
        }
    } 
    return true;
}
```