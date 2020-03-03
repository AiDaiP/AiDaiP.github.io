---
layout: post
title:  "c++STL-vector"
date:   2020-3-3
desc: ""
keywords: ""
categories: [Binary]
tags: []
icon: icon-html
---

# c++STL-vector

可变长的动态数组

```
#include<vector>
```

在命名空间std中

## 构造与初始化

```c
#include<iostream>
#include<vector>

int main(int argc, char const *argv[])
{
	std::vector<int> v1;//初始化为空
	std::vector<int> v2(5);//初始化为有5个元素
	std::vector<int> v3(5,1);//初始化为有5个值为1的元素
	std::vector<int> v4(v3);//v4是v3的副本
	std::vector<int>::iterator it = v3.begin();
	std::vector<int> v5(it,it+2);//使用it到it+2初始化v5
	for (int i = 0; i < v3.size(); ++i)
	{
		std::cout<<v3[i]<<" ";
	}
	std::cout<<std::endl;
	for (int i = 0; i < v4.size(); ++i)
	{
		std::cout<<v4[i]<<" ";
	}
	std::cout<<std::endl;
	for (int i = 0; i < v5.size(); ++i)
	{
		std::cout<<v5[i]<<" ";
	}
	std::cout<<std::endl;
	return 0;
}
/*
1 1 1 1 1
1 1 1 1 1
1 1
*/
```



## 大小

* v.max_size()

  返回向量类型的最大容量

* v.capacity()

  返回当前开辟的空间大小

* v.empty

  为空返回true

* v.size()

  返回元素个数

* v.resize(n)

  更改size使其能容纳n个元素

## 访问

* v.at(index)

  返回索引为index的元素，等价于v[index]

* v.front()

  返回第一个元素的引用

* v.back()

  返回最后一个元素的引用

## 赋值

* v[index] = val

  将val赋给索引为index的元素

* v.at(index) = val

  将val赋给索引为index的元素

* v.assign(begin,end)

  使用迭代器begin到end范围内的元素重置v

* v.assign(n,val)

  用n个val重置v

* v.swap(v1)

  v和v1交换

* v.reverse()

  反转v

## 插入

* v.push_back(val)

  将val插入到尾部

* v.insert(it,val)

  将val插入到迭代器it指向的元素前，返回指向新元素的迭代器

* v.insert(it,n,val)

  将n个val迭代器it指向的元素前

## 删除

* v.pop_back()

  删除最后一个元素

* v.erase(it)

  删除迭代器it指向的元素，返回被删除元素后一个元素的迭代器

* v.erase(begin,end)

  删除迭代器begin到end间的元素，返回end后一个元素的迭代器

## 迭代器

* v.begin()

  返回指向第一个元素的迭代器

* v.end()

  返回指向最后一个元素的下一个位置的迭代器

  只作为结束游标

* v.rbegin()

  返回指向最后一个元素的反向迭代器

* v.rend()

  返回指向第一个元素前一个位置的反向迭代器

## algorithm

```
#include<algorithm>
```

* sort(begin,end)

  迭代器begin到end间(不包括end)的元素从小到大排列

* find(begin,end,val)

  在迭代器begin到end间(不包括end)查找val