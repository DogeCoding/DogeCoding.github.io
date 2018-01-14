title: STL常见用法 — vector
tags: STL
categories: CPP
date: 2017/7/21 13:06
---

> 代码敲得少，STL总是用的不熟，便自己整理一些用法，有眼缘的顺手用着吧。 

***
## 简介
vector是线性容器,它的元素严格的按照线性序列排序,和动态数组很相似,和数组一样,它的元素存储在一块连续的存储空间中,这也意味着我们不仅可以使用迭代器(iterator)访问元素,还可以使用指针的偏移方式访问,和常规数组不一样的是,vector能够自动存储元素,可以自动增长或缩小存储空间。

## 用法
### 头文件

```
#include <vector>
```
### 构造函数

```
// 1
template<class T, class Allocator = std::allocator<T> > class vector;

// 2
namespace pmr {
    template <class T>
    using vector = std::vector<T, std::pmr::polymorphic_allocator<T>>;
}
```
> 1. std::vector is a sequence container that encapsulates dynamic size arrays.
> 2. std::pmr::vector is an alias template that uses a polymorphic allocator

### 成员函数
|成员函数|实现操作|
|----|----|
|c.assign(beg,end)|将[beg; end)区间中的数据赋值给c|
|c.assign(n,elem)|将n个elem的拷贝赋值给c|
|c.at(idx)|传回索引idx所指的数据，如果idx越界，抛出out_of_range|
|c.back()|传回最后一个数据，不检查这个数据是否存在|
|c.begin()|传回迭代器重的可一个数据|
|c.capacity()|返回容器中数据个数|
|c.clear()|移除容器中所有数据|
|c.empty()|判断容器是否为空|
|c.end()|指向迭代器中的最后一个数据地址|
|c.erase(pos)|删除pos位置的数据，传回下一个数据的位置|
|c.erase(beg,end)|删除[beg,end)区间的数据，传回下一个数据的位置|
|c.front()|传回第一个数据|
|get_allocator|使用构造函数返回一个拷贝|
|c.insert(pos,elem)|在pos位置插入一个elem拷贝，传回新数据位置|
|c.insert(pos,n,elem)|在pos位置插入n个elem数据。无返回值|
|c.insert(pos,beg,end)|在pos位置插入在[beg,end)区间的数据。无返回值|
|c.max_size()|返回容器中最大数据的数量|
|c.pop_back()|删除最后一个数据|
|c.push_back(elem)|在尾部加入一个数据|
|c.rbegin()|传回一个逆向队列的第一个数据|
|c.rend()|传回一个逆向队列的最后一个数据的下一个位置|
|c.resize(num)|重新指定队列的长度|
|c.reserve()|保留适当的容量|
|c.size()|返回容器中实际数据的个数|
|c1.swap(c2)|将c1和c2元素互换|
|swap(c1,c2)|同上操作|
|vector<Elem> c|创建一个空的vector|
|vector <Elem> c1(c2)|复制一个vector|
|vector <Elem> c(n)|创建一个vector，含有n个数据，数据均已缺省构造产生|
|vector <Elem> c(n, elem)|创建一个含有n个elem拷贝的vector。
|vector <Elem> c(beg,end)|创建一个以[beg;end)区间的vector|
|c.~ vector <Elem>()|销毁所有数据，释放内存|

## 用法演示
- 使用reverse将元素翻转:
需要头文件`#include<algorithm>`

```
reverse(vec.begin(),vec.end());
```
将元素翻转(在vector中，如果一个函数中需要两个迭代器，一般后一个都不包含)

- 使用sort排序:
需要头文件#include<algorithm>，
sort(vec.begin(),vec.end());(默认是按升序排列,即从小到大).
可以通过重写排序比较函数按照降序比较，如下:

定义排序比较函数:

```
bool Comp(const int &a,const int &b)
{
    return a>b;
}
```
调用时:`sort(vec.begin(),vec.end(),Comp)`，这样就降序排序。



