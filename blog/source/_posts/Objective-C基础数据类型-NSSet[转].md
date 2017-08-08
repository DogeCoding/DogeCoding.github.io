
title: Objective-C基础数据类型-NSSet[转]
tags: iOS
---

> [转自*GISerYang*](http://www.cnblogs.com/GISerYang/p/3340937.html)  

***集合（NSSet）***和***数组（NSArray）***有相似之处，都是存储不同的对象的地址；不过NSArray是有序的集合，NSSet是无序的集合。  

集合是一种哈希表，运用散列算法，查找集合中的元素比数组速度更快，但是它没有顺序。

``` //
NSSet * set = [[NSSet alloc] initWithObjects:@"one",@"two",@"three",@"four", nil];  
[set count]; //返回集合中对象的个数
```

判断集合中是否拥有某个元素  

``` //
//判断集合中是否拥有@“two”  
BOOL ret = [set containsObject:@"two"];
``` 

判断两个集合是否相等  

``` //
NSSet * set2 = [[NSSet alloc] initWithObjects:@"one",@"two",@"three",@"four", nil];   
BOOL ret = [set isEqualToSet:set2];
``` 

判断set是否是set2的子集合  

``` //
NSSet * set2 = [[NSSet alloc] initWithObjects:@"one",@"two",@"three",@"four",@"five", nil];  
BOOL ret = [set isSubsetOfSet:set2];
``` 

 

集合也可以用枚举器来遍历  

``` //
NSEnumerator * enumerator = [set objectEnumerator];  
NSString *str;  
while (str = [enumerator nextObject]) {
    // logic
}
``` 

通过数组来初始化集合（数组转换为集合）  

``` //
NSArray * array = [[NSArray alloc] initWithObjects:@"one",@"two",@"three",@"four", nil];  
NSSet * set = [[NSSet alloc] initWithArray:array];
``` 

集合转换为数组  

``` //
NSArray * array2 = [set allObjects];
``` 
 

#可变集合NSMutableSet
可变集合NSMutableSet

``` //objective-c
NSMutableSet * set = [[NSMutableSet alloc] init];  
[set addObject:@"one"];  
[set addObject:@"two"];  
[set addObject:@"two"]; //如果添加的元素有重复，实际只保留一个
``` 

删除元素

``` //objective-c
[set removeObject:@"two"];  
[set removeAllObjects];
```

将set2中的元素添加到set中来，如果有重复，只保留一个

``` // objective-c
NSSet * set2 = [[NSSet alloc] initWithObjects:@"two",@"three",@"four", nil];  
[set unionSet:set2];
```

删除set中与set2相同的元素

``` // objective-c
[set minusSet:set2];
``` 

 

# 指数集合（索引集合）NSIndexSet

```  //
NSIndexSet * indexSet = [[NSIndexSet alloc] initWithIndexesInRange:NSMakeRange(1, 3)]; //集合中的数字是123
```  

根据集合提取数组中指定位置的元素
 
``` //
NSArray * array = [[NSArray alloc]   initWithObjects:@"one",@"two",@"three",@"four", nil];  
NSArray * newArray = [array objectsAtIndexes:indexSet]; //返回@"two",@"three",@"four"
```

# 可变指数集合NSMutableIndexSet

``` //
NSMutableIndexSet *indexSet = [[NSMutableIndexSet alloc] init];  
[indexSet addIndex:0];  
[indexSet addIndex:3];  
[indexSet addIndex:5];  
//通过集合获取数组中指定的元素  
NSArray *array = [[NSArray alloc] initWithObjects:@"one",@"two",@"three",@"four",@"five",@"six", nil];  
NSArray *newArray = [array objectsAtIndexes:indexSet]; //返回@"one"，@"four"，@"six"
```



