---
title: leetcode-148 Sort Linked List 基于链表排序
date: 2019-09-01 23:23:08
tags: 
  - List
categories: 
  - 算法
description: 针对链表进行排序
---

## 148 链表排序

给定一个链表，将其进行排序。主要考察了基本的排序内容

给定数据结构 ListNode

```c++
struct ListNode {
    int val;
    ListNode *next;

    ListNode(int x) : val(x), next(nullptr) {}
};
```

#### 基于链表的归并排序

链表本身无法直接获取索引的特点，归并排序首先需要得到中间节点，并且能够从中间节点开始分隔。

之后的排序方式和基于数组的归并排序类似，难度不大。

```c++
  //归并排序主函数
    ListNode *mergeSortList(ListNode *head) {
        if (head == nullptr || head->next == nullptr)
            return head;
        else {
            ListNode *fast = head, *slow = head;
            while (fast->next != nullptr && fast->next->next != nullptr) {
                fast = fast->next->next;
                slow = slow->next;
            }
            ListNode *preHead = head;
            ListNode *postHead = slow->next;
            //截断
            slow->next = nullptr;
            preHead = mergeSortList(preHead);
            postHead = mergeSortList(postHead);
            return merge(preHead, postHead);
        }
    }

    //归并操作
    ListNode *merge(ListNode *pre, ListNode *post) {
        if (pre == nullptr)
            return post;
        if (post == nullptr)
            return pre;
        ListNode *result, *p;
        //let pre to be the head
        if (pre->val < post->val) {
            result = pre;
            pre = pre->next;
        } else {
            result = post;
            post = post->next;
        }
        p = result;
        while (pre != nullptr && post != nullptr) {
            if(pre->val < post->val){
                p->next = pre;
                pre=pre->next;
            } else{
                p->next = post;
                post = post->next;
            }
            p=p->next;
        }
        if(pre == nullptr){
            p->next = post;
        } else {
            p->next = pre;
        }

        return result;
    }
```

