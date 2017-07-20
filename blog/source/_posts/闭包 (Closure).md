# 闭包 (Closure)
## 尾随闭包
尾随闭包是一个书写在函数括号之后的闭包表达式，函数支持将其作为最后一个参数调用。在使用尾随闭包时，你不用写出它的参数标签。
如果闭包表达式是函数或方法的唯一参数，则当你使用尾随闭包时，你甚至可以把 () 省略掉。

```
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // 函数体部分
}
 
// 以下是不使用尾随闭包进行函数调用
someFunctionThatTakesAClosure(closure: {
    // 闭包主体部分
})
 
// 以下是使用尾随闭包进行函数调用
someFunctionThatTakesAClosure {
    // 闭包主体部分
}
```

## @autoclosure
`@autoclosure`做的事情就是把一句表达式自动地封装成一个闭包。
一个自动闭包+逃逸闭包的例子：

```
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]

var customerProviders: [() -> String] = []
func collectCustomerProviders(_ customerProvider: @autoclosure @escaping () -> String) {
    customerProviders.append(customerProvider)
}
collectCustomerProviders(customersInLine.remove(at: 0))
collectCustomerProviders(customersInLine.remove(at: 0))

print("Collected \(customerProviders.count) closures.")

for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
```
`??`的例子关于巧妙使用自动闭包代替参数

```
var level: Int?
var startLevel = 1
var currentLevel = level ?? startLevel

print(currentLevel)
```

`??`的两种版本定义

```
public func ??<T>(optional: T?, defaultValue: @autoclosure () throws -> T) rethrows -> T
public func ??<T>(optional: T?, defaultValue: @autoclosure () throws -> T?) rethrows -> T?
```

`??`的猜测实现

```
func ??<T>(optional: T?, defaultValue: @autoclosure () -> T) -> T{
    switch optional {
    case .some(let value):
        value
    case .none:
        return defaultValue()
    }
}
```
如果`defaultValue`是通过一系列复杂计算得到的，通过使用`@autoclosure`可以避免浪费的可能性——因为如果`optional`不是`nil`的话不会用到默认值
> `@autoclosure`不支持带有输入参数的写法，只有形如`() -> T`的参数才能使用这个特性简化，另外调用者容易忽视`@autoclosure`的特性，在容易产生误解或者歧义的地方还是使用完整的闭包

## @escaping


