---
title: leetcode_1043
date: 2019-11-08 19:59:05
tags: 
  - Array
categories: 
  - 算法
description: 给定一个整数数组A，您将该数组划分为(连续的)长度最多为k的子数组。在划分之后，每个子数组的值都更改为该子数组的最大值。分区后返回给定数组的最大和。
---

### 描述

Given an integer array `A`, you partition the array into (contiguous) subarrays of length at most `K`. After partitioning, each subarray has their values changed to become the maximum value of that subarray.

Return the largest sum of the given array after partitioning.

 

**Example 1:**

```
Input: A = [1,15,7,9,2,5,10], K = 3
Output: 84
Explanation: A becomes [15,15,15,9,10,10,10]
```

### 思路

构造一个 _dp_ 数组, 满足

- `dp[i]` 表示max sum of A[0: i+1)

那么我们在接下来的每一个 `dp[i]` 中, 需要不断向前查询.将 `i` 之前的数组内容分成两部分：

1. 已经dp纳入计算的部分 _pre_
2. 当前计算的 _cur_

那么我们逐步增加 _cur_ 的长度 ( from 1 to min(k,i) ) , 找到这个 _cur_ 子数组的最大值 , 乘以 _cur_ 长度，那么就是 _cur_ 子数组的最优解

```cpp
		int maxSumAfterPartitioning(vector<int> &A, int K) {
        int n = A.size();
        vector<int> dp(n + 1);
        for (int i = 1; i <= n; i++) {
            int cur_max = -1;
          	// j from 1 to min(K,i)
            for (int j = 1; j <= min(K, i); j++) {
                cur_max = max(cur_max, A[i - j]);
                dp[i] = max(dp[i], dp[i - j] + cur_max * j); 
            }
        }
        return dp[n];
    }
```

