---
title: Java集合
date: 2020-03-14 17:26:21
tags: 笔记
cover_img: https://images.unsplash.com/photo-1496507161348-aeec0403f141?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
feature_img: https://images.unsplash.com/photo-1517958911667-09c05f6cd698?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=634&q=80
description:
keywords: 笔记
---

## Java 集合类整理

> *JDK* 版本 **1.8**
>
> 本文章就最近看的一些jdk源码来进行总结，可能会比较简洁

### 1. List 大类

主要包含了 **ArrayList , Vector , LinkedList**

#### 1.1 ArrayList

> 基本思想：
>
> - 默认大小为 10 ，初始化的时候不进行对象数组空间分配，直到第一次 `add(E e)`
> - 缓冲区满才进行扩容。扩容策略采取 *OldCapacity* * 1.5
> - `remove` 不进行容量调整 (会有浪费)

关键代码部分 `grow(int minCapacity)`

```java
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
}
private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
}
```

根据之前计算得到的 `minCapacity` , 进行 `ArrayList` 的扩容操作。如果计算得到的容量大于了 `MAX_ARRAY_SIZE = INT_MAX - 8` ，那么会直接扩容到 `Integer.MAX_VALUE` 

> 这里的 `MAX_ARRAY_SIZE` 设置为 `INT_MAX - 8` 主要是为了考虑到，有一些*JVM* 的对象数组 (这里是 `Object[] elementData`) 会采取一定长度的 *header* 。如果直接设置为 `INT_MAX` 可能会引发 OOM

此外，可以看出 `ArrayList` 扩容其实是采取了复制的方法，将原来空间的所有数据放到了另一块空间。这种做法其实在复制上消耗特别大

#### 1.2 Vector

> 基本思想：
>
> - 和 `ArrayList` 颇为类似，它也是以10作为初始容量，初始化不分配对象数组的空间
> - 扩容采取  *OldCapacity* * 2
> - `remove` 不进行容量调整
> - 线程安全 `synchronized`

核心代码如下：

```java
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
}
```

> 这里的 `capacityIncrement` 是在初始化的时候可以额外指定的变量。如果指定了，那么就不进行乘以二扩容，而是每一次增加这个 `capacityIncrement` 变量

#### 1.3 LinkedList

> - 双向链表
> - 内部额外存储头结点和尾结点

### 2. KV 大类

#### 2.1 HashMap

> 面试里面最喜欢问的了，没有之一
>
> 主要几个流程：
>
> - `put`
> - `get`
> - `remove`
>
> 线程安全问题
>
> - put的时候导致的多线程数据不一致 
>   - 两个线程前后写入，产生数据覆盖
>   - 成环问题
>
> <img src="https://i.loli.net/2020/03/14/2WzkGAQVg9Rnpou.png" alt="image.png" style="zoom:80%;" />

基本数据结构：

- `class Node<K,V>` 是桶节点和链表节点的基本数据结构
- `Node<K,V>[] table` 是哈希表内部的实现，其实就是 **链表数组**

##### 2.1.1 put 添加 K,V 对

1. 计算 `key` 的 `hashcode` 值，将对象的 `hashcode` 进行高16位和低16位的异或操作。可以看到， `HashMap` 支持对 `null` 键的处理

