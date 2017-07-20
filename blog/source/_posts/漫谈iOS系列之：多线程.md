# 漫谈iOS系列之：多线程
***
####目录
- 线程基本概念
- pThread
- NSThread
- GCD
- NSOperation

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


> 参考 [dispatch_barrier_sync和dispatch_barrier_async](http://www.jianshu.com/p/9ed95082f256)
> [iOS开发：深入理解GCD 第二篇（dispatch_group、dispatch_barrier、基于线程安全的多读单写）](http://www.cnblogs.com/ziyi--caolu/p/4900650.html)





