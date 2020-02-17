---
title: 初探Anna
date: 2020-02-17 11:05:56
tags: 笔记
cover_img: https://images.unsplash.com/photo-1551342909-187e24a5797d?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=675&q=80
feature_img: https://images.unsplash.com/photo-1517999349371-c43520457b23?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=634&q=80
description: 由伯克利开发的新一代数据库引擎
keywords:
---

## Anna初探

### 背景

实习中主要做的是数据库引擎的开发。这一次遇到的是伯克利开发的 *Anna* ，将透过两篇paper来大致讲述一下入门的内容。

### 进入正题

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
> 达到的事务隔离级别 ：**read-committed transaction**
>
> 就这一点，论文中提到了 **casual consistency** 和 **read committed** 两种一致性的*level* ，具体就留到后面展开论述

#### 可用的分布式状态模型类别

- 共享内存

  该方式常常需要额外的加锁去锁来进行一致性维护，因而极大地影响了系统运行效率

  Lattice也不能解决异步请求的数据同步问题，它最重要的 **merge function** 也是需要

- 消息传递

采用了消息传递的架构后，工作节点能够拥有类似于 **local-thread** 的私有状态(*state*)。由于对外界不可见，也就没有了并发的问题

但是由于各个副本之间牵涉到数据的同步，这就会有一个新的 **状态一致性** 问题的出现，我们需要采用新的机制来进行改进

**Single-Master**

- Key值只有一个副本，可以有比较好的一致性
- 问题：速度不能超过一个node ，这是一个瓶颈

**Multi-Master**

- Key值存在于多个副本中
- 周期性进行广播发送*update* 信息
- 问题：一致性不能够保证

> **Anna** 使用的是无协调的 *multi-master replication* ，并且基于 **lattice** 来进行一致性的保证

### Anna架构

主要根据两篇(2018 , 2019) 中先后两个版本的 *Anna* 架构不同点入手

> #### Lattice based
>
> *Lattice* 是这个框架主打的内容：在给定的一个偏序条件下，只要该偏序关系重的 **least upper bound** 又满足了
>
> - Commutativity 交换律
> - Associativity 结合律
> - Idempotence 幂等
>
> 那么就可以说是满足一个晶格的关系。这样的系统我们也称作 **ACI system**

### 回到我们的一致性问题

*ACI* 系统中，由于它的组件也必然满足一个lattice关系，我们就可以采用 **自底向上组合** 的方式来进行系统逻辑的构建

<img src="https://i.loli.net/2020/02/17/cYaxepWz2ZdgVTj.png" alt="image.png" style="zoom: 77%;" />

如上图所示 （*casual consistency*）

- 每一个工作节点(worker)的状态都采用 **MapLattice** 来进行表示
- **MapLattice**的键，是一种不可变的类型(*immutable*) ；对于值，则是 **ValueLattice**

> 对于 **MapLattice** 的 *merge* 算子，会对两个 MapLattice 的键进行 *Union* 操作，若键相同，那么会进行 **ValueLattice** 的 *merge* 算子

---

接下来我们需要看之前提及的 **casual consistency** 和 **read committed** 

#### Casual Consistency

> A , B 两个用户同时在进行事务处理，如果 B 能够见到 A 的更新，那么 B 的更新将会 *Overwrite* A用户的更新；如果不能够看到，那么就会触发数据记录的 *merge* 算子

关键点：**vector lock**

支持 *casual consistency* 的方案中，*vector lock* 的键由客户端的 *proxy ids* 组成，值则由相对应的版本号组成。他们相互具有关联关系。这里的 *version number* 可以由 **MaxIntLattice** 来进行实现，它每次都取**每一次更新前后的最大值**，所以它的值必定是递增的，也契合了版本号递增的特点。

当进行一次写操作的时候，会把 **vector lock** 的版本号进行增加，连同数据项一起进行写入

图中的 **pair lattice** 也具有 *merge* 算子，具体算法如下

> 对于两个 **pair lattice** P(a,b) 和 Q(a,b) ，如果 P.a > Q.a (表示偏序大小关系) ，我们称 **P casually follows Q** ，并且将 *merge* 结果设置为 P
>
> 如果不能够进行比较，那么*merge* 结果就是
>
> ![image.png](https://i.loli.net/2020/02/17/cbe2SQOmtJ9KPrx.png)

#### Read Committed

和数据库事务隔离级别一样，读提交的要求就是避免 **脏读** 和 **脏写**

##### 脏写

多个写事件的执行中，我们只需要添加一个 *larger timestamp wins* 策略

##### 脏读

添加事务写入的缓冲区，保证未提交的事务不会被放入 *KVS*

### 遗留的一些问题

- Critical path of every request 是什么？

