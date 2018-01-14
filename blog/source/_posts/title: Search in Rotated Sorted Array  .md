title: Search in Rotated Sorted Array  
tags: LeetCode
categories: Algorithm
date: 2017/7/23 23:56
---

> 题目来源LeetCode: **[LeetCode OJ](https://leetcode.com/problems/search-in-rotated-sorted-array/)**  

***    
> 题目描述: Suppose a sorted array is rotated at some pivot unknown to you beforehand.  
(i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2).  
You are given a target value to search. If found in the array return its index, otherwise return -1.  
You may assume no duplicate exists in the array.  

## 题目解释  
一个排好序的数组，不知道以哪个点为中心旋转了(部分有序)，你的任务是查找给定的数是否存在数组中。在的话返回下标，不在的话返回－1. 
  
## 分析  
查找首先想到O(log(n))的二分查找，但是二分查找的前提是有序数组。题目里一个有序数组旋转后变成了部分有序。通过比较两端大小找到增序部分。  
>eg: 4 5 6 0 1 2 3  
first = 4, mid = 0, last = 3，  
通过比较first,mid,last找到增序部分。
在这个例子中为1 2 3，  
然后判断target是否在这个增序子序列中，  
如果在则直接用二分查找，  
不在则在另一部分(例子中为4 5 6 0)中继续分解。

 

### Code
``` //
int search(const vector<int>& nums, int target)
{
	int first = 0;
	int last = nums.size();
	while(first != last)
	{
		const int mid = (first+last)>>1;	// 使用位运算加速
		if(nums[mid] == target)
			return mid;
		if(nums[first] <= nums[mid])	// 找到增序子序列
			if(nums[first]<=target && target<nums[mid])	// 找到target在哪个部分
				last = mid;
			else
				first = mid+1;
		else
			if(nums[mid]<target && target<=nums[last-1])
				first = mid+1;
			else
				last = mid;
	}
	return -1;
}
```  
# 拓展
当数组为无序时的二分查找  
## 分析
一种是先排序再二分，一种是结合快排思想，每次选择一个关键字，比他大的放右边，比他小的放左边，然后再比较他和需要查找的数的关系，再选择区间进行迭代。如果需要返回查找数的下标，则添加一个纪录下标的数组，这样排好序后也能知道当前数在原始数组中的位置。  
>初始数组3 1 2 5 4 7 0 6  
mid = key = 3 进行一次快排填坑  
得到数组0 1 2 3 4 7 5 6  
比较mid与target  
如果target>mid则迭代mid后半部分  
如果target<mid则迭代mid前半部分  
直到找到target  

### Code
``` //
int BinarySearch(vector<int>& nums, int target)
{
	int num = nums.size();
	int index[num];		// index数组纪录下标 以便能找到在数组的初始位置
	for(int i = 0; i < num; i++)	// 初始化index数组
		index[i] = i;
	int l, r, m, sl, sr, mIndex;
	l = 0, r = num-1;
	while(l<=r)		// 开始迭代
	{
		mIndex = index[l], m = nums[l];
		sl = l, sr = r;
		while(sl<sr)	// 快排思想，左右填坑，并用index记录位置
		{
			while(sl<sr && m<nums[sr])
				sr--;
			nums[sl] = nums[sr];
			index[sl] = index[sr];
			while(sl<sr && m>nums[sl])
				sl++;
			nums[sr] = nums[sl];
			index[sr] = index[sl];
		}
		nums[sl] = m;
		index[sl] = mIndex;
		if(m == target)
			return mIndex;
		if(target > m)		// 判断target在哪个区间
			l = sl+1;
		else
			r = sl-1;
	}
	return -1;
}
```  

## 相关题目
[Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/)  

## 相关题目分析
因为允许出现重复数字，但是数组还是部分有序的，所以跳过重复数字即可  

### Code
``` //
int search(const vector<int>& nums, int target)
{
	int first = 0;
	int last = nums.size();
	while(first != last)
	{
			const int mid = (first+last)>>1;
		if(nums[mid] == target)
			return mid;
		if(nums[first] < nums[mid])
			if(nums[first]<=target && target<nums[mid])
				last = mid;
			else
				first = mid+1;
		else if(nums[first] > nums[mid])
		{
			if(nums[mid]<target && target<=nums[last-1])
				first = mid+1;
			else
				last = mid;
		}
		else	// 特判相等时跳过
			first++;
	}
	return -1;
}
```  

