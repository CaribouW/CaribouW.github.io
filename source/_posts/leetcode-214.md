---
title: leetcode_214
date: 2019-10-10 08:41:32
tags: 
  - String
categories: 
  - 算法
description: 对于给定的字符串s，可以通过在其前面添加字符将其转换为回文。查找并返回通过执行此转换可以找到的最短回文。
---

### 描述

对于给定的字符串s，可以通过在其前面添加字符将其转换为回文。查找并返回通过执行此转换可以找到的最短回文

**Example 1:**

```
Input: "aacecaaa"
Output: "aaacecaaa"
```

**Example 2:**

```
Input: "abcd"
Output: "dcbabcd"
```

### 思路

首先想到的, 我们需要找到字符串的中轴字符. 从该字符往外进行查找, 其最坏情况就是以**第一个字符**来进行扩展. 最好情况就是从中间的字符进行扩展. 我们可以从最中间开始进行查找, 第一个找到的就是最短的字符串

```cpp
string shortestPalindrome(string s) {
        int n = s.length();
        //从最好情况往最差情况找
        //起点是 .5
        int index = 0;
        string ans;
        while (n >= 0) {
            if (n % 2 == 0) {
                index = n / 2;
                ans = findPalindrome(s, index - 1, index);
            } else {
                index = (n - 1) / 2;
                ans = findPalindrome(s, index, index);
            }
            if (!ans.empty())return ans;
            --n;
        }
        return findPalindrome(s, 0, 0);
    }

    string findPalindrome(string s, int leftIndex, int rightIndex) {
        int n = s.length();
        if (rightIndex >= n) return "";
        while (leftIndex >= 0 && rightIndex < n) {
            if (s[leftIndex] != s[rightIndex]) return "";
            --leftIndex;
            ++rightIndex;
        }
        string ans = s;
        while (rightIndex < n) {
            ans = s[rightIndex++] + ans;
        }
        return ans;
    }
```

BTW , 这个Solution 很不幸的TLE了hh

### Solutions

#### Brute force

将原有的字符串 `s` 看做是两个部分：前半部分是一个 Palidorm **A** , 后半部分 **B** 不是. 那么当我们找到最长的 **Palidorm** 时, 只需要把第二个部分回转, 复制到首部就可以. (这样相当于 $B^rAB$ )

```cpp
string shortestPalindrome(string s)
{
    int n = s.size();
    string rev(s);
    reverse(rev.begin(), rev.end());
    int j = 0;
    for (int i = 0; i < n; i++) {
        if (s.substr(0, n - i) == rev.substr(i))
            return rev.substr(0, i) + s;
    }
    return "";
}
```

但是很不幸 , 这个在最后一个用例也 TLE 翻车了

#### Manacher's Algorithm 马拉车算法

我的算法中, 主要的想法就是从各个可能的中心点往两边进行展开，时间复杂度达到了 $O(N^2)$。 而Manacher's Algorithm 将查询回文串的时间复杂度优化到了 $O(N)$

1. 在原有的字符串中，相邻两个字符之间都插入 $\#$ ，两边插入标识开始和结尾的符号。最终该字符串长度一定是**奇数**

