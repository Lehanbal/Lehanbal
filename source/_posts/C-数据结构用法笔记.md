title: C++数据结构用法笔记
author: Lehanbal
tags:
  - 数据结构
  - C++
categories:
  - C++
  - ''
date: 2022-03-26 19:52:00
---
## 数组
数组类型：
1. 数组
2. vector

### 声明

1. 数组

```c++
//变量类型 变量名[数组大小]
int a[n];

//可以直接进行初始化赋值，未进行赋值的都会使用0作为初始化
int a[5] = {1, 2, 3};//a[3] = 0
```

2. vector

```c++
//需要引入头文件
#include "vector"
//- vector():创建一个空vector
vector<int> arr1;
printf("%d\n", arr1.size());
//- vector(int nSize):创建一个vector,元素个数为nSize
vector<int> arr2(10);
//- vector(int nSize,const t& t):创建一个vector，元素个数为nSize,且值均为t
vector<int> arr3(10, 1);
//- vector(const vector&):复制构造函数
vector<int> arr4(5, 2);
vector<int> arr5(arr4);
//- vector(begin,end):复制[begin,end)区间内另一个数组的元素到vector中
int tmpArr[5] {2, 5, 4,3,2};
vector<int> arr6(tmpArr + 2, tmpArr + 4);
```

### 遍历



### 插入



### 删除



## MAP

### 声明



### 遍历



### 插入



### 删除



## SET

### 声明



### 遍历



### 插入



### 删除



## 链表

### 声明

C

### 遍历



### 插入



### 删除



## 二叉树

### 声明



### 遍历



### 插入



### 删除