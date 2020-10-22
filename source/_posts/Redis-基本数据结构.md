---
title: Redis-基本数据结构
date: 2020-03-24 10:03:53
tags: 笔记
cover_img: https://images.unsplash.com/photo-1522968941782-e27ac665baa3?ixlib=rb-1.2.1&auto=format&fit=crop&w=634&q=80
cover: https://images.unsplash.com/photo-1522968941782-e27ac665baa3?ixlib=rb-1.2.1&auto=format&fit=crop&w=634&q=80
feature_img: https://images.unsplash.com/photo-1518976024611-28bf4b48222e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=632&q=80
description:
keywords: Redis
---

## Redis 基本数据结构

> *Redis* 的数据类型有五大类，分别是 **列表、字符串、哈希表、有序集合、无序集合**
>
> 字符串底层：SDS
>
> 列表底层：链表或者是 *ziplist* 压缩列表
>
> 哈希对象：ziplist 或者是 hashtable
>
> 集合：*intset* 或 *hashtable*
>
> 有序集合：*ziplist* 或 *skiplist & dictionary*

底层的数据结构实现讲解

### 简单动态字符串 SDS

#### 涉及数据结构

```c
struct sdshdr
{

    // buf 中已占用空间的长度
    int len;

    // buf 中剩余可用空间的长度
    int free;

    // 数据空间
    char buf[];
};
```

#### 空间预加载策略

当我们进行 `sdscat(sds s1 , const char* t)` 的时候，**可能** 会引发空间重新分配

- 如果 **free space** 足够，那么不进行分配
- 如果不够，看 **t** 的大小是不是超过 1M (`SDS_MAX_PREALLOC`- $1024\times 1024$) 
  - 超过 **1M** ，直接 `newLen + SDS_MAX_PREALLOC`
  - 否则 `newLen = newLen * 2`

```c
sds sdsMakeRoomFor(sds s, size_t addlen) {

    struct sdshdr *sh, *newsh;

    // 获取 s 目前的空余空间长度
    size_t free = sdsavail(s);

    size_t len, newlen;

    // s 目前的空余空间已经足够，无须再进行扩展，直接返回
    if (free >= addlen) return s;

    // 获取 s 目前已占用空间的长度
    len = sdslen(s);
    sh = (void*) (s-(sizeof(struct sdshdr)));

    // s 最少需要的长度
    newlen = (len+addlen);

    // 根据新长度，为 s 分配新空间所需的大小
    if (newlen < SDS_MAX_PREALLOC)
        // 如果新长度小于 SDS_MAX_PREALLOC 
        // 那么为它分配两倍于所需长度的空间
        newlen *= 2;
    else
        // 否则，分配长度为目前长度加上 SDS_MAX_PREALLOC
        newlen += SDS_MAX_PREALLOC;
    // T = O(N)
    newsh = zrealloc(sh, sizeof(struct sdshdr)+newlen+1);

    // 内存不足，分配失败，返回
    if (newsh == NULL) return NULL;

    // 更新 sds 的空余长度
    newsh->free = newlen - len;

    // 返回 sds
    return newsh->buf;
}
```

#### 空间懒释放策略

`sdstrim(sds s, const char*)` 会削减掉 **s** 两边的字符

去掉之后，我们不改变 **len** , 而是作为 **free space** 进行了保留

#### 二进制安全

对于普通的 **C** 字符串，由于是按照空串 `\0` 来作为结束标志

而对于 **SDS** ，它使用的 **len** 字段就可以避免这一点



### 链表

基本的数据结构——双向链表

```c
/*
 * 双端链表节点
 */
typedef struct listNode {

    // 前置节点
    struct listNode *prev;

    // 后置节点
    struct listNode *next;

    // 节点的值
    void *value;

} listNode;

/*
 * 双端链表迭代器
 */
typedef struct listIter {

    // **当前**迭代到的节点
    listNode *next;

    // 迭代的方向
    int direction;

} listIter;

/*
 * 双端链表结构
 */
typedef struct list {

    // 表头节点
    listNode *head;

    // 表尾节点
    listNode *tail;

    // 节点值复制函数
    void *(*dup)(void *ptr);

    // 节点值释放函数
    void (*free)(void *ptr);

    // 节点值对比函数
    int (*match)(void *ptr, void *key);

    // 链表所包含的节点数量
    unsigned long len;

} list;

```

可以看出，**Redis**中的list采取了双端链表来进行实现。结构体内部包含了：

- 头结点、尾结点
- 链表长度
- 三个支持多态的函数指针



### 字典 hash

Redis中称作 **字典**。它的实现上都是采取了 **链地址法** 的哈希表结构。

> *redis* 后续还引入了 *zipmap* 来作为 `字符串到字符串`的小*hash* 底层数据结构

### 主要数据结构

```C
/*
 * 哈希表节点
 */
typedef struct dictEntry {
    
    // 键
    void *key;

    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;

    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;

} dictEntry;


/*
 * 哈希表
 *
 * 每个字典都使用**两个**哈希表，从而实现渐进式 rehash 。
 */
typedef struct dictht {
    
    // 哈希表数组
    dictEntry **table;

    // 哈希表大小
    unsigned long size;
    
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;

    // 该哈希表已有节点的数量
    unsigned long used;

} dictht;
//=======================================
/*
 * 字典
 */
typedef struct dict {

    // 类型特定函数
    dictType *type;

    // 私有数据
    void *privdata;

    // 哈希表
    dictht ht[2];

    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */

    // 目前正在运行的安全迭代器的数量
    int iterators; /* number of iterators currently running */

} dict;
```

