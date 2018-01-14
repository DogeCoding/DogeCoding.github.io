title: [Unix Shell Command] rm
tags: Unix
categories: Shell
date: 2017/9/1 10:59
---

## 作用
remove directory entries. 在Wiki上的解釋是，用於刪除文件系統中的文件、目錄、設備文件、符號鏈接等對象的引用。
## 使用格式

```
rm [-dfiPRrvW] file ...
```

## 命令参数
**-d/--directory**, 直接把欲删除的目录的硬连接数据删成0，删除该目录
**-f/--force**, 强制删除文件或目录
**-i/--interactive**, 删除既有文件或目录之前先询问用户
**-r, -R/--recursive**,  指示rm将参数中列出的全部目录和子目录均递归地删除
**-v/--verbose**,  显示指令执行过程

> 參考:
> [每天一个linux命令（5）：rm 命令](http://www.cnblogs.com/peida/archive/2012/10/26/2740521.html)


