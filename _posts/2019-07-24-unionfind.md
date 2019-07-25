---
layout: post
title:  "합집합 찾기(Union-Find)"
date:   2019-07-24 18:32:30
author: me
categories: Algorithm
tags:	Union Find Search Algorithm
cover:  "/assets/instacode.png"
---

## 합집합 찾기(Union-Find) 이란?


### 합집합 찾기(Union-Find)의 특징


### 합집합 찾기(Union-Find)의 알고리즘 예시


### Source Code


#### C++ STL Library를 사용한 합집합 찾기(Union-Find) 구현

{% highlight javascript %}
#include <stdio.h>

// 부모 노드를 찾는 함수 
int getParent(int parent[], int x) {
	if(parent[x] == x) return x;
	return parent[x] = getParent(parent, parent[x]);
} 
// 두 부모 노드를 합치는 함수 
int unionParent(int parent[],int a, int b){
	a = getParent(parent, a);
	b = getParent(parent, b);
	if(a < b) parent[b] = a;
	else parent[a] = b;
	
}

// 같은 부모를 가지는지 확인 == 같은 그래프인지 확인 
int findParent(int parent[], int a, int b){
	a = getParent(parent, a);
	b = getParent(parent, b);
	if(a == b) return 1;
	return 0;
}

int main(void){
	int parent[11];
	for(int i = 0; i <= 10; i++){
		parent[i] = i;
	}
	unionParent(parent, 1, 2);
	unionParent(parent, 2, 3);
	unionParent(parent, 3, 4);
	unionParent(parent, 5, 6);
	unionParent(parent, 6, 7);
	unionParent(parent, 7, 8);
	
	printf("1과 5가 연결되어있나? %d \n", findParent(parent, 1, 5));
	unionParent(parent, 1, 5);
	printf("1과 5가 연결되어있나? %d \n", findParent(parent, 1, 5));
	
	return 0;
}
{% endhighlight %}



### 관련된 Post


### References
> * <a href="https://www.youtube.com/watch?v=AMByrd53PHM&list=PLRx0vPvlEmdDHxCvAQS1_6XV4deOwfVrz&index=18">18강 - 합집합 찾기(Union-Find)<a>