---
title: leetcode-101 Symmetrc Tree 对称树的判定
date: 2019-09-01 23:23:08
tags: 
  - Tree
categories: 
  - 算法
description: 判定一颗给定树是否对称
---

## Leet code 101 对称树判定

树的数据格式如下

```c++
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;

    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```

我们需要判定一棵树是否按照根节点镜像对称，属于比较简答的树的递归问题

一颗对称树可以如下：

```
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

#### 递归版本

首先想到的就是，可以顺着左右子树分别向下递归。相同深度时，比较的两棵子树的根节点相同，并且可以从 `左子树的左儿子` 和 `右子树的右儿子` 是否相同，并且 `左子树的右儿子` 和 `左子树的左儿子`是否相同入手。

代码如下

```c++
class Solution {
public:
    //check if the tree is symmetric
    bool isSymmetric(TreeNode *root) {
        if (root == nullptr) {
            return true;
        }
        return isSymmetric(root->left, root->right);
    }

private:
    bool isSymmetric(TreeNode *left, TreeNode *right) {
        if (!left && !right)
            return true;
        else if (!left || !right)
            return false;
        return left->val == right->val &&
               isSymmetric(left->left, right->right) &&
               isSymmetric(left->right, right->left);
    }
};
```

