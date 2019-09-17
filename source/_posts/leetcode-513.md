---
title: leetcode-513 Find Bottom Left Tree Value
date: 2019-09-17 23:23:08
tags: 
  - Tree
categories: 
  - 算法
description: 寻找二叉树中最深一层的最左侧节点
---

## 513 寻找最深最左二叉树节点

Given a binary tree, find the leftmost value in the last row of the tree.

### 思路

二叉树的层次序遍历. 但是有所不同的是, 需要判定每一层的开始位置 。所以我们在每一层queue push 进入之后，添加一个nullptr来进行标记

- 当队列中第一次遇到 `nullptr` 处在队列末尾, 表示队首节点是最左侧 , 更新 `ans`。后续再次遇到 `nullptr` 在队尾时不再计入
- 当`nullptr`处在队首, 表示一行已经被遍历完全. 直接把 `nullptr` 重新放置到队尾. 置标记位为 `true`

### 实现

- 首先我们来看一下简单的二叉树层次序遍历的逻辑

```cpp
void Solution::levelPrint(TreeNode *root) {
    queue<TreeNode *> queue;

    queue.push(root);
    while (!queue.empty()) {
        //visit
        TreeNode *node = queue.front();
        cout << node->val << ",";
        //pop
        queue.pop();
        //push back
        if (node->left)
            queue.push(node->left);
        if (node->right)
            queue.push(node->right);
    }
}
```

非常明显，我们一直进行队列的出、入, 当队列为空时, 表示我们已经完成了层次序遍历

- 我们继续看本题. 由于队尾标识 `nullptr` 会一直存在, 我们无法根据 `队列为空` 来进行遍历终结的判定. 那么什么时候已经遍历完了呢？ 非常简单，整个队列只有 `nullptr` 一个元素. 

具体代码如下：

```cpp
int Solution::findBottomLeftValue(TreeNode *root) {
    queue<TreeNode *> queue;
    auto *ans = new TreeNode(0);
    queue.push(root);
    queue.push(nullptr);
    bool endFlag = true;
    while (!queue.empty()) {
        //visit
        TreeNode *node = queue.front();
        queue.pop();
        if (node) {//第一个不是空
            if (endFlag && !queue.back()) {
                ans->val = node->val;
                endFlag = false;
            }
            //push back
            if (node->left) {
                queue.push(node->left);
                endFlag = false;
            }
            if (node->right) {
                queue.push(node->right);
                endFlag = false;
            }
        }
        if (queue.size() == 1)
            break;
        if (queue.front() == nullptr) {
            queue.pop();
            queue.push(nullptr);
            endFlag = true;
        }

    }
    return ans->val;
}

```

