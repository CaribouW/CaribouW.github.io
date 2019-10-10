---
title: leetcode_941
date: 2019-10-10 08:41:32
tags: 
  - Array
categories: 
  - 算法
description: 给定一个整数数组，当且仅当它是一个有效的山地数组时返回true。
---

### 描述

Given an array `A` of integers, return `true` if and only if it is a *valid mountain array*.

Recall that A is a mountain array if and only if:

- `A.length >= 3`

- There exists some i with `0 < i < A.length - 1`

   such that:

  - `A[0] < A[1] < ... A[i-1] < A[i]`
  - `A[i] > A[i+1] > ... > A[A.length - 1]`

 

**Example 1:**

```
Input: [2,1]
Output: false
```

**Example 2:**

```
Input: [3,5,5]
Output: false
```

**Example 3:**

```
Input: [0,3,2,1]
Output: true
```

### 思路

- 数组长度不小于3
- 数组中不存在连续的相同的数
- 需要满足：
  - 存在最高点
  - 最高点出现之前需要上升
  - 最高点之后严格递减

```cpp
class Solution {
public:
    bool validMountainArray(vector<int>& A) {
        if (A.size() < 3) return false;

        int mount = A[0];
        bool reachTop = false , asc = false;
        for(int i=1; i<A.size();++i){
            if (mount == A[i]) return false;
            if (!reachTop){
                if(mount < A[i]) {
                    asc = true;
                }
                else if (mount > A[i]){
                    reachTop = true;
                }
                mount = A[i];
            }else{
                if(mount <= A[i]) return false;
                else{
                    mount = A[i];
                }
            }
        }
        return asc && reachTop;
    }
};
```

