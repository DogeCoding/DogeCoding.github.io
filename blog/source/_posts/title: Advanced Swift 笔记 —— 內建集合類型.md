title: Advanced Swift 笔记 —— 內建集合類型
tags: Reading Notes
categories: iOS
date: 2017/8/30 15:47
---

## 數組(Array)
盡量不要使用下標索引，如果下標越界會直接導致crash（在並發情況下尤其需要考慮）。

- 迭代全部

```
for x in array
```
- 迭代除了第一個元素以外的其餘部分

```
for x in array.dropFirst()
```
- 迭代除了最後5個元素以外的數組

```
for x in array.dropLast(5)
```
- 列舉數組中的元素和對應下標

```
for (index, value) in array.enumerated()
```
- 尋找一個指定元素的位置

```
_ = array.index {
    someMatchingLogic($0)
}
```
- 對數組所有元素變形

```
_ = array.map {
    someTransformation($0)
}
```
- 篩選符合某個標準的元素

```
_ = array.filter {
    someCriteria($0)
}
```


