---
title: leetcode_470
date: 2019-10-15 09:18:45
categories: 
  - 算法
description: 使用rand7()函数来实现rand10()
---

### 描述

Given a function `rand7` which generates a uniform random integer in the range 1 to 7, write a function `rand10` which generates a uniform random integer in the range 1 to 10.

Do NOT use system's `Math.random()`.

**Example 1:**

```
Input: 1
Output: [7]
```

**Example 2:**

```
Input: 2
Output: [8,4]
```

**Example 3:**

```
Input: 3
Output: [8,1,10]
```

 

### 思路

非常非常有趣的一个题目！

已知有一个函数rand7() , 需要我们给出rand10()的算法实现

思路可以说百花齐放. 这里我给出几个

#### 两次rand7扩展范围

一次 `rand7()` 的范围是 `[1,7] ` , 那么 `(rand7() - 1) * 7 + rand7() - 1` 的范围就是 `[0,47]`

这时我们舍弃大于等于40的部分 即可

```cpp
class Solution {
public:
    int rand10() {
        int rand40 = 40;
        while (rand40 >= 40) {
            rand40 = (rand7() - 1) * 7 + rand7() - 1;
        }
        return rand40 % 10 + 1;
    }
    
};
```

#### 暴力版随机

没有用到一次rand7()

```cpp
class Solution {
public:
    int rand10() {
        return 1 + count++ % 10;
    }
    
private:
    int count = 0;    
};
```