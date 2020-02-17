---
title: 初探Anna
date: 2020-02-17 11:05:56
tags: 笔记
cover_img: https://images.unsplash.com/photo-1551342909-187e24a5797d?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=675&q=80
feature_img: https://images.unsplash.com/photo-1517999349371-c43520457b23?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=634&q=80
description: 由伯克利开发的新一代数据库引擎
keywords:
---

## 初探Anna

解决目标：

- *data scaling* ：牵涉到数据分片 **partition**
- *workload scaling* ：牵涉到 **multi-master replication**
- 硬件最大化利用 & 多核计算机性能 ：**wait-free execution** (每一个core尽可能让他们一直工作)
- 无协调的一致性 **coordination-free consistency model**
  - 一致性哈希 & 哈希环

> ##### Lattice 晶格
>
> 离散里面的一种偏序结构. 整个图里面只有一个起点和一个终点
>
> 达到的事务级别 ：**read-committed transaction**

#### 可用的分布式状态模型类别

- 共享内存
  - Lattice也不能解决异步请求的数据同步问题
- 消息传递
  - **Single-Master**
    - Key值只有一个副本，可以有比较好的一致性；
    - 速度不能超过一个node ，这是一个瓶颈
  - **Multi-Master**
    - Key值存在多个副本

> **Anna** 使用的是无协调的 *multi-master replication* ，并且基于 **lattice** 来进行一致性的保证

