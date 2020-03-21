---
title: c++-use-priority-queue-with-custom-compare-function
date: 2020-03-21 16:13:08
tags: [c++, queue, priority_queue, cmp, operator extend]
---

# C++ 使用带自定义比较函数的优先队列

## 优先队列 priority_queue

### 介绍

C++中自动保持有序的队列，位于 queue 头文件中，能够以 O(logn) 的复杂度取得 n 个元素中的最小元素，不需要自行实现堆排序。此队列默认为大顶堆，可以通过自定义来实现小顶堆。

### 基本用法

#### 定义
```
queue<int> data; // 储存 int 型的从大到小优先队列

queue<int, vector<int>, less<int>> data; // 储存 int 型的从大到小优先队列的等价写法

queue<int, vector<int>, greater<int>> data; // 储存 int 型的从小到大优先队列
```

#### 成员函数

参考 https://www.cplusplus.com/reference/queue/priority_queue/

#### 自定义类型时的比较设定
由于 less<> 与 greater<> 仅支持系统自带的类型，这里有两种方法可以支持自定义类型的比较
- 重载 struct 的 operator <（即重载小于号实现比较，因为内部的排序仅使用小于号比较），来为 less<> 与 greater<> 扩展比较能力
- 传入自定义的 cmp 函数（更准确的说是重载了()操作符的数据结构，因为此处是使用泛型传入的），来完全自定义比较过程

## struct 重载运算符

### 重载语法

重载小于号例子:
```
struct node{
	int data;
	int test;
	bool operator < (const node &b) const {
		return data < b.data;
	}
};

priority_queue<node, vector<node>, greater<node>>
```

## 自定义 cmp 函数

当涉及到指针类型时只能通过这种方法

### 语法

```
struct cmp{
	bool operator() (node* a, node* b){
		return a->data < b->data;
	}
};

priority_queue<node*, vector<node*>, cmp> data;
```


## 完整总结例子

```
#include <iostream>
#include <queue>
#include <vector>

using namespace std;
struct node{
	int data;
	int test;
	bool operator < (const node &b) const {
		return data < b.data;
	}
	
	node(int d){data = d;}
};

struct cmp{
	bool operator() (node* a, node* b){
		return *a < *b;
	}
};

int main(){
	priority_queue<node*, vector<node*>, cmp> data;
	node* new_node;
	
	int input, count;
	cin >> count;
	for(int i = 0; i < count; i++){
		cin >> input;
		new_node = new node(input);
		data.push(new_node);
	}

	node* input1;
	node* input2;
	while(data.size() > 1){
		input1 = data.top();
		data.pop();
		input2 = data.top();
		data.pop();
		input1->data = input1->data +  input2->data;
		delete input2;
		data.push(input1);
	}
	cout << data.top()->data << endl;
	new_node = data.top();
	data.pop();
	delete new_node;
	return 0;
}
```

