---
layout: post
title:  "[Programmers] 완전탐색(Exhaustive Search) 모의고사"
date:   2020-02-17 14:31:59
author: me
categories: Ps
tags:	Problem Solving Ps Algorithm C++
cover:  "/assets/instacode.png"
---

[모의고사 - 문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42840#)


### 문제 설명
수포자는 수학을 포기한 사람의 준말입니다. 수포자 삼인방은 모의고사에 수학 문제를 전부 찍으려 합니다. 수포자는 1번 문제부터 마지막 문제까지 다음과 같이 찍습니다.

* 1번 수포자가 찍는 방식: 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, ...
* 2번 수포자가 찍는 방식: 2, 1, 2, 3, 2, 4, 2, 5, 2, 1, 2, 3, 2, 4, 2, 5, ...
* 3번 수포자가 찍는 방식: 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, ...

1번 문제부터 마지막 문제까지의 정답이 순서대로 들은 배열 answers가 주어졌을 때, 가장 많은 문제를 맞힌 사람이 누구인지 배열에 담아 return 하도록 solution 함수를 작성해주세요.

...(중략)

### 접근 방법
* 1,2,3 수포자들의 찍는 패턴을 배열에 저장
* answers의 사이즈 만큼 각각의 수포자들의 패턴을 반복해야한다. 즉 1번 수포자는 5개의 원소가 반복되는 패턴, 2번 은 8개의 원소가 반복, 3번은 10개
* 각 수포자의 반복되는 원소 개수가 다르므로 __idx2__ 가 0 일때 idx2 = 0으로 초기화를 해줘서 answers 사이즈 만큼 반복 할 수 있게 한다.
* 각 수포자의 패턴과 문제의 정답이 동일할 경우 `if(answers[idx] == p[i][idx2])` 해당 수포자의 정답 개수를 1 증가시킨다. `correct[i] += 1;`
* `correct[]` 에서 최대 값을 구하고 최고 득점을 얻은 수포자들을 answer에 저장하고 리턴

### 코드(C++)

```
// Programmers 완전탐색 모의고사 1번 
#include <iostream>
#include <string>
#include <algorithm>
#include <vector>

using namespace std;

int p[4][11] = {{0},
                {1,2,3,4,5},
                {2,1,2,3,2,4,2,5},
                {3,3,1,1,2,2,4,4,5,5}
               };

vector<int> solution(vector<int> answers) {
    vector<int> answer;
    int size = answers.size();
    int correct[4] = {0,0,0,0};
    
    for(int i = 1; i <= 3; i++) {
        int idx = 0;
        int idx2 = 0;
       while(idx < size) {
         if(p[i][idx2] == 0) {
             idx2 = 0;
         }
         if(answers[idx] == p[i][idx2]) {
            correct[i] += 1;
         }
           idx++;
           idx2++;
        }
    }
    int excellent = *max_element(correct,correct+4);
    for(int i = 1; i <= 3; i++) {
        if(excellent == correct[i]) {
            answer.push_back(i);
        }
    }
    
    return answer;
}

int main(void) {
	vector<int> testcase;
//	testcase.push_back(1);
//	testcase.push_back(2);
//	testcase.push_back(3);
//	testcase.push_back(4);
//	testcase.push_back(5);
	
	testcase.push_back(1);
	testcase.push_back(3);
	testcase.push_back(2);
	testcase.push_back(4);
	testcase.push_back(2);

	for(int i = 0; i < solution(testcase).size(); i++) {
		cout << solution(testcase)[i] << " ";
	}

	return 0;
}
```