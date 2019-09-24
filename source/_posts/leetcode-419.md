---
title: leetcode-419 Battleships in a Board
date: 2019-09-24 09:19:23
tags: 
  - One way pass
categories: 
  - 算法
description: 计数二维矩阵中的战舰数目
---

### 描述

Given an 2D board, count how many battleships are in it. The battleships are represented with `'X'``'.'`

- You receive a valid board, made of only battleships or empty slots.
- Battleships can only be placed horizontally or vertically. In other words, they can only be made of the shape `1xN` (1 row, N columns) or `Nx1` (N rows, 1 column), where N can be of any size.
- At least one horizontal or vertical cell separates between two battleships - there are no adjacent battleships.

**Example:**

```
X..X
...X
...X
```

In the above board there are **2** battleships.

### 思路

一开始想到 DP 来做, 当前位置的数和三个数有关 : `dp[i-1][j-1] , dp[i][j-1] , dp[i-1][j]` . 当且仅当 : `board[i][j] = 'X'` 并且左节点和上节点都是 `.` 时 , 战舰数目加一

```cpp
int Solution::solutionDP(vector<vector<char>> &board) {
    int row = board.size();
    int col = board[0].size();
    if (row * col == 0)
        return 0;
    //construct the dp arr
    vector<vector<int>> dp(row, vector<int>(col, 0));

    if (board[0][0] == dot)
        dp[0][0] = 0;
    else if (board[0][0] == ship)
        dp[0][0] = 1;
    //首先进行边界的分析. 具体为 : 最左侧和最上侧
    for (int i = 1; i < col; ++i) {
        if (board[0][i - 1] == dot && board[0][i] == ship)
            dp[0][i] = dp[0][i - 1] + 1;
        else
            dp[0][i] = dp[0][i - 1];
    }

    for (int i = 1; i < row; ++i) {
        if (board[i - 1][0] == dot && board[i][0] == ship)
            dp[i][0] = dp[i - 1][0] + 1;
        else
            dp[i][0] = dp[i - 1][0];
    }

    for (int i = 1; i < row; ++i) {
        for (int j = 1; j < col; ++j) {
            int basic = dp[i - 1][j] + dp[i][j - 1] - dp[i - 1][j - 1];
            //add a ship
            if (board[i][j] == ship && board[i - 1][j] == dot && board[i][j - 1] == dot) {
                dp[i][j] = basic + 1;
            } else {
                dp[i][j] = basic;
            }
        }
    }

    return dp[row - 1][col - 1];
}
```

但是后来一想，根本不需要这么麻烦啊. 谨记 ：当状态转移方程出现了 `dp[i][0] = dp[i - 1][0]` 这样的方式, 都是可以进行空间复杂度上的优化的 . 在这里就可以直接优化到 `O(1)`.

我们判定当前的 `board[i][j]`:

- 如果是 `X`, 那么当且仅当 `board[i-1][j] , board[i][j-1]`都是 `.` 时需要把计数器加一
- 其他所有情况, 计数器不变即可

### 实现

Mem rank : 100 %

Speed rank : 99.4 %

```cpp
int Solution::solutionOnePass(vector<vector<char>> &board) {
    int ans = 0;
    int m = board.size();
    int n = board[0].size();
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (board[i][j] == ship &&
                ((i == 0 || board[i - 1][j] == dot)) && (j == 0 || board[i][j - 1] == dot)) {
                ++ans;
            }
        }
    }
    return ans;
}
```

