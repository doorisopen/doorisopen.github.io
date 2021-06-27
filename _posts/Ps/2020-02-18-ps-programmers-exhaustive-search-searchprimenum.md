---
layout: post
title:  "[Programmers] 완전탐색(Exhaustive Search) 소수 찾기"
date:   2020-02-18 15:56:59
author: me
categories: Ps
tags:	Problem Solving Ps Algorithm C++ Java
cover:  "/assets/instacode.png"
---


완전탐색 문제 출처 [소수 찾기 - 문제 링크](https://programmers.co.kr/learn/courses/30/lessons/42839)이 곳 입니다.


## 문제 설명
한자리 숫자가 적힌 종이 조각이 흩어져있습니다. 흩어진 종이 조각을 붙여 소수를 몇 개 만들 수 있는지 알아내려 합니다.

각 종이 조각에 적힌 숫자가 적힌 문자열 numbers가 주어졌을 때, 종이 조각으로 만들 수 있는 소수가 몇 개인지 return 하도록 solution 함수를 완성해주세요.

## 제한사항
* numbers는 길이 1 이상 7 이하인 문자열입니다.
* numbers는 0~9까지 숫자만으로 이루어져 있습니다.
* 013은 0, 1, 3 숫자가 적힌 종이 조각이 흩어져있다는 의미입니다.

입출력 예 설명
## 예제 #1
[1, 7]으로는 소수 [7, 17, 71]를 만들 수 있습니다.

## 예제 #2
[0, 1, 1]으로는 소수 [11, 101]를 만들 수 있습니다.

11과 011은 같은 숫자로 취급합니다.

## 접근 방법



## 소스 코드(Java)
```
import java.util.HashSet;

class Solution {
    
    public int solution(String numbers) {
        HashSet<Integer> set = new HashSet<>();
        permutation("", numbers, set);
        int cnt = 0;
        while(set.iterator().hasNext()) {
            int num = set.iterator().next();
            set.remove(num);
            if(num == 2) cnt++;
            if(num%2 !=0 && isPrime(num)) cnt++;
        }
        return cnt;
    }
    
    public boolean isPrime(int n) {
        if(n == 0 || n == 1) return false;
        for(int i = 3; i <= (int)Math.sqrt(n); i += 2) {
            if(n % i == 0) return false;
        }
        return true;
    }
    public void permutation(String prefix, String numbers, HashSet<Integer> set) {
        int num_size = numbers.length();
        if(!prefix.equals("")) {
            set.add(Integer.valueOf(prefix));
        }
        for(int i = 0; i < num_size; i++) {
            permutation(prefix + numbers.charAt(i),
                    numbers.substring(0,i) + numbers.substring(i+1,num_size),
                    set);
        }
    }
    
}
```