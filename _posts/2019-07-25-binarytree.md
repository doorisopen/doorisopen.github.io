---
layout: post
title:  "이진 트리(Binary Tree)"
date:   2019-07-25 18:32:30
author: me
categories: Algorithm
tags:	Binary Tree Algorithm
cover:  "/assets/instacode.png"
---

## 이진 트리(Binary Tree) 이란?


### 이진 트리(Binary Tree)의 특징
간선을 거리가 짧은 순서대로 그래프에 포함시킨다.

### 이진 트리(Binary Tree)의 알고리즘 예시


### Source Code


#### 이진 트리(Binary Tree) C++ 구현

{% highlight javascript %}
#include <iostream>

using namespace std;

int num = 15;

// 하나의 노드 정보를 선언한다.
typedef struct node *treePointer;
typedef struct node {
	int data;
	treePointer leftChild, rightChild;
} node;

// 중위 순회
void inorder(treePointer ptr){
	if(ptr){
		inorder(ptr->leftChild);
		cout << ptr->data << ' ';
		inorder(ptr->rightChild);
	}
}
// 전위 순회 
void preorder(treePointer ptr){
	if(ptr){
		cout << ptr->data << ' ';
		preorder(ptr->leftChild);
		preorder(ptr->rightChild);
	}
}
//후위 순회 
void postorder(treePointer ptr){
	if(ptr){	
		postorder(ptr->leftChild);
		postorder(ptr->rightChild);
		cout << ptr->data << ' ';
	}
}

int main(void){
	// 데이터가 들어갈 배열을 생성 
	node nodes[num + 1];
	// 각 원소를 초기화 
	for(int i = 1; i <= num; i++){
		nodes[i].data = i;
		nodes[i].leftChild = NULL;
		nodes[i].rightChild = NULL;
	}
	// 각 노드를 연결해준다. 
	for(int i = 1; i <= num; i++){
		// 2의 배수라면 짝수라면 왼쪽에 넣어준다. 
		if(i % 2 == 0){
			nodes[i / 2].leftChild = &nodes[i];
		}
		// 홀수라면 오른쪽에 넣어준다.
		else {
			nodes[i / 2].rightChild = &nodes[i];
		}
	}
	cout<<"pre : ";
	preorder(&nodes[1]);
	cout<<endl;
	cout<<"in : ";
	inorder(&nodes[1]);
	cout<<endl;
	cout<<"post : ";
	postorder(&nodes[1]); 
	
	return 0;
}
{% endhighlight %}



### 관련된 Post


### References
> * <a href="https://blog.naver.com/ndb796/221233560789">19. 이진 트리의 구현과 순회(Traversal) 방식<a>