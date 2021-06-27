---
layout: post
title:  "[Programmers] 스택/큐(Stack/Queue) 탑"
date:   2020-03-18 21:46:59
author: me
categories: Ps
tags:	Problem Solving Ps Algorithm C++
cover:  "/assets/instacode.png"
---


스택/큐 문제 출처 [탑 - 문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42588)이 곳 입니다.


## 문제 설명
수평 직선에 탑 N대를 세웠습니다. 모든 탑의 꼭대기에는 신호를 송/수신하는 장치를 설치했습니다. 발사한 신호는 신호를 보낸 탑보다 높은 탑에서만 수신합니다. 또한, 한 번 수신된 신호는 다른 탑으로 송신되지 않습니다.

예를 들어 높이가 6, 9, 5, 7, 4인 다섯 탑이 왼쪽으로 동시에 레이저 신호를 발사합니다. 그러면, 탑은 다음과 같이 신호를 주고받습니다. 높이가 4인 다섯 번째 탑에서 발사한 신호는 높이가 7인 네 번째 탑이 수신하고, 높이가 7인 네 번째 탑의 신호는 높이가 9인 두 번째 탑이, 높이가 5인 세 번째 탑의 신호도 높이가 9인 두 번째 탑이 수신합니다. 높이가 9인 두 번째 탑과 높이가 6인 첫 번째 탑이 보낸 레이저 신호는 어떤 탑에서도 수신할 수 없습니다.

## 제한 사항
* heights는 길이 2 이상 100 이하인 정수 배열입니다.
* 모든 탑의 높이는 1 이상 100 이하입니다.
* 신호를 수신하는 탑이 없으면 0으로 표시합니다.

## 접근 방법
* [6 9 5 7 4] 순으로 탑의 높이(heights)가 주어질때, heights를 모두 stack에 넣는다.
* 그러면 4(top) 7 5 9 6 순으로 스택에 저장이 된다.
* top을 변수에 저장하고 스택을 pop 해준다(한번만)
* top을 가진 변수 t(=4)를 [6 9 5 7 __4__] 차례로 뒤에서(heights[4])부터 t보다 큰 높이의 탑이 있는지 비교한다.
* 만약 더 큰 탑이 있다면 answer에 해당 인덱스를 저장한다.   


## 소스코드(C++)

```
#include <iostream>
#include <string>
#include <stack>
#include <vector>

using namespace std;

vector<int> solution(vector<int> heights) {
    vector<int> answer(heights.size());
    stack<int> s;
    
    for(int i = 0; i < heights.size(); i++) {
        s.push(heights[i]);
    }
    
    int t_idx = heights.size()-1;
    while(!s.empty()) {
        int t = s.top();
        s.pop();
    
        for(int i = t_idx-1; i >= 0; i--) {
            if(heights[i] > t) {
                // cout << heights[i] << ">" << t << endl;
                answer[t_idx] = i+1;
                break;
            }
        }
        t_idx--;
    }
    
    return answer;
}
```