可以看出，**Entry** 作为节点，维护了 `K,V` 关系；哈希表维护了基本的**哈希表大小**和**已占用Hash数量**。

#### 哈希算法

根据 `hashFunc(key)` 可以得到哈希值 **hash** 。但是这个值往往很大，需要规整到 `[0,size-1]` 范围内，所以我们使用 `hash & mask` 来进行 **模计算**。这里的 **mask** 为 `size - 1` 

##### 再哈希

> 大多数的哈希表实现思路类似——**size**都需要是 **2的幂次**，便于进行和掩码的与运算

`dict` 结构中，采用了 `dictht ht[2];` 两个哈希表来进行。一般 `ht[0]` 存储数据，当要进行再哈希的时候，先给 `ht[1]` 分配一定的空间，随后把 `ht[0] ` 的数据再哈希到 `ht[1]` 当中。完成之后，释放 `ht[0]` 空间，调换两个指针 (类似*JVM* *survivor0，1* 的拷贝)

-  如果进行的是扩展操作，那么*rehash* 之后的大小是 大于等于 `ht[0].used * 2` 的第一个 $2^n$  (和 *Java*的实现有点区别，*redis* 这里是针对已经使用的大小乘以二，然后再找到不小于这个数的第一个二的次幂)
- 如果是伸缩操作，那么 *rehash* 之后的大小是 大于等于  `ht[0].used` 的第一个 $2^n$ 

随后，我们进行一个再哈希 (也就是根据新的大小重新分配 `K,V` ) ，放置到 `ht[1]`。随后互换两者指针即可。

##### 渐进式再哈希

再哈希时的数据拷贝工作是最耗时的。Redis 采用 **rehashidx** 来进行渐进式的处理。

- 初始值设置为0，表示再哈希开始
- 每一次对哈希表的增删改查，都会随即触发再哈希。
  - 仅仅再哈希 **rehashidx** 索引对应的节点
  - 此时的增删改查涉及到两个哈希表
- 完成所有的再哈希之后，设置为 -1，表示完成

> 通过将再哈希的行为，均摊到增删改查当中，避免了集中式的再哈希操作

### 集合

#### 跳表——有序集合key的底层实现

> 使用于有序集合元素数量大，或者元素成员是字符串类型
>
> 跳表还使用在了集群节点中的内部数据结构

##### 跳表节点定义

每一个节点内部，除了基本的数据 `robj` ，还包含了后退指针，以及一个 `level` 数组

```c
typedef struct zskiplistNode {

    // 成员对象
    robj *obj;

    // 分值
    double score;

    // 后退指针
    struct zskiplistNode *backward;

    // 层
    struct zskiplistLevel {

        // 前进指针
        struct zskiplistNode *forward;

        // 跨度
        unsigned int span;

    } level[];

} zskiplistNode;
```

##### 跳表定义

```c
typedef struct zskiplist {

    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;

    // 表中节点的数量
    unsigned long length;

    // 表中层数最大的节点的层数
    int level;

} zskiplist;
```

几个要点：

- `level` 表示层，当有新的跳跃表节点 *insert* ，**level** 将会是 `[1,32]`之间的一个随机值
  - 层的 **跨度** `level[i].span`，主要用于计算 **rank**。
  - 对于某一个需要查询的节点，头结点到它的**跨度累积值**就是它的 **rank**
- 前进指针和后退指针都是用于 **遍历**
- 成员和分值
  - 跳表内部按照分值由小到大来进行组织——从这一点上看，分值大的一般 **rank** 也大
  - 分值可以重复，成员不可以

*redis* 中的 **有序集合 zset** 使用了一个跳表 + 一个字典来进行实现。通过跳表来进行 *rank* 的从小到大排序，然后通过字典来实现对象到分值的一个映射。不会产生额外的数据空间浪费，并且能够让 **遍历** 和 **获取对象分值** 都能够有一个比较小的时间复杂度

#### Intset 整数集合

> 用于集合键的底层实现之一，如果集合只包含整数，并且数量不多，就采用整数集合来进行实现

##### 数据结构

```C
typedef struct intset {
    
    // 编码方式
    uint32_t encoding;

    // 集合包含的元素数量
    uint32_t length;

    // 保存元素的数组, 按照升序进行排列
    int8_t contents[];

} intset;
```

关注：

- 编码方式决定了 **contents[]** 数组的元素的大小
- 插入操作：
  - 为了维护 **升序**关系，插入的时间复杂度是 $O(N)$ ——这里其实可以优化
  - 若出现了大小超过编码的，需要进行 **升级**
  - 不支持 **降级**



### Ziplist 压缩列表

> 可以用于基本的列表数据结构；也可以用于哈希字典 (键和值相邻排放) ，同时也是有序集合的底层实现之一。一般用于存储少量的列表项，并且列表项是一些小整数或小字符串

