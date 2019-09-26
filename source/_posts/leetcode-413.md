---
title: leetcode-413 Arithmetic Slices
date: 2019-09-26 20:15:02
categories: 
  - 算法
description: 等差数列计数
---

### 描述

A sequence of number is called arithmetic if it consists of at least three elements and if the difference between any two consecutive elements is the same.

For example, these are arithmetic sequence:

```
1, 3, 5, 7, 9
7, 7, 7, 7
3, -1, -5, -9
```

The following sequence is not arithmetic.

```
1, 1, 2, 5, 7
```



A zero-indexed array A consisting of N numbers is given. A slice of that array is any pair of integers (P, Q) such that 0 <= P < Q < N.

A slice (P, Q) of array A is called arithmetic if the sequence:
A[P], A[p + 1], ..., A[Q - 1], A[Q] is arithmetic. In particular, this means that P + 1 < Q.

The function should return the number of arithmetic slices in the array A.

### 实现

简单的DP问题. 每一次遇到 `dp[i]` 的时候, 需要考虑之前的 **最长等差数列的长度** .

如果最长等差数列的长度为 $n$ , 那么可以得到状态转移方程 `dp[i] = dp[i-1] + n​`

```cpp
class Solution {
public:
    int numberOfArithmeticSlices(vector<int>& A) {
        int n = A.size();
        if(n <= 2) return 0;
        
        vector<int> dp(n,0);
        dp[0] = dp[1] = 0;
        
        for(int i = 2 ; i < n ; ++i){
            dp[i] = dp[i-1] + countArithLength(A,i);
        }
        
        return dp[n-1];
    }
    
private:
    int countArithLength(vector<int>& A,int endIndex){
        int pad = A[endIndex] - A[endIndex - 1] , ans = 0;
        for(int i = endIndex-1 ; i >= 1 ; --i){
            if(A[i] - A[i-1] == pad) ++ans;
            else break;
        }
        return ans;
    }
};
```

speed 100% ;

mem 100%