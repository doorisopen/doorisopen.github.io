---
layout: post
title:  "계수 정렬(Counting Sort)"
date:   2019-07-17 10:30:59
author: me
categories: Sort
tags:	Counting Sort Algorithm
cover:  "/assets/instacode.png"
---

## 계수 정렬(Counting Sort) 이란?


### 계수 정렬(Counting Sort) 알고리즘의 특징


### 계수 정렬(Counting Sort)의 알고리즘 예시


### Source Code

{% highlight javascript %}
#include <stdio.h>

int main(void){
	int tmp;
	int count[5];
	int array[30] = {
		1, 3, 2, 4, 3, 2, 5, 3, 1, 2,
		3, 4, 4, 3, 5, 1, 2, 3, 5, 2,
		3, 1, 2, 5, 3, 4, 1, 2, 1, 1
	};
	for(int i = 0; i < 5; i++){
		count[i] = 0;
	}
	for(int i = 0; i < 30; i++){
		count[array[i] - 1]++;
	}
	for(int i = 0; i < 5 ; i++){
		if(count[i] != 0){
			for(int j = 0; j < count[j]; j++){
				printf("%d ", i+1);
			}
		}
	}
	
	return 0;
} 
{% endhighlight %}


### 계수 정렬(Counting Sort)의 시간복잡도
>
> 


### 정렬 알고리즘 시간복잡도 비교

<a href="/assets/images/sort/sorting_bigo_comp.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="/assets/images/sort/sorting_bigo_comp.JPG" title="Check out the image">
</a>


### 관련된 Post



### References
> * <a href="https://blog.naver.com/ndb796/221228361368">https://blog.naver.com/ndb796/221228361368<a>

