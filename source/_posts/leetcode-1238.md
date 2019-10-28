---
title: leetcode_1238 Grey Code
date: 2019-10-28 09:12:41
categories: 
  - 算法
description: 给你两个整数 n 和 start。你的任务是返回任意 (0,1,2,,...,2^n-1) 的排列 p (满足格雷码序列)
---

引用：https://cp-algorithms.com/algebra/gray-code.html

### 描述

Given 2 integers `n` and `start`. Your task is return **any** permutation `p` of `(0,1,2.....,2^n -1) `such that :

- `p[0] = start`
- `p[i]` and `p[i+1]` differ by only one bit in their binary representation.
- `p[0]` and `p[2^n -1]` must also differ by only one bit in their binary representation.

### 思路

说实话，一开始没思路. 之后根据discussion才了解到格雷码的概念

实现特别简单，直接套公式

```cpp
class Solution {
    public List<Integer> circularPermutation(int n, int start) {
       List<Integer> res = new ArrayList<>();
        for (int i = 0; i < 1 << n; ++i)
            res.add(start ^ i ^ i >> 1);
        return res; 
    }
}
```

### 格雷码

格雷码是一种准权码，具有一种反射特性和循环特性的单步自补码，它的循环、单步特性消除了随机取数时出现重大误差的可能，它的反射、自补特性使得求反非常方便。格雷码属于可靠性编码，是一种错误最小化的编码方式。

自然二进制码可以直接由数/模转换器转换成模拟信号，但是在某些情况下，例如从十进制的3转换到4的二进制时，每一位都要变。3的二进制位：**011**，而4的二进制位：**100**，所以这就会使数字电路产生很大的尖峰电流脉冲。而格雷码则没有这一缺点，它是一种数字排序系统，其中的所有相邻整数在它们的数字表示中只有一个数字不同。它在任意两个相邻的数之间转换时，只有一个数位发生变化。它大大地减少了由一个状态到下一个状态时逻辑的混淆。另外由于最大数与最小数之间也仅一个位不同，故通常又叫**格雷反射码**或**循环码**。

#### 求解

```
　　G(i) = B(i+1) XOR B(i); 0 <= i < N - 1
　　G(i) = B(i);      i = N - 1
```

也就是说, 格雷码根据当前位和前一位进行 **异或** 运算 , 最高位不变

那么

```
G = G ^ (G >> 1)
```

只能说，这个没接触过的知识，让我在一开始做的时候根本找不到入手点... 还是太弱了