```java
static final int hash(Object key) {
  int h;
	return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

2. `putValue` 主体部分

在之前的 `hash` 获取之后，接下来一步就是进行哈希表的一个处理了，由于细节比较多，我直接在源代码里面加注释来解释了

```java
/**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
      	//如果tab没有初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            //resize进行空间分配.
          	n = (tab = resize()).length;
      	//如果当前桶是空的 , 那么新建节点 , 直接插入
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //当前桶已经有节点 , 需要在后面进行插入
      	else {
            Node<K,V> e; K k;
          	//对于桶的第一个节点 p , 如果插入的点和第一个点 p 一样
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
          	//如果是红黑树 , 就进行红黑树插入
          	else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //如果第一个节点不同 , 并且后面是链表
          	else {
                  //遍历链表
              		for (int binCount = 0; ; ++binCount) {
 										//遇到空节点 , 表示可以插入 (1.7进行头插 , 1.8 进行尾插)         
                    if ((e = p.next) == null) {
                      	//插入到尾端
                        p.next = newNode(hash, key, value, null);
                      	//如果 binCount >= 8 - 1 (链表长度为8) , 就把链表转红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                  	//如果遇到相同的节点
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;	 // p = p.next
                }
            }
          	//如果这个键值对存在 , 就进行覆盖
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
      	//如果map大小大于了阈值 , 再次进行 resize
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

有几个细节要提及：

- `1.8` 中 , 链表和红黑树是交替使用的。
  - 当链表长度达到了 **8** 之后，会转换为红黑树
  - 当 `size` 是 **6** 之后，红黑树退化为链表
  - 原因分析：
    - 红黑树**平均**查询效率 O(logN) , 链表 O(N) / 2。在长度是 **8** 的时候，红黑树时间复杂度为 3 , 链表是 *8 / 2 = 4* 。这时候使用红黑树是效率更高的。而另一边退化的阈值设置为 **6** , 主要是中间有一个 **7** 的 *gap* , 从而避免红黑树和链表直接的频繁切换
    - 当hashCode离散性很好的时候，树型bin用到的概率非常小，因为数据均匀分布在每个bin中，几乎不会有bin中链表长度会达到阈值。但是在随机hashCode下，离散性可能会变差，然而JDK又不能阻止用户实现这种不好的hash算法，因此就可能导致不均匀的数据分布。不过理想情况下随机hashCode算法下所有bin中节点的分布频率会遵循泊松分布，一个bin中链表长度达到8个元素的概率为0.00000006，几乎是不可能事件

这里还有一个重要的调用函数 `resize()` 需要看一下

```java
// Cap 指的是哈希表中 , 链表数组的长度
// Thr 指的是键值对的个数
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
  			//阈值更新 (也就是整个哈希表的可容纳大小)
        if (oldCap > 0) {
          	// 如果旧有oldCap很大 , 就把阈值设置为 Integer.MAX_VALUE
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
          	//DEFAULT_INITIAL_CAPACITY = 16
          	//MAXIMUM_CAPACITY = 1 << 30
          	//如果 oldCap * 2 比 MAXIMUM_CAPACITY 小 , 那么阈值更新到 2 倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
  			// 把新的链表数组长度设置为原来的 KV 对个数
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        // 对于初始化的情况
  			// 数组长度设置 16
  			// 阈值设置 0.75f * 16 = 12
  			else {
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
  			//更新阈值 , 替换原有的哈希表. 数组长度为 newCap
        threshold = newThr;
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
  			//如果不是初始化的情形 , 需要进行rehash
        if (oldTab != null) {
          	//遍历数组每一个桶
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
              	//当前桶非空 , 需要进行rehash
                if ((e = oldTab[j]) != null) {
                  	//原有桶清空
                    oldTab[j] = null;
                  	//如果只有一个节点 , 直接再次计算hash , 填入就可以
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                  	//如果是红黑树 , 那么红黑树的 split
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    //如果是链表
                  	else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                          	//获取next节点
                            next = e.next;
                          	// e 作为前驱节点 , 同时记录下头节点和尾节点
														// 低若干位 , 和原来的一致
                            if ((e.hash & oldCap) == 0) {
																// 插入到 loTail  , 进行尾插
                              	if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                      	//到这里 , 已经形成了单独的链表 
                      	//如果是低位 , 那么直接重新插入到原来的桶
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                      	//插入到高位桶
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

#### 2.2 HashTable

> 基本要点
>
> - 采用链表实现
> - 线程安全 `synchronized`
> - 默认的 Capacity (链表数组长度)是 **11** , 装载因子 0.75f
> - *index* 计算：`index = (hash & 0x7FFFFFFF) % tab.length`
> - 头插
> - rehash的变化为 `newCapacity = (oldCapacity << 1) + 1`
> - KV 不可以为空

#### 2.3 LinkedHashMap

这个类继承自 `HashMap`，内部维护了一个双向链表。这个链表可以决定迭代的遍历顺序

在添加新节点的时候，将新节点链接在**内部双向链表的尾部**。

`accessOrder=true`的模式下,在`afterNodeAccess()`函数中，会将当前被访问到的节点e，移动至内部的双向链表的尾部。

> 重点关注：afterNodeAccess()函数中，会修改modCount,因此当你正在accessOrder=true的模式下,迭代LinkedHashMap时，如果同时查询访问数据，也会导致fail-fast，因为迭代的顺序已经改变。

#### 2.4 TreeMap

> LinkedHashMap保证数据可以保持插入顺序
>
> 而如果我们希望Map可以保持key的大小顺序的时候，我们就需要利用TreeMap了

内部采用了红黑树，并不是基于 **hash** 来进行实现的

### 3. SET 大类

#### 3.1 HashSet

内部实现完全使用了 `HashMap`

#### 3.2 TreeSet

内部实现采用了 `TreeMap` ， 持有 `NavigableMap` 类型的引用

### 4. 线程安全大类

#### 4.1 ConcurrentHashMap

> - *Key , Value* 不可以为空
> - Hash计算如下 `(h ^ (h >>> 16)) & HASH_BITS`
> - 链表 & 红黑树使用。链表进行尾插
> - 链表和红黑树的临界值也是 **8**

主要讲述一下和 *HashMap* 的不同

##### Node定义

在 *ConcurrentHashMap* 中，*Value,next* 都设置为了 `volatile` 内存可见

```java
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
          	//CAS控制
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```

**yield 和 sleep 的异同**

1）yield, sleep 都能暂停当前线程，sleep 可以指定具体休眠的时间，而 yield 则依赖 CPU 的时间片划分。

2）yield, sleep 两个在暂停过程中，如已经持有锁，则都不会释放锁资源。

3）yield 不能被中断，而 sleep 则可以接受中断。

在 *1.8* 之前，采用了 **分段锁** ； 而 *1.8* 中主要使用了 **CAS** 和 **synchronized** 来进行并发控制

**CAS** 没什么能说的，主要看一下分段锁 *Segment* 

这是一种 *ReentrantLock*，段的结构和 *HashMap* 类似，也是数组 + 链表。每一个段包含了 *HashEntry* 数组。当要对这个数组进行修改的时候，必须要先获得对应的锁(*Get*不需要获取锁，因为共享变量设置为了 `volatile`，除非读到的值是空的才会加锁重读)

*volatile* 底层使用了**内存屏障**来加以完成，实现对内存操作的顺序控制。

> *ReentrantLock* 可重入锁，表示已经获取到这个资源的线程，可以再次进入。*Synchronized* 也是可以重入的。每一个线程进入一次，那么锁计数器+1.直到计数器为0的时候才释放
>
> - *ReentrantLock* 采用 **JDK** 实现；*Synchronized*则是 **JVM** 实现的
> - *Synchronized* 底层使用监视器（管程）实现，管程的本质就是操作系统的 *Mutex Lock*，需要牵涉到用户态和核心态的切换。 优化之前，性能比可重入锁差；
> - *1.6* 之后引入了*Synchronized* 轻量级锁和偏向锁，也是默认开启的。此外还有自适应自旋锁的优化

