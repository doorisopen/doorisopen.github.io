---
layout: post
title:  "삽입 정렬(Insertion Sort)"
date:   2019-07-10 11:21:59
author: me
categories: Sort
tags:	Insertion Sort Algorithm
cover:  "/assets/instacode.png"
---
## 삽입 정렬(Insertion Sort) 이란?
* 각 숫자를 필요할 때만 적절한 위치에 삽입하는 방법 
* 삽입 정렬 은 버블 정렬<h4>'정렬이 되어있다고 가정'</h4>을 한다는 점에서 특정한 경우에 따라 아주 빠른 속도를 낼 수 있다.
 

### 삽입 정렬(Insertion Sort) 알고리즘의 특징
* <strong>버블 정렬</strong>, <strong>선택 정렬</strong>보다 빠르다.
* <strong>특정한 경우</strong>에 따라 아주 빠른 속도를 낼 수 있다.
  * <strong>거의 정렬된 상태</strong> 에서는 어떤 알고리즘 보다도 빠르다는 특징을 가지고 있다.

### 삽입 정렬(Insertion Sort)의 알고리즘 예시
* 배열에 4 1 5 2 3 이 저장되어 있다고 가정하고 오름차순으로 정렬해 보자.


<a href="/assets/images/sort/insertionsort.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="/assets/images/sort/insertionsort.JPG" title="Check out the image">
</a>


### Source Code
{% highlight javascript %}
#include <stdio.h>

int testcase = 5;

int main(){
	
	// case 1 
	// int num[10] = {1, 10, 5, 8, 7, 6, 4, 3, 2, 9};
	// case 2
	int num[testcase] = {4, 1, 5, 2, 3};
	// case 3 (Best Case) 
	// int num[10] = {2, 3, 4, 5, 6, 7, 8, 9, 10, 1};
	
	int i, j, r, tmp;
	printf("***초기값***\nnum[0~4] : ");
	for(r = 0; r < testcase; r++){
		printf("%d ",num[r]);
	}
	printf("\n");
	for(i=0; i< testcase-1 ; i++){
		j = i;
		while(num[j] > num[j+1]){
			printf("[%d] 이동 num[%d]->num[%d] : ",num[j+1],j+1,j);
			tmp = num[j];
			num[j] = num[j+1];
			num[j+1] = tmp;
			j--;
			for(r = 0; r< testcase; r++){
				printf("%d ",num[r]);
			}
			printf("\n");
		}
	}	
	return 0;
}
{% endhighlight %}


### 삽입 정렬(Insertion Sort)의 시간복잡도
>
> * <h5>삽입 정렬의 시간 복잡도는 O(N^2) 입니다.<h5>
> * <h5>But, Best Case의 경우 시간 복잡도는 O(N) 입니다.<h5>


### 정렬 알고리즘 시간복잡도 비교


<a href="/assets/images/sort/sorting_bigo_comp.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="/assets/images/sort/sorting_bigo_comp.JPG" title="Check out the image">
</a>


### 관련된 Post

### References

> * <a href="https://gmlwjd9405.github.io/2018/05/06/algorithm-insertion-sort.html">https://gmlwjd9405.github.io/2018/05/06/algorithm-insertion-sort<a>
> * <a href="https://ko.wikipedia.org/wiki/%EC%82%BD%EC%9E%85_%EC%A0%95%EB%A0%AC">삽입 정렬 – 위키백과<a>
> * <a href="https://blog.naver.com/ndb796/221226806398">https://blog.naver.com/ndb796/221226806398<a>
