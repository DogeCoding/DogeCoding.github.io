title: 剑指Offer —— 最小的k个数
tags: 剑指Offer
categories: Algorithm
date: 2017/8/1 10:20
---

***    
## 题目描述
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4。(很典型的题目了)
 
## 解题思路
1. 插入或者冒泡排序，优化点在记录已排好的个数(O(n*k))
2. 最小堆(O(nlogk))
3. 快排思想(O(n))

### way1
不上代码了。

### way2
上个建堆的算法，通用、解耦、易测试😂

``` //
void HeapAdjust(int H[],int s, int length)  
{  
    int tmp  = H[s];  
    int child = 2*s+1; //左孩子结点的位置。(i+1 为当前调整结点的右孩子结点的位置)  
    while (child < length) {  
        if(child+1 <length && H[child]<H[child+1]) { // 如果右孩子大于左孩子(找到比当前待调整结点大的孩子结点)  
            ++child ;  
        }  
        if(H[s]<H[child]) {  // 如果较大的子结点大于父结点  
            H[s] = H[child]; // 那么把较大的子结点往上移动，替换它的父结点  
            s = child;       // 重新设置s ,即待调整的下一个结点的位置  
            child = 2*s+1;  
        }  else {            // 如果当前待调整结点大于它的左右孩子，则不需要调整，直接退出  
             break;  
        }  
        H[s] = tmp;         // 当前待调整的结点放到比其大的孩子结点位置上  
    }  
    print(H,length);  
}  

/** 
 * 初始堆进行调整 
 * 将H[0..length-1]建成堆 
 * 调整完之后第一个元素是序列的最小的元素 
 */  
void BuildingHeap(int H[], int length)  
{   
    //最后一个有孩子的节点的位置 i=  (length -1) / 2  
    for (int i = (length -1) / 2 ; i >= 0; i--)  
    {
    	cout << "i: " << i << endl;
    	HeapAdjust(H,i,length);
    }
}  
/** 
 * 堆排序算法 
 */  
void HeapSort(int H[],int length)  
{  
    //初始堆  
    BuildingHeap(H, length);  
    //从最后一个元素开始对序列进行调整  
    for (int i = length - 1; i > 0; --i)  
    {  
        //交换堆顶元素H[0]和堆中最后一个元素  
        int temp = H[i]; H[i] = H[0]; H[0] = temp;  
        //每次交换堆顶元素和堆中最后一个元素之后，都要对堆进行调整  
        HeapAdjust(H,0,i);  
  }  
} 
```

### way3

``` //
int Partition(vector<int> &input, int begin, int end)
{
	int first = begin;
	int last = end;
	int pivot = input[first];
	while (first < last)
	{
		while (first < last && input[last] >= pivot) last--;
		input[first] = input[last];
		while (first < last && input[first] <= pivot) first++;
		input[last] = input[first];
	}
	input[first] = pivot;
	return first;
}
vector<int> GetLeastNumbers_Solution(vector<int> &input, int k)
{
	int len=input.size();
	vector<int> res;
	if(len==0||k>len||k<=0) return res;
	if(len==k) return input;
	int start=0;
	int end=len-1;
	int index=Partition(input,start,end);
	while(index!=(k-1))
	{
		if(index>k-1)
		{
			end=index-1;
			index=Partition(input,start,end);
		}
		else
		{
			start=index+1;
			index=Partition(input,start,end);
		}
	}
	for (int i = 0; i < k; i++)
		res.push_back(input[i]);
	return res;
}
```


