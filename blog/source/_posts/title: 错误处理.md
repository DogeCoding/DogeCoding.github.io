title: 错误处理
tags: Swift
categories: iOS
date: 2017/8/3 10:47
---

错误处理（Error handling）是响应错误以及从错误中恢复的过程。Swift 提供了在运行时对可恢复错误的抛出、捕获、传递和操作的一等公民支持。

Swift 中有4种处理错误的方式

- 把函数抛出的错误传递给调用此函数的代码
- 用`do-catch`语句处理错误
- 将错误作为可选类型处理
- 或者断言此错误根本不会发生

> Swift 中的错误处理并不涉及解除调用栈，这是一个计算代价高昂的过程。就此而言，throw语句的性能特性是可以和return语句相媲美的。

一个 throwing 函数可以在其内部抛出错误，并将错误传递到函数**被调用时的作用域**。






> 参考:
> [ The Swift Programming Language 中文版](http://wiki.jikexueyuan.com/project/swift/)
> [SWIFT 的必备 TIP](http://swifter.tips)

