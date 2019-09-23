---
title: leetcode-740 Delete and earn
date: 2019-09-23 23:23:08
tags: 
  - DP
categories: 
  - 算法
description: 给定一个整数数组 nums ，你可以对它进行一些操作。每次操作中，选择任意一个 nums[i] ，删除它并获得 nums[i] 的点数。之后，你必须删除每个等于 nums[i] - 1 或 nums[i] + 1 的元素。开始你拥有 0 个点数。返回你能通过这些操作获得的最大点数。
---

### 描述

给定一个整数数组 nums ，你可以对它进行一些操作。每次操作中，选择任意一个 `nums[i]` ，删除它并获得 `nums[i]` 的点数。之后，你必须删除每个等于 `nums[i] - 1` 或 `nums[i] + 1` 的元素。开始你拥有 0 个点数。返回你能通过这些操作获得的最大点数。

### 思路

典型的使用 DP . 每一次的pick选取都会导致近邻的元素被同时删除. 这时候我们就需要考虑 `选取pick的时候进行profit最大化` . 这样也有一点贪心算法的味道. 具体是不是我也不清楚hhh

首先需要得到数组的各个数产生的 `profit` 大小 , 这里我使用了一个数组 `profits` 来进行表示.

此后进行 `dp` 数组的维护, 状态转移方程分析如下 :

在选取位置 **i** 处的总效益时

1. 如果选取 `profit[i]` ，那么 `profit[i-1]` 必定没有被选取. 我们得到 `profit[i] + dp[i-2]`
2. 如果不选取 `profit[i]` , 那么就有 `dp[i] = dp[i-1]`

将如上两种情况进行最大值比较，状态转移方程如下 :

```
dp[i] = max (dp[i-1] , dp[i-2] + profit[i])
```

### Post on LC

The  `profit` means the total profit of the single number.
For example , the array [2,2,3,3,4] . The profit of **2** is **4**  and the profit of **3** is **6**. 

Then we could maintain the **dp** array. That is : If we choose the current **i** to pick , the we cannot pick **i-1** . So if **i-1** is chosen , we have to check the dp[i-2] + profits[i] to see the total profit , otherwise we compare it with the dp[i-1]
Thus the `dp[i]=max(dp[i-1],dp[i-2]+profits[i])`
```c++
class Solution {
public:
    int deleteAndEarn(vector<int>& nums) {
        const int n = 10001;
        
        vector<int> profits = vector<int>(n,0);
        
        for(auto n : nums)
            profits[n]+=n;
        
        vector<int> dp = vector<int>(n,0);
        dp[1] = profits[1];
        for(int i= 2 ; i< n ;++i){
            dp[i] = max(dp[i-1],profits[i] + dp[i-2]);
        }
        return dp[n-1];   
    }
};
```