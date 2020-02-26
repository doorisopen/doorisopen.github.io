---
layout: post
title:  "[Programmers] 정렬(sort) 가장 큰 수"
date:   2020-02-26 10:47:59
author: me
categories: Ps
tags:	Problem Solving Ps Algorithm C++
cover:  "/assets/instacode.png"
---



정렬 문제 출처 [가장 큰 수 - 문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42746)이 곳 입니다.


## 문제 설명
0 또는 양의 정수가 주어졌을 때, 정수를 이어 붙여 만들 수 있는 가장 큰 수를 알아내 주세요.

예를 들어, 주어진 정수가 [6, 10, 2]라면 [6102, 6210, 1062, 1026, 2610, 2106]를 만들 수 있고, 이중 가장 큰 수는 6210입니다.

0 또는 양의 정수가 담긴 배열 numbers가 매개변수로 주어질 때, 순서를 재배치하여 만들 수 있는 가장 큰 수를 문자열로 바꾸어 return 하도록 solution 함수를 작성해주세요.

## 제한 사항
* numbers의 길이는 1 이상 100,000 이하입니다.
* numbers의 원소는 0 이상 1,000 이하입니다.
* 정답이 너무 클 수 있으니 문자열로 바꾸어 return 합니다.

## 접근 방법
numbers를 string vector에 저장하고 a + b < b + a __(ex. [6, 10, 2] -> 106 < 610)__ 따라서 [10, 6, 2]처럼 배열 끝까지 정렬한다.

정렬이 완성되면 [10, 2, 6]이 되고 배열 마지막 요소 6 부터 answer += 6 해준다. 

마지막 원소까지 합치면 __6210__ 이 된다.

그리고 __조합된 결과가 0 일 경우__ "0"을 return 한다.


## 소스코드(C++)
```
#include <string>
#include <vector>
#include <iostream>
#include <algorithm>
 
using namespace std;
 
bool compare(string &a, string &b) {
    return a + b < b + a ?  true : false;
}
 
string solution(vector<int> numbers) {
    string answer = "";
    vector<string> v;
    
    for (int i = 0; i < numbers.size(); i++) {
        v.push_back(to_string(numbers.at(i)));
    }
    
    sort(v.begin(), v.end(), compare);
    
    while (!v.empty()) {
        answer += v.back();
        v.pop_back();
    }
    
    if (answer[0] == '0')
        return "0";
    
    return answer;
}
```