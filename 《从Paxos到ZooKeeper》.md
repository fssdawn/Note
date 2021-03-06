### 2.2.3 Paxos算法详解

假设有一组可以提出提案的进程集合，那么对于一个一致性算法来说需要保证以下几点：

- 在这些被提出的提案中，只有一个会被选定。
- 如果没有填被提出，那么就不会有被选定的提案。

- 当一个填被选定后，进程应该可以获取被选定的提案信息。

对于一致性来说，安全性（Safety）需求如下：

- 只有被提出的提案才能被选定。
- 只有一个值被选定。
- 如果某么进程认为某个提案被选定了，那么这个提案必须是真的被选定的那个。

**提案的选定**

大多数。

**推导过程**

P1：一个Accptor必须批准它收到的第一个提案

P1a：一个Acceptor只要尚未响应过任何编号大于Mn的Prepare请求，那么它就可以接收这个编号为Mn的提案。（优化：可以忽略已批准过的提案的Prepare请求）

P2：如果编号为M0，Value值为V0的提案（即[M0,V0]）被选定了，那么所有比编号M0更高的，且被选定的提案，其Value值必须为V0。

P2a：如果编号为M0，Value值为V0的提案（即[M0,V0]）被选定了，那么所有比编号M0更高的，且被Acceptor批准的提案，其Value值必须为V0。

P2b：如果一个提案[M0,V0]被选定后，那么之后任何Proposer产生的编号更高的提案，其Value值都为V0。

P2c：对于任意的Mn和Vn，如果提案[Mn,Vn]被提出，那么肯定存在一个由半数以上的Acceptor组成的集合S，满足以下两个条件中的任意一个：

- S中不存在任何批准过编号小于Mn的提案的Acceptor。
- 选取S中所有Acceptor批准的编号小于Mn的提案，其中编号最大的那个提案其Value值是Vn。

推导过程为第二数学归纳法。略

**Proposer生成提案**

对于Proposer，获取被通过的提案比预测可能会被通过的提案简单。

1. Proposer选择一个新的提案编号Mn，然后向某个Acceptor集合的成员发送请求，要求该集合中的Acceptor做出如下回应。
	- 像Proposer承诺，保证不再批准任何编号小于Mn的提案。
	- 如果Acceptor已经批准过任何提案，那么其就向Proposer反馈当前该Acceptor已经批准的编号小于Mn但为最大编号的那个提案的值。
2. 如果Proposer收到了来自半数以上的Acceptor的响应结果，那么它就可以产生编号为Mn、Value值为Vn的提案，这里的Vn是所有响应中编号最大的提案Value值。当然还存在一种情况，就是半数以上的Acceptor都没有批准过任何提案，即响应中不包含任何的提案，那么此时Vn值就可以由Proposer任意选择。

**Acceptor批准提案**

一个Acceptor可能会收到来自Proposer的两种请求，分别是Prepare请求和Accept请求，分别相应如下：

- Prepare请求：Acceptor可以在任何时候响应一个Prepare请求。
- Accept请求：在不违背Accept现有承诺的前提下，可以任意响应Accept请求。

**算法陈述**

阶段一：

1. Proposer发送提案编号Mn；
2. Acceptor根据约束接收提案，如果接收过返回接收最大值Vn；

阶段二：

1. 如果Proposer收到大多数A的响应，发送[Mn,Vn]；
2. Acceptor根据约束接收提案；

**提案的获取**

1. 通知全部Learner

2. 选取主Learner
3. 将主Learner改为Learner集合

**通过选取主Proposer保证算法的活性**

# 三、Paxos的工程实践

## 3.1 Chubby

一个分布式锁服务。解决分布式协作，元数据存储，Master选举等一系列与分布式锁服务相关的问题。

底层为Paxos算法。

### 3.1.1 概述