![img](https://pic4.zhimg.com/80/v2-8c613143224234c971d9b5353a3f05aa_hd.jpg)

2. 最终需要构造出一个生成数组 $P$ , 它如下表示 : 

每一个位置都表示以当前位置为中心进行扩展的字符串长度 (包括 **#** )  那么在原字符串中，其

字符串的位置是 `(i - P[i])/2 , (i - P[i])/2 + P[i] - 1`

回文串长度是 `P[i] - 1`

在新的字符串中, `Right_index = P[i] + i` . 也就是说，我们可以根据 **P[i]**来获取最 **右** 字符的位置
![img](https://pic4.zhimg.com/80/v2-d570f7a9e732d577d556b0e6cff1d263_hd.jpg)

3. 数组 $P$ 生成

我们获取 $P[i]$ 的时候不再总是根据当前为中心进行两边扩展，而是根据 $P[j], for\;0\le j <i$ 来进行

假设我们需要求 `P[i]` , 此前求出的最大回文串的中心 index 为 `Po` , 它的最右侧 index 为 `P`

#### KMP算法

```C
int KMP(char *t, char *p) {
    int i = 0;
    int j = 0;

    while (i < strlen(t) && j < strlen(p)) {
        if (j == -1 || t[i] == p[j]) {
            i++;
            j++;
        } else
            j = next[j];
    }

    if (j == strlen(p))
        return i - j;
    else
        return -1;
}

void getNext(char *p, int *next) {
    next[0] = -1;
    int i = 0, j = -1;

    while (i < strlen(p)) {
        if (j == -1 || p[i] == p[j]) {
            ++i;
            ++j;
            next[i] = j;
        } else
            j = next[j];
    }
}

```

给出如下 PMT 的示例

![img](https://pic1.zhimg.com/80/v2-40b4885aace7b31499da9b90b7c46ed3_hd.jpg)

PMT的含义：

```
对于字符串 s , index 为 i 时，PMT[i] 表示它的 [前缀集合] 交  [后缀集合] 之后的最长匹配长度
其中前缀集合和后缀集合不包含数组本身
```

为编程方便需要，我们给出 next 数组。它实质上是 PMT数组右移一位, 且`next[0] = -1`

根据这个，我们可以给出算法逻辑

- 如果当前指针匹配，那么原字符串 target 和匹配串 pattern 的指针分别自增即可
- 如果发生不匹配，那么 target 不变动 (懒惰式), 匹配串指针 `j = next[j]`

相比如马拉车算法，KMP算法的适用性还是更加广泛的

---

我们回到这个问题上来，回顾一下我们的目标

```
add string before the input so the result string will be a palindrome
在原有的字符串之前插入一些字符串，使得它最终是一个回文串
```

在 **Brute force** 里面已经提到：我们先找出最长的、从0下标开始的最长回文串 P , 那么只要把 **剩下的子串反转之后insert到开头**就可以构造出结果. 这一点很好理解

那么我们的目标就变成

```
寻找从0下标开始的最长回文串
```

那么我们怎么把这个问题和之前说到的 **KMP** 算法结合呢？同样的也是进行字符串匹配. 我们需要把字符串进行预先的反转，而后把回文的匹配转化为正向的字符串匹配. 具体算法流程如下：

1. 给定字符串 **s** , 我们反转它, 得到 **s\*** . 生成字符串 `s#s*`. 这样一来，我们只需要生成 KMP 表即可 (因为 **s** 和 **s\***就是前缀和后缀)

2. 假设新的字符串 `s#s*` 的长度是 m , 那么 next[m] 的值就是我们所需要查找的 `从0下标开始的最长回文串`的长度. 取得 s.substr(m) , 进行反转之后复制到首部即可

```java
public String shortestPalindrome(String s) {
    String temp = s + "#" + new StringBuilder(s).reverse().toString();
    int[] table = getTable(temp);
    
    //get the maximum palin part in s starts from 0
    return new StringBuilder(s.substring(table[table.length - 1])).reverse().toString() + s;
}

public int[] getTable(String s){
    //get lookup table
    int[] table = new int[s.length()];
    
    //pointer that points to matched char in prefix part
    
    int index = 0;
    //skip index 0, we will not match a string with itself
    for(int i = 1; i < s.length(); i++){
        if(s.charAt(index) == s.charAt(i)){
            //we can extend match in prefix and postfix
            table[i] = table[i-1] + 1;
            index ++;
        }else{
            //match failed, we try to match a shorter substring
            
            //by assigning index to table[i-1], we will shorten the match string length, and jump to the 
            //prefix part that we used to match postfix ended at i - 1
            index = table[i-1];
            
            while(index > 0 && s.charAt(index) != s.charAt(i)){
                //we will try to shorten the match string length until we revert to the beginning of match (index 1)
                index = table[index-1];
            }
            
            //when we are here may either found a match char or we reach the boundary and still no luck
            //so we need check char match
            if(s.charAt(index) == s.charAt(i)){
                //if match, then extend one char 
                index ++ ;
            }
            
            table[i] = index;
        }
        
    }
    
    return table;
}
```