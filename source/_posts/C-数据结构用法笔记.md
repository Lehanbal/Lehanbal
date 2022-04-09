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
2. vector [C++ vector 使用详解_Tyler_Zx的博客-CSDN博客_c++ vector](https://blog.csdn.net/qq_38289815/article/details/105896622)

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

1. 数组

```c++
   int main() {
       int tmp[5] = {1,2,3,4,5};
   
       //auto
       for (auto i : tmp) {
           printf("%d\n", i);
       }
       
       //for sizeof函数计算地址
       for (int i  = 0; i < sizeof (tmp) / sizeof (tmp[0]); i++) {
           printf("%d\n", tmp[i]);
   	   }
       
       //库函数strlen 但是只适用字符串数组
       //for (int i  = 0; i < strlen(tmp); i++) {
       //    printf("%d\n", tmp[i]);
   	   //}
       return 0;
   }
```

2. vector

```c++
//下标访问
vector<int> list;
for (int i = 0; i < list.size(); i++) {
    printf("%d\n", list[i]);
}

//迭代器访问
vector<string> tmp = { "1","2","3","4","5" };
for (vector<string>::iterator iter = tmp.begin(); iter != tmp.end(); iter++)
{
	cout << *iter << endl;
	//下面两种方法都都可以检查迭代器是否为空
	cout << (*iter).empty() << endl;
	cout << iter->empty() << endl; 
}

//迭代器逆序访问
for (vector<string>::reverse_iterator iter = tmp.rbegin(); iter != tmp.rend(); iter++)
{
	cout << *iter << endl;
}
```

### 删除

1. 删除最后一个元素pop_back()

```c++
#include <vector>
#include <iostream>
using namespace std;
 
int main()
{
    vector<int>demo{ 1,2,3,4,5 };
    demo.pop_back();
    //输出 dmeo 容器新的size
    cout << "size is :" << demo.size() << endl;
    //输出 demo 容器新的容量
    cout << "capacity is :" << demo.capacity() << endl;
    for (int i = 0; i < demo.size(); i++) {
        cout << demo[i] << " ";
    }
    return 0;
}

/* 结果
size is :4
capacity is :5
1 2 3 4
*/
```

2. 指定删除一个元素erase(position)

```c++
#include <vector>
#include <iostream>
using namespace std;
 
int main()
{
    vector<int>demo{ 1,2,3,4,5 };
    auto iter = demo.erase(demo.begin() + 1);//删除元素 2
    //输出 dmeo 容器新的size
    cout << "size is :" << demo.size() << endl;
    //输出 demo 容器新的容量
    cout << "capacity is :" << demo.capacity() << endl;
    for (int i = 0; i < demo.size(); i++) {
        cout << demo[i] << " ";
    }
    //iter迭代器指向元素 3
    cout << endl << *iter << endl;
    return 0;
}

/* 结果
size is :4
capacity is :5
1 3 4 5
3
最终容器大小减1，且从被删除的位置后面的元素向前挪一个位置
*/
```

## MAP

我们可以使用map存储这类一对一的数据：

第一个可以称为关键字(key)，每个关键字只能在map中出现一次；

第二个可能称为该关键字的值(value)；

另外需要注意的是，使用 map 容器存储的各个键-值对，键的值既不能重复也不能被修改。换句话说，map 容器中存储的各个键值对不仅键的值独一无二，键的类型也会用 const 修饰，这意味着只要键值对被存储到 map 容器中，其键的值将不能再做任何修改。

```bash
　　	 begin()          #返回指向map头部的迭代器
　	　clear()          #删除所有元素
      count()          #返回指定元素出现的次数
      empty()          #如果map为空则返回true
      end()            #返回指向map末尾的迭代器
      equal_range()    #返回特殊条目的迭代器对
      erase()          #删除一个元素
      find()           #查找一个元素
      get_allocator()  #返回map的配置器
      insert()         #插入元素
      key_comp()       #返回比较元素key的函数
      lower_bound()    #返回键值>=给定元素的第一个位置
      max_size()       #返回可以容纳的最大元素个数
      rbegin()         #返回一个指向map尾部的逆向迭代器
      rend()           #返回一个指向map头部的逆向迭代器
      size()           #返回map中元素的个数
      swap()           #交换两个map
      upper_bound()    #返回键值>给定元素的第一个位置
      value_comp()     #返回比较元素value的函数
```



### 声明

1. 引入头文件

   `#include <map>` 

2. 声明与赋值

```c++
#include <iostream>
#include <map>
using namespace std;

int main() {
    //初始化一
	map<string, int>myMap{ {"a",1},{"b",2} };
    //初始化二
    map<int, int> _map;
    _map[0] = 1;
    _map[1] = 2;
    _map[2] = 3;
    
    //若是map<string, string> _map，赋值方式也一样
    //_map["123"] = "324";

    return 0;
}
```

### 遍历

```c++
#include <iostream>
#include <map>
using namespace std;

int main() {
    map<int, int> _map;
    _map[0] = 1;
    _map[1] = 2;
    _map[2] = 3;

    map<int, int>::iterator iter;
    iter = _map.begin();
    while (iter != _map.end()) {
        printf("%d : %d\n", iter->first, iter->second);
        iter++;
    }

    for (iter = _map.begin(); iter != _map.end(); iter++){
        printf("%d : %d\n", iter->first, iter->second);
    }
    
    //访问一个值
    int val = _map[1];
    //如果访问不存在的值,如果是基本数据类型则为0，如果是string类型，则为""空字符串
    int val1 = _map[4];
    //val1 = 0;

    return 0;
}
```

### 插入

插入元素很简单，可以直接使用`[]`来完成元素的插入

```c++
	//如初始化上    
	//初始化一
	map<string, int>myMap{ {"a",1},{"b",2} };
    //初始化二
    map<int, int> _map;
    _map[0] = 1;
    _map[1] = 2;
    _map[2] = 3;
    
    //若是map<string, string> _map，赋值方式也一样
    //_map["123"] = "324";
```

### 删除

```c++
//删除键为bfff指向的元素
cmap.erase("bfff");

//删除迭代器 key所指向的元素
//find(key)返回的是迭代器
map<string,int>::iterator key = cmap.find("mykey");
if(key!=cmap.end())
{
    cmap.erase(key);
}
 
//删除所有元素
cmap.erase(cmap.begin(),cmap.end());
```

## SET

所有元素都会在插入时自动被排序
set/multiset属于关联式容器，底层结构是用二叉树实现。
**set和multiset区别:**
●set不允许容器中有重复的元素
●multiset允许容器中有 重复的元素

```bash
#交换两个set集合
swap(set)
#判断set是否为空
empty()
#集合大小
size()
```

### 声明

1. 引入头文件 

   ` #include <set>`

2. 构造

```c++
set<int> tmp;
set<int> tmp{1,2,3,4,5};
```

### 遍历

```c++
#include <iostream>
#include <set>

using namespace std;

int main() {
    set<int> tmp{1,2,3,4,5};

    for (auto i = tmp.begin(); i != tmp.end(); i++) {
        printf("%d\n", *i);
    }
    return 0;
}
```

### 插入

关键函数：insert(elem);

int count(elem);可以判断是否有相关元素，返回0或1；

find(elem);返回该元素位置的迭代器

```c++
	set<int>s1;
	//插入数据,插入队尾
  	s1.insert(10);
  	s1.insert(20);
 	s1.insert(30);
 	s1.insert(40);
	//同样可以给定迭代器位置，插入到对应元素的后面(set会自己排序，所以这个插到对应位置没意义)
	s1.insert(s1.find(10),15);
```

### 删除

关键函数：earse();

```c++
//删除30这个元素
s1.erase(30);

//清空set
//等价s1.earse(s1.begin(),s1.end())
s1.clear();

```

### 排序

将重写了operator 方法的类传入构造中

```c++
//逆序排列
class compare1
{
public:
	bool operator ()(int v1, int v2)const 
	{
		return v1 < v2;
	}
};

set<int,compare1>s1
```

