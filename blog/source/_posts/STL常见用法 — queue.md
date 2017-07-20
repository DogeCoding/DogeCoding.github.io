# STL常见用法 — queue
> 代码敲得少，STL总是用的不熟，便自己整理一些用法，有眼缘的顺手用着吧。  

## 简介
 队列(queue)是一种特殊的线性表，是一种先进先出(First In First Out)的数据结构，允许在队列末尾插入元素，队列头取出元素，在STL中是用list或者deque实现，封闭头部即可。
## 用法
### 头文件
```
#include <queue>
```
### 构造函数
```
template <class T, class Container = deque<T> > class queue; 
```
队列适配器默认用deque容器实现，也可以指定使用list容器来实现
```
queue <Elem> q;					// 创建一个空的queue，默认使用deque容器
queue <Elem, list<Elem> > q;		// 使用list容器
queue <Elem> q1(q2);				// 复制q2
```
### 成员函数
 成员函数 | 实现操作 
 -------- | ------------
 Elem& back() | 返回队列最后一个元素 
 bool empty()const | 如果队列为空，返回true，否则返回false 
 Elem& front() | 返回队列第一个元素 
 void pop() | 移除队列中的第一个元素 
 void push(const Elem& e) | 在队列末尾插入元素e 
 size_type size()const | 返回队列中的元素数目 
## 用法演示
```
#include <stdio.h>  
#include <queue>  
  
using namespace std;  
  
int main()  
{  
    int n, m, size;  
    queue <int> q;			//定义队列q  
  
    q.push(1);  
    q.push(2);				//将1 、2压入队列  
  
    while (!q.empty())		//判断队列是否为空  
    {  
        n = q.front();		//返回队列头部数据  
        m = q.back();			//返回队列尾部数据  
        size = q.size();		//返回队列里的数据个数  
        q.pop();				//队列头部数据出队  
        printf("%d %d %d\n", n, m, size);  
    }  
    return 0;  
} 
```







