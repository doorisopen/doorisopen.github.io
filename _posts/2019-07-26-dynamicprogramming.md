---
layout: post
title:  "다이나믹 프로그래밍(Dynamic Programming)"
date:   2019-07-26 16:40:30
author: me
categories: Algorithm
tags:	Dynamic Programming Algorithm
cover:  "/assets/instacode.png"
---

## 다이나믹 프로그래밍(Dynamic Programming) 이란?

<hr/>

### 다이나믹 프로그래밍(Dynamic Programming)의 특징

<hr/>

### 다이나믹 프로그래밍(Dynamic Programming)의 알고리즘 예시

<hr/>

### Source Code


#### 1. Dynamic Programming 기본 형태 구현
{% highlight javascript %}
#include <stdio.h>

int d[100];
// dp 의 기본 형태 
int dp(int x){
	if( x == 1 ) return 1;
	if( x == 2 ) return 1;
	return dp(x-1) + dp(x-2);
}
int main(void){
	printf("%d ",dp(40));
	return 0;
}
{% endhighlight %}

<hr/>

#### 2. Dynamic Programming 메모이제이션(Memoization)이용하여 구현
{% highlight javascript %}
#include <stdio.h>

int d[100];

// 개선한 dp 형태 
int dp(int x){
	if( x == 1 ) return 1;
	if( x == 2 ) return 1;
    // 메모이제이션 이용
	if(d[x] != 0) return d[x];
	return d[x] = dp(x-1) + dp(x-2);
}

int main(void){
	printf("%d ",dp(40));
	return 0;
}
{% endhighlight %}



### 관련된 Post
> * <a href="https://github.com/doorisopen/ProblemSolving/blob/master/BOJ/boj11726.cpp">[BOJ]11726 - 2*n 타일링<a>
> * <a href="https://github.com/doorisopen/ProblemSolving/blob/master/BOJ/boj11727.cpp">[BOJ]11727 - 2*n 타일링2<a>
> * <a href="https://github.com/doorisopen/ProblemSolving/blob/master/BOJ/boj2133.cpp">[BOJ]2133 - 타일 채우기<a>
> * <a href="https://github.com/doorisopen/ProblemSolving/blob/master/BOJ/boj14852.cpp">[BOJ]14852 - 타일 채우기3<a>


### References
> * <a href="https://www.youtube.com/watch?v=LQ3JHknGy8c&list=PLRx0vPvlEmdDHxCvAQS1_6XV4deOwfVrz&index=19">19강 - 크루스칼 알고리즘(Kruskal Algorithm)<a>