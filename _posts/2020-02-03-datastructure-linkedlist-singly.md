---
layout: post
title:  "단 방향 연결 리스트(LinkedList)"
date:   2020-02-03 18:04:30
author: me
categories: Datastructure
tags:	DataStructure Singly Linked List
cover:  "/assets/instacode.png"
---


# 연결 리스트(Linked List)
* 연결 리스트는 __길이가 정해져 있지 않은__ 데이터의 연결된 집합 이다
* 연결 리스트는 일렬로 연결된 데이터를 저장할 때 사용하는 자료구조이다
* 연결 리스트의 각각의 데이터는 `노드`라고 부르고 각 노드는 `data 필드`와 다음 data를 바라보는 `주소값(next) 필드`로 이뤄져있다  
* 연결 리스트는 `단 방향/양 방향` 연결리스트가 있다.

<a href="{{ site.2020_datastructure_img }}/linkedlist1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2020_datastructure_img }}/linkedlist1.JPG" title="Check out the image">
</a>

### 배열(Array)과 연결 리스트(Linked List)
배열과 연결 리스트는 아주 유사한 형태를 가지고 있지만 차이가 있다. 그것은 __배열__ 의 경우 한번 선언하면 __늘이거나 줄일 수 없다__ 는 것 이다.

# 단 방향 연결 리스트 Code(C++)
```
#include <iostream>

using namespace std;

class Node {
	public:
		int data;
		Node* next = NULL;
};

class Link {
	public:
		Node* head = new Node();
		
		void addData(int d); // 데이터 추가
		void deleteData(int d); // 데이터 삭제
		const void printList(); // 리스트 출력
		
	private:
		int size = 0;
};

	void Link::addData(int d) {
		if(size == 0) {
			head->data = d;
			head->next = NULL;
		} else {
			Node *newNode = new Node();
			newNode->data = d;
			newNode->next = NULL;
			
			Node *tempNode = head;
			
			while(tempNode->next != NULL) {
				tempNode = tempNode->next;
			}
			tempNode->next = newNode;
		}
		++size;
	}
	
	void Link::deleteData(int d) {
		Node* tempNode = head;
		
		while(tempNode->next != NULL) {
			if(tempNode->next->data == d) {
				tempNode->next = tempNode->next->next;
			} else {
				tempNode = tempNode->next;
			}
		}
	}
	
	const void Link::printList() {
		Node *tempNode = head;
		int tempSize = size;
		while(tempNode->next != NULL) {
			cout << tempNode->data << "->";
			tempNode = tempNode->next;
		}
		cout << tempNode->data << endl;
	}	

int main(void) {
	Link l;
	l.addData(1);
	l.addData(2);
	l.addData(3);
	l.addData(4);
	l.printList();
	l.deleteData(3);
	l.deleteData(2);
	l.printList();
	return 0;
}
```
이 코드는 첫 번째 노드인 head 부터가 아닌 다음 노드 부터 탐색을 한다

만약 head 노드 삭제할경우 다른 노드들이 삭제된 노드를 가지게 되고 문제가 발생할 가능성이 있는 코드이다

그렇기 때문에 이 코드에서는 첫 번째 노드는 삭제를 하지 않는 것 으로 한다

# Reference
> * [엔지니어대한민국](https://www.youtube.com/watch?v=DzGnME1jIwY)