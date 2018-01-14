title: Dispatch
tags: 多线程
categories: iOS
date: 2017/8/2 20:01
---

> Execute code concurrently on multicore hardware by submitting work to dispatch queues managed by the system.

在多核硬件上，通过提交任务到由这个系统管理的派遣队列上，并发的执行代码。

一种把保证回调会在主线程执行的方法:

``` //
#ifndef dispatch_main_async_safe
#define dispatch_main_async_safe(block)\
    if (strcmp(dispatch_queue_get_label(DISPATCH_CURRENT_QUEUE_LABEL), dispatch_queue_get_label(dispatch_get_main_queue())) == 0) {\
    block();\
    } else {\
    dispatch_async(dispatch_get_main_queue(), block);\
    }
#endif
```

``` //
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
for(id obj in array)
    dispatch_group_async(group, queue, ^{
        [self doSomethingIntensiveWith:obj];
    });
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
dispatch_release(group);
 
[self doSomethingWith:array];
```


``` //
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
for(id obj in array)
    dispatch_group_async(group, queue, ^{
        [self doSomethingIntensiveWith:obj];
    });
dispatch_group_notify(group, queue, ^{
    [self doSomethingWith:array];
});
dispatch_release(group);
```

### dispatch_barrier_sync和dispatch_barrier_async

| common | different |
| --- | --- |
| 等待在它前面插入队列的任务先执行完 | dispatch_barrier_sync将自己的任务插入到队列的时候，需要等待自己的任务结束之后才会继续插入被写在它后面的任务，然后执行它们 |
| 等待他们自己的任务执行完再执行后面的任务 | dispatch_barrier_async将自己的任务插入到队列之后，不会等待自己的任务结束，它会继续把后面的任务插入到队列，然后等待自己的任务结束后才执行后面任务 |


> 参考:
> [GCD入门（一）: 基本概念和Dispatch Queue](http://www.dreamingwish.com/article/grand-central-dispatch-basic-1.html)
> [深入理解GCD](https://bestswifter.com/deep-gcd/)
> [底层并发 API](https://objccn.io/issue-2-3/)
> [GCD 中 dispatch_once 的性能与实现](http://blog.jimmyis.in/dispatch_once/)
> [读 Concurrency Programming Guide 笔记（一）](http://www.devtalking.com/articles/read-concurrency-programming-guide-1/)
> [iOS并发编程](https://github.com/ming1016/study/wiki/iOS并发编程)


