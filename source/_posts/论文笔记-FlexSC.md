---
title: 论文笔记-FlexSC
date: 2020-12-03 10:35:50
tags:
cover: https://images.unsplash.com/photo-1601758282715-6446bbde7082?ixid=MXwxMjA3fDF8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1253&q=80
description: FlexSC
keywords:
---

### 1. 概述

系统调用采用同步的机制，在用户态转移到内核态的过程中，需要把当前上下文保存，但是其中的模式切换成本，与间接的处理器上下文数据污染（影响局部性 *locality*），会对系统调用造成比较大的负面影响

![image.png](https://i.loli.net/2020/12/20/5jrZnDoWebTua9s.png)

文中给出的 *proposal* : ***exception-less system call***，在请求内核服务时，不需要使用同步处理器异常。实现过程中，(*invocation stage*)系统调用通过把内核请求，以普通的内存存储操作写入到一个预留的系统调用页 (*syscall page*)。(*execution stage*)随后的系统调用执行采用 **异步** 的方式 (*special in-kernel **syscall threads***) ，将结果异步载入 *syscall page*

从上可以看出，它其实是把 **invocation** 和 **execution** 解耦。一方面可以在执行流上保证<u>时间的局部性</u>；另一方面能够让 *syscall* 独立运行在一个core上，与 *user mode threads* 隔离，保证每一个core内的<u>空间局部性</u>

与此同时，有一个很关键的 **动态核特化** (*dynamic core specialization*)概念——也就是core可以根据当前OS的*workload*，动态调整某一个core是进行用户态 / 内核态的工作。为了减轻编程负担，给出了一个 ***M-on-N*** 的线程模型(*M*个用户态线程，执行于 *N* 个内核可见的线程) 。关于POSIX线程模型，参见[POSIX线程模型](https://www.yuque.com/barret/giv6pv/ogmd8f)



### 2. cost of system-call

#### 2.1 模式切换cost

时间损耗发生于：用户态下必要的syscall调用、执行权转交给内核的执行、执行权反交给用户态。现代处理器通过处理器异常来做这个模式切换，切换时会将用户模式下的指令流水线清空、将CPU上下文保存入栈、改变 *protection domain*、重定向到异常处理*handler*

> 这里对 *protection domain* 不是特别理解，详见 [Lampson: Protection Models](https://zhuanlan.zhihu.com/p/59796446)。可以和chcore中的pmo-cap关联起来看，个人理解为一个资源的句柄域，由于内核态和用户态的 *domain* 不同(可以使用的资源范围)，故需要在这里进行*domain* 的改变



#### 2.2 系统调用cost

处理器状态污染：

进入内核态执行时，处理器的L1缓存数据、指令缓存、TLB、分支预测表，以及L2缓存，都会因为系统调用而被污染，从而在返回用户态之后，极大地影响用户态程序执行速度

![image.png](https://i.loli.net/2020/12/20/vfRDmGitJOsWB6F.png)



#### 2.3 返回用户态执行cost

> 衡量指标：IPC (*instructions per cycle*)

理想状态下，用户模式下的*IPC* 应当不变化。但是实际实验中，有如下的几个影响因素。

**直接影响**：系统调用关联的异常，将处理器执行流水线清空

**间接影响**：处理器状态污染



<img src="https://i.loli.net/2020/12/20/vFbzLq7xU2ZRiAH.png" alt="image.png" style="zoom: 67%;" />

实验结果解读：

1. 横轴表示syscall的一个频率，纵轴表示 $1-\frac{IPC_{user}}{IPC_{base}}$。从这里可以看出，在per-1K条指令一次系统调用的情况下，会造成比较大的损耗。
2. 使用空的syscall来评估系统调用带来的 *direct* 影响，使用一个 *pwrite* 调用来评估系统调用带来的 *indirect* 影响。可以看出，在syscall频繁的时候(1K)，直接影响占据主导；而当syscall不频繁时，间接影响占据主导



 IPC受syscall频率的影响。可以见到，当系统调用频繁时，内核的状态也能够比较好地被保持。由此看出syscall对 user / kernel mode均会造成局部性的影响

<img src="https://i.loli.net/2020/12/20/qDFpQv7kRSaLEnd.png" alt="image.png" style="zoom:67%;" />



### 3. Exception-Less system call

> 提供灵活的系统调用执行调度，进而提升用户 / 内核态下的局部性

在两个方面下功夫：

- *System call batching* : 延缓systemcall的执行，让他们以一个batch来一起执行
- *Core specialization* : 内核特化，*execution* / *invoke* 分开于不同的core上执行

#### 3.1 Exception-Less syscall接口

通过一个 user-kernel共享的内存页支持(*syscall page*)，包含了syscall入口，关联到请求状态、系统调用号、入参、出参

在调用一个exception-less syscall，用户态的线程会找到 *syscall page*内的一个空闲*entry*，将系统调用相关的数据写入。之后用户态线程可以继续执行，不被中断。用户线程后续会读取*entry*中的执行状态，校验系统调用是否已经完成。这些操作中，用户线程不需要通过*exception* 

*syscall page* 中的每一个*entry* 结构如下：

> 64-bit系统中，每一个 *entry* 占用 **64 bytes**

![image.png](https://i.loli.net/2020/12/20/Nv5WgtnKT8qSreu.png)

在调用 *exception-less syscall* 时，用户线程需要找到一个 *status = free* 的 *entry*，随后将必要的参数写入到这个入口。写入完成后，*status* 置为 *submitted* 。随后用户线程可以执行自己的代码，后续需要check这个*status* 是否为 *done*，表明系统调用是否完成。若完成，那么用户线程设置 *status* 为 *free*

> ==Question== : 这里会不会有并发问题？ 当一个线程正在write argument，另一个用户线程发现 *status = free*，那么也会进行write argument。这时候会有race ？



#### 3.2 执行、调用解耦

不同于 *exception-based syscall* ，*exception-less syscall* 采用一种 **异步** 的策略，调用时，用户线程写入 *syscall page*；执行*syscall* 时，***syscall thread*** (必在内核态) 从 *syscall page* 拉取系统调用请求

<img src="https://i.loli.net/2020/12/20/2F4gn3JqKUo1pGX.png" alt="image.png" style="zoom:80%;" />

- 用户线程只在无法进行更多的执行，才会唤醒系统调用线程(*syscall thread*) ——保证时间局部性
- 系统调用线程可以被调度——保证空间局部性



#### 3.3 FlexSC

> *exception-less syscall* 的具体实现实践

给出两个新的linux系统调用：

- **flexsc_register()** 

在进程初始化的时候**显式**调用，是 *exception-based* 的。由于只需要执行一次，对性能的影响可以忽略

这个系统调用做了两件事：将若干 *syscall pages* 映射至用户虚存空间、为每一个 *entry* 产生一个 *syscall thread*



- **flexsc_wait()**

对于传统的同步系统调用，用户线程只需要简单地进行 *sleep* ，等待*syscall* 执行完毕。当前的执行模式中，用户线程会显示地与内核 *communicate*，告知自己无法进行进一步的执行，直到调用的*syscall* 已经完成。这一个函数调用是 *exception-based* ，调用过后能够在至少一个syscall完成后，唤醒用户空间的线程



#### 3.4 syscall thread

之前说到，**syscall thread** 从 *syscall pages* 拉取请求。

在 *flexsc_register()* 阶段，*syscall thread* 从注册的进程中克隆得到，他们也就拥有相同的虚拟地址空间。这一点能够让后续的数据迁移变得容易(不需要重新定义虚存映射)

针对每一个entry，都会在kernel mode创建一个 *syscall thread*。但是只有一个 *syscall thread* (per core) 是处于active状态。当系统调用线程需要立即阻塞时，FlexSC会notify工作队列，使另一个线程wake up，立即开始执行下一个 *syscall*。 当资源空闲时，当前的Linux代码会唤醒等待的线程，并恢复其执行



#### 3.5 syscall thread调度器

单核：

调度器假设用户空间会尝试调用尽可能多的*syscall*，调度器随后会唤醒其中的一个空闲 *syscall thread*，用来执行第一个 *syscall*。

- 如果这个系统调用不会被阻塞，那么当前的 *syscall thread* 会继续执行下一个 *syscall*
- 如果会被阻塞，那么阻塞时，调度器会唤醒下一个 *syscall thread*，来执行下一个系统调用

调度器不会唤醒用户线程，直到所有的系统调用已经被 *issued*，



多核：

通过尝试使用预定义的静态core list调度。

- 选择*list*中的第一个核，如果某个进程的系统调用线程当前正在该核上运行，则选择列表中下一个核
- 如果所core当前未执行系统调用线程，则将处理器间中断发送到core，以表明它必须唤醒系统调用线程

每一个core上只能同时运行一个进程的*syscall thread*，但是之前的问题也提到了一个并发竞争问题。这里通过对 page 加锁来避免并发的问题，直到所有 *submitted* 系统调用请求都被发出了才进行解锁





### 4. FlexSC - Threads

*Exception-less syscall* 更加类似于 **事件驱动** 模型，能够让IO相关的操作在未来执行，并且能够请求、验证完成、处理任何系统调用

文中着重探讨了实现 ***FlexSC-Threads***，一个 $M-on-N$ 线程模型。依赖于仅在用户空间中进行线程切换的能力，来让同步系统调用透明地转变为 *exception-less syscall*。



*FlexSC-Threads* 工作流程如下：

1. 将每个libc调用重定向到我们的库。 通常应用程序不直接嵌入代码以发出系统调用，而是在**动态加载**的libc中调用包装器，故采用Linux的动态加载功能将此类调用的执行重定向到我们的库
2. 随后将对应的 *exception-less syscall* 写入到 *syscall page*，切换至另一个就绪的用户线程
3. 如果没有就绪用户线程，那么*FlexSC*检查那些 *status = completed* 的 *syscall page*，将对应的线程唤醒，来获取系统调用完成后的*result*
4. 当所有的用户线程都等待系统调用，FlexSC Thread lib将调用**flexsc_wait()**，使内核可见线程进入睡眠状态，直到一个挂起的系统调用完成。

多核环境下的每一个进程，为每一个core建立一个内核可见线程

























