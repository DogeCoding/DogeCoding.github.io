title: 剑指Offer —— 滑动窗口的最大值
tags: Algorithm
---

> 题目来源:**[滑动窗口的最大值](https://www.nowcoder.com/practice/1624bc35a45c42c0bc17d17fa0cba788?tpId=13&tqId=11217&tPage=4&rp=4&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking)**

***    
## 题目描述
给定一个数组和滑动窗口的大小，找出所有滑动窗口里数值的最大值。例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为{4,4,6,6,6,5}； 针对数组{2,3,4,2,6,2,5,1}的滑动窗口有以下6个： {[2,3,4],2,6,2,5,1}， {2,[3,4,2],6,2,5,1}， {2,3,[4,2,6],2,5,1}， {2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}， {2,3,4,2,6,[2,5,1]}。
 

## 解题思路
用一个双端队列维护当前滑动窗口的状态，队首是当前窗口最大值的下标，当窗口滑动进入一个新值k时，k从队尾依次向前比较，比k小的全部出队，保障了k的权重应有的位置

``` //
vector<int> maxInWindows(const vector<int> &num, unsigned int size) {
    vector<int> ans;
    deque<int> q;
    if (num.size() < size || size == 0) 
        return ans;
    for (int i = 0; i < size; i++) {
        while(!q.empty() && num[i] > num[q.back()])
        q.pop_back();
        q.push_back(i);
    }
    for (int i = size; i < num.size(); i++) {
        ans.push_back(num[q.front()]);
        while (!q.empty() && num[i] >= num[q.back()])
            q.pop_back();
        while (!q.empty() && q.front() <= i-size)
            q.pop_front();
        q.push_back(i);
    }
    ans.push_back(num[q.front()]);
    return ans;
}
```

