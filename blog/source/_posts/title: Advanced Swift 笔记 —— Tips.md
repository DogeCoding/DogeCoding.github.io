title: Advanced Swift 笔记 —— Tips
tags: Reading Notes
categories: iOS
date: 2017/8/20 17:35
---

### Curried Function 柯里化函數
一个函数不是接受多个参数，而是只接受部分参数，然后返回一个接受其余参数的函数。

### Statically Dispatched 靜態派發
定義在類或者協議中的函數就是方法(Method)，他們有一個隱式的`self`。不是方法的函數叫做自由函數(Free Function)。自由函数和那些在结构体上调用的方法是静态派发 (statically dispatched)的。对于这些函数的调用，在编译的时候就已经确定了。对于静态派发的调用，编译器可能能够内联 (inline)这些函数，也就是说，完全不去做函数调用，而是将这部分代码替换为需要执行的函数。静态派发还能够帮助编译器丢弃或者简化那些在编译时就能确定不会被实际执行的代码。

