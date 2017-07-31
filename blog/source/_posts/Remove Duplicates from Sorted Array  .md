title: Remove Duplicates from Sorted Array  
tags: Algorithm
---
 
> 题目来源LeetCode: **[LeetCode OJ](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)**  

***    
## 题目描述
Given a sorted array, remove the duplicates in place such that each element appear only once and return the new length.  
Do not allocate extra space for another array, you must do this in place with constant memory.
For example,  
Given input array nums = [1,1,2],  
Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively. It doesn't matter what you leave beyond the new length.  
 

## 题目解释  
给出一个sorted array(意思是指已经排好序了?)，处理后数组里每一个元素只能出现一次，返回处理后的数组长度。  
不能使用额外的数组空间，只能用已经给出的确定的内存空间。  
## 分析  
因为不太懂sorted array具体指的什么，第一次做的时候以为数组是随机的，相同元素出现的位置是随机的，然后题目也没给出limit time，随手就写了一个O(n^3)  

```
for(int i = 0; i < num; i++){  
	for(int j = i+1; j < num; j++){  
		if(array[i] == array[j]){  
 			for(int k = j; k < num-1; k++)  
 				array[k] = array[k+1];  
 			num--;  
 			j--;  
 		}  
 	}  
}
```  
自然是T了。然后就把sorted array当做已经排好序的数组，那就容易多了，算法也都是O(1)，一看代码就明白，水题，直接上代码。  

### way1
```
if (nums.empty()) return 0;  
   int index = 0;  
   for (int i = 1; i < nums.size(); i++) {  
      if (nums[index] != nums[i])  
         nums[++index] = nums[i];  
 	}  
return index + 1;
```  

### way2 STL
```
return distance(nums.begin(), unique(nums.begin(), nums.end()));
```  

 

- std::distance  
template<class InputIterator>  
  typename iterator_traits<InputIterator>::difference_type  
    distance (InputIterator first, InputIterator last);  
Return distance between iterators  
Calculates the number of elements between first and last.  
[c++ reference](http://www.cplusplus.com/reference/iterator/distance/)  

- std::unique  
equality (1)  
template <class ForwardIterator>  
  ForwardIterator unique (ForwardIterator first, ForwardIterator last);  
predicate (2)  	
template <class ForwardIterator, class BinaryPredicate>
  ForwardIterator unique (ForwardIterator first, ForwardIterator last,
                          BinaryPredicate pred);  
Remove consecutive duplicates in range  
Removes all but the first element from every consecutive group of equivalent elements in the range [first,last).  
[c++ reference](http://www.cplusplus.com/reference/algorithm/unique/?kw=unique)  

## 相关题目
[RemoveDuplicatesfromSortedArrayII](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/)

