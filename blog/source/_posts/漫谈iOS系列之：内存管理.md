# 漫谈iOS系列之：内存管理
####目录
- 引用计数(Reference Counting)
- MRC
- ARC
- 循环引用
- Autorelease
- 修饰词

***
#引用计数
[推荐一篇来自@杨萧玉的引用计数原理Blog](http://yulingtianxia.com/blog/2015/12/06/The-Principle-of-Refenrence-Counting/)
***
- **简介**  
iOS中对内存管理的机制（堆内存），每一个对象都有一个与之关联的引用计数（Reference Counting）。当一个对象“被拥有”的时候引用计数+1，当一个对象引用计数为零时该对象被释放。  
- **比拟**  
比如上班，最早进入办公室的人需要开灯，之后进入办公室的人需要照明， 下班离开办公室的人不需要照明，最后离开办公室的人需要关灯。  
这样对应的引用计数就是：第一个人进入办公室开灯，引用计数是1。之后进入办公室需要照明引用计数是2。下班一个人离开办公室引用计数变成了1，最后一个离开了办公室，引用计数变成了0 。  
- **引用计数如何储存**
 1. TaggedPointer  
 [一篇极好的文章](http://www.cocoachina.com/ios/20150918/13449.html)  
总体来说，我的理解是如果一个对象使用了**Tagged Pointer**技术（比如*NSString*，*NSNumber*等），指针里面会直接存数据内容，不会再作为“指针”指向其它地址，从Runtime来理解就是不会使用isa指针，也就不会继承苹果的内存管理方式（Reference Counting）。  
判断当前对象是否在使用 TaggedPointer 是看标志位是否为1:  
```
#if SUPPORT_MSB_TAGGED_POINTERS
#   define TAG_MASK (1ULL<<63)
#else
#   define TAG_MASK 1
inline bool 
objc_object::isTaggedPointer() 
{
#if SUPPORT_TAGGED_POINTERS
    return ((uintptr_t)this & TAG_MASK);
#else
    return false;
#endif
}
```
 2. isa 指针
 指针的内存空间很大，有时候可以优化指针，在指针中存储一部分内容。下面列出不同架构下的64位环境中`isa`指针结构:  

```
union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;

#if SUPPORT_NONPOINTER_ISA
# if __arm64__
#   define ISA_MASK        0x00000001fffffff8ULL
#   define ISA_MAGIC_MASK  0x000003fe00000001ULL
#   define ISA_MAGIC_VALUE 0x000001a400000001ULL
    struct {
        uintptr_t indexed           : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 30; // MACH_VM_MAX_ADDRESS 0x1a0000000
        uintptr_t magic             : 9;
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 19;
#       define RC_ONE   (1ULL<<45)
#       define RC_HALF  (1ULL<<18)
    };

# elif __x86_64__
#   define ISA_MASK        0x00007ffffffffff8ULL
#   define ISA_MAGIC_MASK  0x0000000000000001ULL
#   define ISA_MAGIC_VALUE 0x0000000000000001ULL
    struct {
        uintptr_t indexed           : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 44; // MACH_VM_MAX_ADDRESS 0x7fffffe00000
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 14;
#       define RC_ONE   (1ULL<<50)
#       define RC_HALF  (1ULL<<13)
    };

# else
    // Available bits in isa field are architecture-specific.
#   error unknown architecture
# endif

// SUPPORT_NONPOINTER_ISA
#endif

};
```
只有**arm64**架构的设备支持优化，下面列出了`isa`指针中变量对应的含义: 
 
| 变量名 | 含义 |
| --- | --- |
| indexed | 0 表示普通的`isa`指针，1 表示使用优化，存储引用计数 |
| has_assoc | 表示该对象是否包含 associated object，如果没有，则析构时会更快 |
| has_cxx_dtor | 表示该对象是否有 C++ 或 ARC 的析构函数，如果没有，则析构时更快 |
| shiftcls | 类的指针 |
| magic | 固定值为 0xd2，用于在调试时分辨对象是否未完成初始化 |
| weakly_referenced | 表示该对象是否有过`weak`对象，如果没有，则析构时更快 |
| deallocating | 表示该对象是否正在析构 |
| has_sidetable_rc | 表示该对象的引用计数值是否过大无法存储在`isa`指针 |
| extra_rc | 存储引用计数值减一后的结果 |
  3. 散列表
散列表来存储引用计数具体是用DenseMap类来实现，实现中有锁保证其安全性。


- 获取引用计数  
在MRC环境下可以使用```retainCount```方法获取某个对象的引用计数。  
在ARC环境下可以使用Core Foundation 库的```CFGetRetainCount((__bridge CFTypeRef)(obj))```方法和Runtime的```_objc_rootRetainCount()```方法来获取引用计数，也可以使用KVC技术来获取```valueForKey:@"retainCount"```。注意以上方法不是线程安全的。
- 注意  
NSString 定义的对象是保存在字符串常量区，没有用引用计数管理内存，如果输出其```retainCount```，为-1。  
 
> ![retainCount](http://upload-images.jianshu.io/upload_images/2590408-a0c222416dbc63a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 注意其中的**Do not use this method**。   


#MRC(Manual Reference Counting)
***
MRC从字面上理解就是手动管理引用计数，也就是手动管理内存。相关的内存管理方法有`retain`，`release`，`autorelease`，其中`retain`方法是对引用计数+1，相应的`release`是对引用计数-1，`autorelease`是将对象加入自动释放池，下文会讲到。

- 示例代码

 ```
 // 以预定Person类为例
 Person* person = [[Person alloc] init];    // 申请对象，此时引用计数=1
 [person retain];  //此时引用记数+1，现为2
 [person release];   //引用计数-1，此时引用计数=1
 [person release];   //引用计数-1，此时引用计数=0，内存被释放
 [person autorelease];  // 将对象加入自动释放池
 Person *person = [[[Person alloc] init] autorelease]; // 也可以在创建对象时将其加入自动释放池
```
按道理来说创建一个对象，然后`release`后该对象引用计数为零，但是实际情况中并不会出现这种现象，`release`后再输出其引用计数还是为1，在我的理解中有两种可能：
 1. 该对象在引用计数为1的时候进行`release`后，对象已经被释放，此时再调用`retainCount`毫无意义，因为该对象已经不存在了，为了防止某些错误保护这个`retainCount`方法所以编译器不会报错，但是输出值为释放前的值；
 2. 编译器为我们做了各种优化，也许是记录`retainCount`为零消耗过大或者没有意义。

![重写了`dealloc`方便查看对象是否被释放](http://upload-images.jianshu.io/upload_images/2590408-c23186f1a9b761f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![输出其`retainCount`然后释放](http://upload-images.jianshu.io/upload_images/2590408-85f2e1aadd64812e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![可以看到并不会出现引用计数为零的情况，但是该对象确实被释放了](http://upload-images.jianshu.io/upload_images/2590408-386dbf919566a2b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> **小知识**
> ***
> **指针错误**：访问了一块坏的内存（已经被回收的，不可用的内存）。
**僵尸对象**：所占内存已经被回收的对象，僵尸对象不能再被使用。（打开僵尸对象检测）
**空指针**：没有指向任何东西的指针（存储的东西是0,null，nil），给空指针发送消息不会报错。
**注意**：不能使用[p retaion]让僵尸对象起死复生。

在MRC管理时代有一个黄金法则：
 1. 谁创建谁负责。如果你通过alloc,new,copy来创建了一个对象，那么你就必须调用release或者autorelease方法；
 2. 谁retain，谁release。只要你调用了retain，无论这个对象时如何生成的，你都要调用release；

#ARC
***
##原理
前段编译器会为“拥有的”每一个对象加入相应的`release`语句，如果对象的所有权修饰符是`__strong`，那么它就是被拥有的。如果再某个方法内创建了一个对象，前端编译器会在方法末尾自动插入`release`语句已销毁它。而类拥有的对象（实例变量/属性）会在`dealloc`方法内被释放。
![编译器所为](http://upload-images.jianshu.io/upload_images/2590408-087aa0f74748d421.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 编译器为我们做的，我们可以手动完成达到优化
比如：
`__autoreleasing`在ARC中主要用在参数传递返回值（out-parameters）和引用传递参数（pass-by-reference）的情况下，有这种指针（`NSError **`）的函数参数如果不加修饰符，编译器会默认将他们认定为`__autoreleasing`类型。  
比如常用的`NSError`的使用：

```
NSError *__autoreleasing error;
if (![data writeToFile:filename options:NSDataWritingAtomic error:&error])
{
    NSLog(@"Error: %@", error);
}
```
如果你把`error`定义为了`strong`型，编译器会隐式地做如下事情，保证最终传入函数的参数依然是个`__autoreleasing`类型的引用。

```
NSError *error;
NSError *__autoreleasing tempError = error; // 编译器添加
if (![data writeToFile:filename options:NSDataWritingAtomic error:&tempError])
{
    error = tempError; // 编译器添加
    NSLog(@"Error :%@", error);
}
```
所以为了提高效率，避免这种情况，我们一般在定义`error`的时候将其老老实实地声明为`__autoreleasing`类型。

***
#循环引用
***
平常我们容易造成循环引用的三种情况：
1. **`NSTimer`**
    先看`NSTimer`使用的代码：
    
```
_timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(runTimer) userInfo:nil repeats:YES];
```
其中`_timer`是实例变量被`self`保留，`_timer`的target是`self`，`self`被`_timer`保留，引发循环引用。
![循环引用](http://upload-images.jianshu.io/upload_images/2590408-0e9a0b344f927e34.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
解除方法就是使target中的对象不是`viewController`从而断开引用，iOS10之前我们可以写个类别重新封装target来实现，iOS10之后系统给了新方法：

```
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block;
```
 * 不再需要target，而是传入一个block，在block里面进行循环调用方法
 * 关于block怎么解决循环引用请看下面


2. **block**

> 简介
block和其他语言的闭包或lambda表达式是一回事,block的使用很像函数指针，不过与函数最大的不同是：block可以访问函数以外、词法作用域以内的外部变量的值。换句话说，block不仅实现函数的功能，还能携带函数的执行环境。

**block基本语法**

```
// 声明一个block变量
long (^sum) (int, int) = nil;
// sum是个block变量，该block类型有两个int型参数，返回类型是long。
// 定义block并赋给变量sum
sum = ^ long (int a, int b) {
  return a + b;
};
// 调用block：
long s = sum(1, 2);
```
定义一个实例函数，该函数返回block：

```
- (long (^)(int, int)) sumBlock {
    int base = 100;
    return [[ ^ long (int a, int b) {
      return base + a + b;
    } copy] autorelease];
  }
// 调用block
[self sumBlock](1,2);
```
根据在内存中的位置将block分为三种类型：
    * `NSGlobalBlock`: 类似函数，位于text段；
    * `NSStackBlock`: 位于栈内存，函数返回后block将无效；
    * `NSMallocBlock`: 位于堆内存。

block其实包含两个部分内容：
1. block执行的代码，这是在编译的时候已经生成好的；
2. 一个包含block执行时需要的所有外部变量值的数据结构。 block将使用到的、作用域附近到的变量的值建立一份快照拷贝到栈上。
![block的数据结构](http://upload-images.jianshu.io/upload_images/2590408-cc743eb4f46bc26b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于 block 外的变量引用，block 默认是将其复制到其数据结构中来实现访问的：
![传入外部变量](http://upload-images.jianshu.io/upload_images/2590408-2b3a94efc4a64b1b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对于用 `__block` 修饰的外部变量引用，block 是复制其引用地址来实现访问的：
![用__block修饰](http://upload-images.jianshu.io/upload_images/2590408-6d48edcdc97d0af2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

初步了解了block后看看它怎么构成循环引用并怎么解决的吧

```
typedef void(^block)();

@property (copy, nonatomic) block myBlock;
@property (copy, nonatomic) NSString *blockString;

- (void)testBlock {
    self.myBlock = ^() {
        //其实注释中的代码，同样会造成循环引用
        NSString *localString = self.blockString;
          //NSString *localString = _blockString;
          //[self doSomething];
    };
}
```
看了前面关于block的一些介绍应该容易看出来，当我们往block中传入数据时是保存在了block的堆中，如上述代码中引用了`self`相当于对`self`进行了一次`retain`，而`self`本身持有`block`于是造成了循环引用，同时在block中`release``self`没有用，因为在block中操作作用范围仅仅来block的函数栈，影响不到堆中的`self`，解决方法如下：

```
__weak typeof(self) weakSelf = self;
self.myBlock = ^(){
    __strong typeof(weakSelf) = strongSelf;
    NSString *localString = strongSelf;
}
```
其中传入一个若引用就不会造成循环引用，然后在block的函数栈中用一个强指针来接受传进来的弱指针，防止弱指针被提前释放产生野指针。
> 参考文章：
> Cooper -- [正确使用Block避免Cycle Retain和Crash](http://tanqisen.github.io/blog/2013/04/19/gcd-block-cycle-retain/)
> 唐巧 -- [谈Objective-C block的实现](http://blog.devtang.com/2013/07/28/a-look-inside-blocks/)
> Dev Talking -- [Objective-C中的Block](http://www.cocoachina.com/ios/20150109/10891.html)

3. **delegate**
我们对代理的写法一般都是：

```
@property (nonatomic, weak) id<TestDelegate> delegate;
```
如果使用`strong`的话很明显会造成循环引用（`delegate`调用`self`的一些东西），今天被面试官问道如果使用`delegate`出现了循环引用怎么解决，我说用weak，他说换一个，然后就懵住了，只回答了思路，找到互相引用的对象(可以用Instruments)然后断开引用。

# Autorelease
- 简介  
很好理解，字面意思上看就是自动释放，我们可以通过使用autorelease让编译器帮我们在某个时刻自动释放内存。在MRC时我们使用NSAutorelease类来使用自动释放机制，代码如下：

```
NSAutoreleasePool pool = [[NSAutoreleasePool alloc] init];
// Code benefitting from a local autorelease pool.
[pool release];
```
也可以直接使用`[obj autorelease]`。
现在基本上都是ARC环境，这个时候我们使用的是`autoreleasepool`（自动释放池），比如常见的：

```
//iOS program
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
//Command line program
int main(int argc, const char *argv[]) {
    @autoreleasepool {
        //...
    }
    return 0;
}
```
它的作用是把我们在`{}`中申请的对象在事件处理完时自动释放掉，其中的原理推荐阅读**Qi Tang**的[iOS 中的 Autorelease Pool](http://itangqi.me/2016/04/15/autorelease-pool-in-ios/)。
前面说到的事件处理完时其实就是一次`runloop`结束时。
![runloop和autorelease](http://upload-images.jianshu.io/upload_images/2590408-efe8d04b74c7faf3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 程序运行 -> 开启事件循环 -> 发生触摸事件 -> 创建自动释放池 -> 处理触摸事件 -> 事件对象加入自动释放池 -> 一次事件循环结束, 销毁自动释放池

- 什么时候用`@autoreleasepool`
    1. 写基于命令行的的程序时，就是没有UI框架，如AppKit等Cocoa框架时。
    * 写循环，循环里面包含了大量临时创建的对象。（本文的例子）
    * 创建了新的线程。（非Cocoa程序创建线程时才需要）
    * 长时间在后台运行的任务。

- 利用`@autoreleasepool`优化循环
当我们一个循环内创建了很多临时对象时，可以通过使用`@autoreleasepool`在每次循环结束时释放内存：

```
//来自Apple文档，见参考
NSArray *urls = <# An array of file URLs #>;
for (NSURL *url in urls) {
    @autoreleasepool {
        NSError *error;
        NSString *fileContents = [NSString stringWithContentsOfURL:url
                                        encoding:NSUTF8StringEncoding error:&error];
        /* Process the string, creating and autoreleasing more objects. */
    }
}
```
> 参考文章：
> sunnyxx —— [黑幕背后的Autorelease](http://blog.sunnyxx.com/2014/10/15/behind-autorelease/)
> Jerry4me —— [iOS中autorelease的那些事儿](http://www.jianshu.com/p/5559bc15490d)
> tutuge —— [@autoreleasepool-内存的分配与释放](http://tutuge.me/2015/03/17/what-is-autoreleasepool/)


# 修饰词





