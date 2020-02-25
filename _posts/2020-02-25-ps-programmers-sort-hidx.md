---
layout: post
title:  "[Programmers] 정렬(sort) H-Index"
date:   2020-02-25 16:22:59
author: me
categories: Ps
tags:	Problem Solving Ps Algorithm C++
cover:  "/assets/instacode.png"
---



정렬 문제 출처 [H-Index - 문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42747)이 곳 입니다.


## 문제 설명
H-Index는 과학자의 생산성과 영향력을 나타내는 지표입니다. 어느 과학자의 H-Index를 나타내는 값인 h를 구하려고 합니다. 위키백과1에 따르면, H-Index는 다음과 같이 구합니다.

어떤 과학자가 발표한 논문 n편 중, h번 이상 인용된 논문이 h편 이상이고 나머지 논문이 h번 이하 인용되었다면 h가 이 과학자의 H-Index입니다.

어떤 과학자가 발표한 논문의 인용 횟수를 담은 배열 citations가 매개변수로 주어질 때, 이 과학자의 H-Index를 return 하도록 solution 함수를 작성해주세요.

## 제한 사항
* 과학자가 발표한 논문의 수는 1편 이상 1,000편 이하입니다.
* 논문별 인용 횟수는 0회 이상 10,000회 이하입니다.

## 접근 방법
문제 설명을 이해하기 어려웠던 문제다

만약 n개의 논문이 있다고 할때 __h번 이상__ 인용된 논문이 __h편 이상__ 이면 h가 H-Index 이다.

예를 들면 n = 5, 5편의 논문이 있고 citations(인용) 횟수 [3, 0, 6, 1, 5] 배열이 있다고 할때 

1. citations를 정렬한다. -> [0, 1, 3, 5, 6]
2. 인용의 횟수 0 부터 논문의 개수 5까지 for loop를 돈다. `for(int i = 0; i <= citations.size(); i++)`
3. `for(int j = 0; j < citations.size(); j++)` i 번 이상 인용한 논문의 개수를 구한다. 
4. `if(i <= citations[j])` 만약 i = 0 일때, __0번 이상__ 인용한 논문이라면 `h++`이고 j가 끝까지 돌면 h = 5, 5편이다.
5. `if(i <= h)` __i(인용 횟수)__ 가 __h(i번 이상 인용한 논문의 개수)__ 보다 작거나 같으면 H-Index가 된다.

ex 1) </br>
citations -> [0, 1, 3, 5, 6] answer: 3</br>

i=0, h = 5, answer = 0 </br>
i=1, h = 4, answer = 1 </br>
i=2, h = 3, answer = 2 </br>
i=3, h = 3, answer = 3 </br>
i=4, h = 2, answer = 3 </br>
i=5, h = 2, answer = 3 </br>

ex 2) </br>
citations -> [0, 0, 0, 0] answer: 0</br>

i=0, h = 4, answer = 0 </br>
i=1, h = 0, answer = 0 </br>
i=2, h = 0, answer = 0 </br>
i=3, h = 0, answer = 0 </br>
i=4, h = 0, answer = 0 </br>

ex 3) </br>
citations -> [2, 2, 2] answer: 2</br>

i=0, h = 3, answer = 0 </br>
i=1, h = 3, answer = 1 </br>
i=2, h = 3, answer = 2 </br>
i=3, h = 3, answer = 2 

## 소스코드(C++)
```
#include <iostream>
#include <string>
#include <algorithm>
#include <vector>

using namespace std;

int solution(vector<int> citations) {
    int answer = 0;
    
    sort(citations.begin(), citations.end());

    for(int i = 0; i <= citations.size(); i++) {
        int h = 0;
        for(int j = 0; j < citations.size(); j++) {
            if(i <= citations[j]) {
               h++;    
            }
        }
        if(i <= h) {
            answer = i;
        }
    }
    
    return answer;
}
```