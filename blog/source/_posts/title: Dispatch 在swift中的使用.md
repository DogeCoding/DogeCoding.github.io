title: Dispatch 在swift中的使用
tags: Swift
categories: iOS
date: 2017/8/3 9:49
---

## 创建队列


当group里所有事件都完成GCD API有两种方式发送通知，第一种是dispatch_group_wait，会阻塞当前进程，等所有任务都完成或等待超时。第二种方法是使用dispatch_group_notify，异步执行闭包，不会阻塞。

``` //
func downloadPhotosWithCompletion(completion: BatchPhotoDownloadingCompletionClosure?) {
     dispatch_async(GlobalUserInitiatedQueue) { // 因为dispatch_group_wait会租塞当前进程，所以要使用dispatch_async将整个方法要放到后台队列才能够保证主线程不被阻塞
          var storedError: NSError!
          var downloadGroup = dispatch_group_create() // 创建一个dispatch group

          for address in [OverlyAttachedGirlfriendURLString,
               SuccessKidURLString,
               LotsOfFacesURLString]
          {
               let url = NSURL(string: address)
               dispatch_group_enter(downloadGroup) // dispatch_group_enter是通知dispatch group任务开始了，dispatch_group_enter和dispatch_group_leave是成对调用，不然程序就崩溃了。
               let photo = DownloadPhoto(url: url!) {
                    image, error in
                    if let error = error {
                         storedError = error
                    }
                    dispatch_group_leave(downloadGroup) // 保持和dispatch_group_enter配对。通知任务已经完成
               }
               PhotoManager.sharedManager.addPhoto(photo)
          }

          dispatch_group_wait(downloadGroup, DISPATCH_TIME_FOREVER) // dispatch_group_wait等待所有任务都完成直到超时。如果任务完成前就超时了，函数会返回一个非零值，可以通过返回值判断是否超时。也可以用DISPATCH_TIME_FOREVER表示一直等。
          dispatch_async(GlobalMainQueue) { // 这里可以保证所有图片任务都完成，然后在main queue里加入完成后要处理的闭包，会在main queue里执行。
               if let completion = completion { // 执行闭包内容
                    completion(error: storedError)
               }
          }
     }
}
```

## Serial Queues

``` //
private let serialQueue = DispatchQueue(label: “serialQueue”)
private var dictionary: [String: Any] = [:]

public func set(_ value: Any, forKey key: String) {
    serialQueue.sync {
        dictionary[key] = value
    }
}

public func object(forKey key: String) -> Any? {
    var result: Any?
       
    serialQueue.sync {
        result = storage[key]
    }
    // returns after serialQueue is finished operation
    // beacuse serialQueue is run synchronously
    return result
}
```

## Concurrent Queues

``` //
private let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)
private var dictionary: [String: Any] = [:]
    
public func set(_ value: Any?, forKey key: String) {
    concurrentQueue.async {
        self.storage[key] = value
    }
}

public func object(forKey key: String) -> Any? {
    var result: Any?

    concurrentQueue.sync {
        result = storage[key]
    }

    // returns after concurrentQueue is finished operation
    // beacuse concurrentQueue is run synchronously
    return result
}
```

## Concurrent Queue with Barrier

``` //
private let concurrentQueue = DispatchQueue(label: "concurrentQueue", attributes: .concurrent)
private var dictionary: [String: Any] = [:]
    
public func set(_ value: Any?, forKey key: String) {
    // .barrier flag ensures that within the queue all reading is done
    // before the below writing is performed and
    // pending readings start after below writing is performed
    concurrentQueue.async(flags: .barrier) {
        self.storage[key] = value
    }
}

public func object(forKey key: String) -> Any? {
    var result: Any?

    concurrentQueue.sync {
        result = storage[key]
    }

    // returns after concurrentQueue is finished operation
    // beacuse concurrentQueue is run synchronously
    return result
}
```

### 延时

```
typealias Task = (_ cancel: Bool) -> Void

    func delay(time: TimeInterval, task: @escaping ()->()) -> Task? {
        func dispatch_later(block: @escaping ()->()) {
            DispatchQueue.main.asyncAfter(deadline: DispatchTime.now()+time, execute: block)
        }

        var closure: (()->())? = task
        var result: Task?

        let delayedClosure: Task = { cancel in
            if let internalClosure = closure {
                if cancel == false {
                    DispatchQueue.main.async {
                        internalClosure()
                    }
                }
            }
            closure = nil
            result = nil
        }

        result = delayedClosure
        dispatch_later {
            if let delayedClosure = result {
                delayedClosure(false)
            }
        }
        
        return result
    }
    
    func cancel(task: Task?) {
        task?(true)
    }
```

### 网络场景异步开启下载、取消下载，开始下载前保证没有遗留下载任务

```
fileprivate let barrierQueue = DispatchQueue(label: "com.meitu.meipu.videoCache.barrierQueue")

func downloader(url: URL) {
        barrierQueue.sync { [weak self] in
            // add downloader task
        }
}

func cancel() {
    barrierQueue.async(group: nil, qos: .default, flags: .barrier) {
        // cacle task    
    }
}
```

> 参考:
> [GCD 在 Swift 3 中的玩儿法](https://swiftcafe.io/2016/10/16/swift-gcd/)
> [GCD 使用指南](http://swift.gg/2016/05/05/the-gcd-handbook/)
> [Swift 3 中的 GCD 与 Dispatch Queue](http://swift.gg/2016/11/30/grand-central-dispatch/)
> [Swift 3學習指南：重新認識GCD應用](https://www.appcoda.com.tw/grand-central-dispatch/)
> [GCD精讲（Swift 3）](http://blog.csdn.net/hello_hwc/article/details/54293280)







