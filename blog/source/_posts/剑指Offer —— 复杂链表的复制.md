# 剑指Offer —— 复杂链表的复制
> 题目来源牛客网：[复杂链表的复制](https://www.nowcoder.com/practice/f836b2c43afc4b35ad6adc41ec941dba?tpId=13&tqId=11178&tPage=2&rp=2&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)

## 题目描述
输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

## 数据结构

```
/*
struct RandomListNode {
    int label;
    struct RandomListNode *next, *random;
    RandomListNode(int x) :
            label(x), next(NULL), random(NULL) {
    }
};
*/
class Solution {
public:
    RandomListNode* Clone(RandomListNode* pHead) {
    
    }
};
```

## 解题思路
### 一、递归思想：把大问题转化若干子问题
此题转化为一个头结点和除去头结点剩余部分，剩余部分操作和原问题一致

```
RandomListNode* Clone(RandomListNode* pHead)
    {
        if(pHead==NULL)
            return NULL;
         
        //开辟一个新节点
        RandomListNode* pClonedHead=new RandomListNode(pHead->label);
        pClonedHead->next = pHead->next;
        pClonedHead->random = pHead->random;
         
        //递归其他节点
        pClonedHead->next=Clone(pHead->next);
         
        return pClonedHead;
    }
```
### 二、
* 复制每个节点，如：复制节点A得到A1，将A1插入节点A后面
* 遍历链表，A1->random = A->random->next;
* 将链表拆分成原链表和复制后的链表

```
RandomListNode* Clone(RandomListNode* pHead)
{
    if(!pHead) return NULL;
    RandomListNode *currNode = pHead;
    while(currNode){
        RandomListNode *node = new RandomListNode(currNode->label);
        node->next = currNode->next;
        currNode->next = node;
        currNode = node->next;
    }
    currNode = pHead;
    while(currNode){
        RandomListNode *node = currNode->next;
        if(currNode->random){                
            node->random = currNode->random->next;
        }
        currNode = node->next;
    }
    //拆分
    RandomListNode *pCloneHead = pHead->next;
    RandomListNode *tmp;
    currNode = pHead;
    while(currNode->next){
        tmp = currNode->next;
        currNode->next =tmp->next;
         currNode = tmp;
    }
    return pCloneHead;
 }
```
### 三、哈希表法
时间空间复杂度都是O(n)

```
RandomListNode* Clone(RandomListNode* pHead)
{
    if(pHead==NULL)
        return NULL;
         
    //定义一个哈希表
    unordered_multimap<RandomListNode*,RandomListNode*> table;
         
    // 开辟一个头结点
    RandomListNode* pClonedHead=new RandomListNode(pHead->label);
    pClonedHead->next=NULL;
    pClonedHead->random=NULL;
         
    // 将头结点放入map中
    table.insert(make_pair(pHead,pClonedHead));
         
    //设置操作指针
    RandomListNode* pNode=pHead->next;
    RandomListNode* pClonedNode=pClonedHead;
         
    // 第一遍先将简单链表复制一下
    while(pNode!=NULL)
    {
        // 不断开辟pNode的拷贝结点
        RandomListNode* pClonedTail=new RandomListNode(pNode->label);
        pClonedTail->next=NULL;
        pClonedTail->random=NULL;
             
        //连接新节点，更新当前节点
        pClonedNode->next=pClonedTail;
        pClonedNode=pClonedTail;
             
        //将对应关系  插入到哈希表中
        table.insert(make_pair(pNode,pClonedTail));
             
        //向后移动操作节点
        pNode=pNode->next;
    }
         
    //需从头开始设置random节点，设置操作指针
    pNode=pHead;
    pClonedNode=pClonedHead;
         
    // 根据map中保存的数据，找到对应的节点
    while(pNode!=NULL)
    {
             
        if(pNode->random!=NULL)
        {
            //找到对应节点，更新复制链表
            pClonedNode->random=table.find(pNode->random)->second;
        }
             
        //向后移动操作节点
        pNode=pNode->next;
        pClonedNode=pClonedNode->next;
    }
         
    return pClonedHead;
         
}
```

