title: 漫谈iOS系列之：多线程
tags: OC
categories: iOS
date: 2017/7/17 11:17
---

***
# 线程基本概念
线程（thread）是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。  
同一进程中的多条线程将共享该进程中的全部系统资源，如虚拟地址空间，文件描述符和信号处理等等。但同一进程中的多个线程有各自的调用栈（call stack），自己的寄存器环境（register context），自己的线程本地存储（thread-local storage）。  
**线程的状态**：
- 新生态（New Thread）
- 可运行态（Runnable）
- 阻塞/非运行态（Not Runnable）
- 死亡态（Dead）

![](https://ooo.0o0.ooo/2017/05/14/5917f384972b9.jpg)

**死锁**：
是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。

**死锁条件**：
1. 互斥条件：所谓互斥就是进程在某一时间内独占资源。
2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件:进程已获得资源，在末使用完之前，不能强行剥夺。
4. 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

# 创建线程的开销
[多线程的代价及上下文切换](http://www.cnblogs.com/ktgu/p/3529144.html)

# pThread
POSIX线程（POSIX Threads，常被缩写为Pthreads）是POSIX的线程标准，跨平台，适用于多种操作系统（类Unix操作系统中，都使用Pthreads作为操作系统的线程，Windows操作系统也有其移植版pthreads-win32），可移植性强，是一套纯C语言的通用API，且线程的生命周期需要程序员自己管理，使用难度较大，所以在实际开发中通常不使用。  
Pthreads API中大致共有100个函数调用，全都以"pthread_"开头，并可以分为四类：

- 线程管理，例如创建线程，等待(join)线程，查询线程状态等。
- 互斥锁（Mutex）：创建、摧毁、锁定、解锁、设置属性等操作。
- 条件变量（Condition Variable）：创建、摧毁、等待、通知、设置与查询属性等操作。
- 使用了互斥锁的线程间的同步管理。

`pThread`在实际开发中基本不使用，所以大概了解下就好了。

# NSThread

# GCD

## dispatch_barrier_async&dispatch_barrier_sync
在队列中，`barrier`块必须单独执行，不能与其他`block`并行。这只对并发队列有意义，并发队列如果发现接下来要执行的`block`是个`barrier block`，那么就一直要等到当前所有并发的`block`都执行完毕，才会单独执行这个`barrier block`代码块，等到这个`barrier block`执行完毕，再继续正常处理其他并发`block`。
> `async`, `sync`两者区别在于`async`将自己的任务插入队列后, 不用等待自己的任务结束, 继续把后面的任务插入队列, 然后等待自己的任务运行结束才执行后面的任务, `sync`将自己的任务插入队列后，需要等待自己的任务运行结束才能将后面的任务插入队列。

```
#import <Foundation/Foundation.h>
 
@interface Person : NSObject
@property (nonatomic, copy) NSString *name;
@end
 
 
#import "Person.h"
 
@interface Person ()
@end
 
static NSString *_name;
static dispatch_queue_t _concurrentQueue;
@implementation Person
- (instancetype)init
{
    if (self = [super init]) {
       _concurrentQueue = dispatch_queue_create("com.person.syncQueue", DISPATCH_QUEUE_CONCURRENT);
    }
    return self;
}
- (void)setName:(NSString *)name
{
    dispatch_barrier_async(_concurrentQueue, ^{
        _name = [name copy];
    });
}
- (NSString *)name
{
    __block NSString *tempName;
    dispatch_sync(_concurrentQueue, ^{
        tempName = _name;
    });
    return tempName;
}
@end
```
# NSOperation
`NSOperation`默认是非并发的，当你调用`-[NSOperation start]`方法时，该方法会等任务结束才会返回；
并发的`NSOperation`是指,当你调用`-[NSOperation start]`后，`NSOperation`会在非当前线程(建立一个`NSThread`，或是`dispatch async`等)执行任务，并在任务结束之前就返回;

需要注意的是，并发行为都需要你自己实现，若要实现并发，你需要做很多额外的工作：

1. 你需要创建一个`subclass`；
2. 除了重载`main`方法，实现并发你还需要至少重载；`start`,`isConcurrent`,`isExecuting`,`isFinished`四个方法；
3. 在`start`里，创建`Thread`或者调用一个异步函数；
4. 更新`isExecuting`，并且发送相应KVO消息；
5. 任务结束后，你还得更新`isExecuting`和`isFinished`，发送相应KVO消息。
实现一个并发的`NSOperation`比较少见，具体如何实现，可以读读文档: [NSOperation Class Reference](https://developer.apple.com/documentation/foundation/operation#//apple_ref/doc/uid/TP40004591-RH2-1173)

大多数情况下`NSOperation`都设计成非并发，这样实现起来会简单很多;
并且，一般会配合`NSOperationQueue`使用，由`NSOperationQueue`来负责执行`NSOperation`，而非直接调用`-[NSOperation start]`。

若有复杂任务需要并发执行，一般也是拆成多个`NSOperation`，由`NSOperationQueue`来并发的执行多个`NSOperation`。


> 参考:
> [关于iOS多线程，你看我就够了](http://www.jianshu.com/p/0b0d9b1f1f19)
>  [dispatch_barrier_sync和dispatch_barrier_async](http://www.jianshu.com/p/9ed95082f256)
> [iOS开发：深入理解GCD 第二篇（dispatch_group、dispatch_barrier、基于线程安全的多读单写）](http://www.cnblogs.com/ziyi--caolu/p/4900650.html)
> [NSOperation的并发和非并发有什么区别呀？](http://ourcoders.com/thread/show/1067/)




