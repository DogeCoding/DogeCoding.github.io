title: 踩坑实录：Unrecognized Selector -replacementObjectForKeyedArchiver:
tags: Bug
categories: iOS
date: 2017/8/29 9:54
---

当在swift里面对一个类实现`NSCoding`让其能序列/反序列化时，如：

```
class SomeClass: NSCoding {
    var someInstance: String

    init(someInstance: String = "") {
    }

    func encode(with aCoder: NSCoder) {
        aCoder.encode(someInstance, forKey: "SomeInstance")
    }

    required init?(coder aDecoder: NSCoder) {
        if let someInstance = aDecoder.decodeObject(forKey: "SomeInstance") as? String {
            self.someInstance = someInstance
        } else {
            self.someInstance = ""
        }
    }
```

会扔出这样的异常：

```
2017-08-29 10:02:10.874 SwiftLearnDemo[29161:9488531] *** NSForwarding: warning: object 0x608000265f40 of class 'SwiftLearnDemo.SomeClass' does not implement methodSignatureForSelector: -- trouble ahead
Unrecognized selector -[SwiftLearnDemo.SomeClass replacementObjectForKeyedArchiver:]
```

原因是`SomeClass`没有继承自`NSObject`


