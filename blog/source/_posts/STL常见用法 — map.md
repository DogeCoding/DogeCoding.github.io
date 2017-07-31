title: STL常见用法 —— map
tags: CPP
---

# STL常见用法 — 
> 代码敲得少，STL总是用的不熟，便自己整理一些用法，有眼缘的顺手用着吧。 

*** 
## 简介
maps是一个关联容器，用来存储key-value式的元素，提供一对一hash。内部的实现，自建一颗红黑树，这棵树具有对数据自动排序的功能。比如一个班级中，每个学生的学号和他的姓名就存在一对一映射的关系。

- 第一个值是关键字(key)，每个关键字只能在map中出现一次
- 第二个值为关键字的值(value)

![](http://oo8snaf4x.bkt.clouddn.com/15014668312515.png?imageView2/0/q/100)


## 用法
### 头文件

``` //
#include <map>

template<
    class Key,
    class T,
    class Compare = std::less<Key>,
    class Allocator = std::allocator<std::pair<const Key, T> >
> class map;
```

### 构造函数

``` //
namespace pmr {
    template <class Key, class T, class Compare = std::less<Key>>
    using map = std::map<Key, T, Compare,
                         std::pmr::polymorphic_allocator<std::pair<const Key,T>>>
}
```

### 成员函数

| 成员函数 | 实现操作 |
| --- | --- |
| begin | 返回一个起始的迭代器 |
| end | 返回一个末尾的迭代器 |
| empty | 检查容器是否为空 |
| size | 返回容器内元素个数 |
| clear | 清除容器内容 |
| insert | 插入元素或节点 |
| erase | 删除元素 |
| swap | 交换内容 |
| count | 返回指定key的元素个数 |
| find | 通过key查找元素 |




## 用法演示

``` //
#include <map>

void mapTest()
{
	std::map<int, int> m;      // 构造函数
	m.insert(pair<int, int>(1, 10));   // 插入元素
	m.insert(pair<int, int>(2, 30));
	m.insert(pair<int, int>(4, 50));
	map<int, int>::iterator it;    // 迭代器
	it = m.find(3);    // 通过key查找元素
	if (it == m.end()) // 如果返回值为尾部迭代器则无此元素
	{
		cout << "Not find" << endl;
		m.insert(pair<int, int>(3, 20));
	}	
	for (it = m.begin(); it != m.end(); it++)  // 通过迭代器遍历map容器
	{
		cout << it->first << " " << it->second << endl;   // 通过迭代器访问元素的key-value
	}

}
```


> 参考:
> [C/C++ - Map (STL) 用法與心得完全攻略](http://mropengate.blogspot.com/2015/12/cc-map-stl.html)








