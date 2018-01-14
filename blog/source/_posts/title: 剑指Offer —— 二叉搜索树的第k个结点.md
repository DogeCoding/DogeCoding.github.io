title: 剑指Offer —— 二叉搜索树的第k个结点
tags: 剑指Offer
categories: Algorithm
date: 2017/8/4 9:56
---

> 题目来源:**[牛客网](https://www.nowcoder.com/practice/ef068f602dde4d28aab2b210e859150a?tpId=13&tqId=11215&tPage=4&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)**  

***    
## 题目描述
给定一颗二叉搜索树，请找出其中的第k大的结点。例如， 5 / \ 3 7 /\ /\ 2 4 6 8 中，按结点数值大小顺序第三个结点的值为4。

  

## 解题思路

1. 递归
2. 迭代
### way1
中序遍历，找到第k个结点。完全性的模拟左根右，注意返回结果的处理。

``` //
TreeNode* KthNode(TreeNode* pRoot, int k) {
	if (pRoot) {
		TreeNode *node = KthNode(pRoot->left, k);
		if (node) return node;
		index++;
		if (index == k) return pRoot;
		node = KthNode(pRoot->right, k);
		if (node) return node;
	}
	return NULL;
}
```   

