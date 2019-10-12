---
title: leetcode_1143_最长公共序列
date: 2019-10-12 08:25:25
categories: 
  - 算法
description: 给定两个字符串text1和text2，返回它们最长的公共子序列的长度。字符串的子序列是从原始字符串生成的新字符串，其中删除了一些字符（可以是一个字符），而不会更改其余字符的相对顺序。(例如，“ ace”是“ abcde”的子序列，而“ aec”则不是).
---

### 描述

Given two strings text1 and text2, return the length of their longest common subsequence.

A subsequence of a string is a new string generated from the original string with some characters(can be none) deleted without changing the relative order of the remaining characters. (eg, "ace" is a subsequence of "abcde" while "aec" is not). A common subsequence of two strings is a subsequence that is common to both strings.

**Example 1:**

```
Input: text1 = "abcde", text2 = "ace" 
Output: 3  
Explanation: The longest common subsequence is "ace" and its length is 3.
```

**Example 2:**

```
Input: text1 = "abc", text2 = "abc"
Output: 3
Explanation: The longest common subsequence is "abc" and its length is 3.
```

**Example 3:**

```
Input: text1 = "abc", text2 = "def"
Output: 0
Explanation: There is no such common subsequence, so the result is 0.
```

### 思路

使用DP来进行实现. 第一版的想法就是构造一个二维数组 `dp[m][n]` , 对于 `dp[i][j]` 表示 `text1.subStr(0,i)` 和 `text2.subStr(0,j)`的最长公共序列长度.

状态转移方程如下：

- 如果当前 `text1[i]=text2[j]`,  `dp[i][j]=1+dp[i-1][j-1]`
- 否则就是之前进行匹配的 `dp[i][j]=max(dp[i-1][j],dp[i][j-1])`

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        int dp[][] = new int[m + 1][n + 1];
        
        for(int i = 0; i <= m ; ++i){
            for(int j  = 0 ; j <= n ; ++j){
                if(i * j == 0) 
                    dp[i][j] = 0;
                else if (text1.charAt(i-1) == text2.charAt(j-1)){
                    dp[i][j] = 1 + dp[i-1][j-1];
                }else{
                    dp[i][j] = Math.max(dp[i-1][j] , dp[i][j-1]);
                }
            }
        }
        return dp[m][n];
    }
}
```