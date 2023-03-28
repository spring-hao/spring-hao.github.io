---
title: Zookeeper
date: 2022-11-05 22:14:35.449
updated: 2022-11-22 15:28:26.662
url: /archives/zookeeper
categories: 
tags: 
---

# 基础
Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应。

## 特点
1. 一个领导者,多个跟随着组成的集群
2. 集群中 只要有**半数以上**节点存活,集群就可以正常提供服务.
3. 全局数据一致:每个Server保存一份相同的数据副本,Client无论连接到哪个Server,数据都是一致的
4. 更新请求顺序执行,来自同一个Client的更新请求按照发送顺序依次执行
5. 数据更新原子性,一次数据更新要么成功,要么失败
6. 实时性,在一定时间范围内,Client能读到最新数据

提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等。
## 数据结构
数据模型的结构与文件系统类似,整体上来看是一棵树,每个节点叫做一个ZNode.每个ZNode默认能够**存储1MB数据**,每个ZNode可以通过其**路径唯一标识**.

**结点状态stat的属性**
- `cZxid`数据结点创建时的事务ID——针对于`zookeeper`数据结点的管理：我们对结点数据的一些写操作都会导致`zookeeper`自动地为我们去开启一个事务，并且自动地去为每一个事务维护一个事务`ID`
- `ctime`数据结点创建时的时间
- `mZxid`数据结点最后一次更新时的事务ID
- `mtime`数据结点最后一次更新时的时间
- `pZxid`数据节点最后一次修改此`znode`子节点更改的`zxid`
- `cversion`子结点的更改次数
- `dataVersion`结点数据的更改次数
- `aclVersion`结点的ACL更改次数——类似`linux`的权限列表，维护的是当前结点的权限列表被修改的次数
- `ephemeralOwner`如果结点是临时结点，则表示创建该结点的会话的`SessionID`；如果是持久结点，该属性值为0
- `dataLength`数据内容的长度
- `numChildren`数据结点当前的子结点个数

**结点类型**
- 临时节点：该节点的生命周期依赖于创建它们的会话。一旦会话( `Session`）结束，临时节点将被自动删除，当然可以也可以手动删除。虽然每个临时的 `Znode`都会绑定到一个客户端会话，但他们对所有的客户端还是可见的。另外，`Zookeeper`的**临时节点不允许拥有子节点**
- 持久化结点：该结点的生命周期不依赖于会话，并且只有在客户端显示执行删除操作的时候，它们才能被删除
- 持久化顺序编号节点：创建节点时设置顺序标识,顺序号是一个单调递增的计数器,由父节点维护
- 临时顺序编号节点： 客户端与 Zookeeper断开连接后 ，该 节 点 被 删 除 ，只是Zookeeper给该节点名称进行顺序编号

## 选举机制
- SID：服务器ID。用来唯一标识一台ZooKeeper集群中的机器，每台机器不能重复，和myid一致。
- ZXID：事务ID。ZXID是一个事务ID，用来标识一次服务器状态的变更。在某一时刻，集群中的每台机器的ZXID值不一定完全一致，这和ZooKeeper服务器对于客户端“更新请求”的处理逻辑有关。
- Epoch：每个Leader任期的代号。没有Leader时同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加

**选举Leader规则**：
1. EPOCH大的直接胜出
2. EPOCH相同，事务id大的胜出
3. 事务id相同，服务器id大的胜出

# 分布式算法
## Paxos 算法
Paxos算法：一种基于消息传递且具有高度容错特性的一致性算法。
Paxos算法解决的问题：就是如何快速正确的在一个分布式系统中对某个数据值达成一致，并且保证不论发生任何异常，都不会破坏整个系统的一致性。

Paxos算法描述：
1. 在一个Paxos系统中，首先将所有节点划分为Proposer（提议者），Acceptor（接受者），和Learner（学习者）。（每个节点都可以身兼数职）。
2. 一个完整的Paxos算法流程分为三个阶段：
3. Prepare准备阶段
	- Proposer向多个Acceptor发出Propose请求Promise（承诺）
	- Acceptor针对收到的Propose请求进行Promise（承诺）
4. Accept接受阶段
	- Proposer收到多数Acceptor承诺的Promise后，向Acceptor发出Propose请求
	- Acceptor针对收到的Propose请求进行Accept处理
5. Learn学习阶段：Proposer将形成的决议发送给所有Learners

Paxos 算法缺陷：在网络复杂的情况下，一个应用 Paxos 算法的分布式系统，可能很久无法收敛，甚至陷入**活锁**的情况。

## ZAB 协议
Zookeeper 设计为只有一台客户端（Leader）负责处理外部的写事务请求，然后Leader 客户端将数据同步到其他 Follower 节点。即 Zookeeper 只有一个 Leader 可以发起提案。Zab 协议包括两种基本的模式：消息广播、崩溃恢复。

ZAB协议定义的四种节点状态:
1. Looking：选举状态
2. Following：从节点所处的状态
3. Leading：主节点所处状态
4. Observing：观察者节点所处状态

### 消息广播
ZAB协议针对事务请求的处理过程类似于一个**两阶段提交**过程
1. 广播事务阶段
2. 广播提交操作

这两阶段提交模型有可能因为Leader宕机带来数据不一致

1. 客户端发起一个写操作请求。
2. Leader服务器将客户端的请求转化为事务Proposal提案，同时为每个Proposal 分配一个全局的ID，即zxid。
3. Leader服务器为每个Follower服务器分配一个单独的队列，然后将需要广播的 Proposal依次放到队列中去，并且根据FIFO策略进行消息发送。
4. Follower接收到Proposal后，会首先将其以事务日志的方式写入本地磁盘中，写入成功后向Leader反馈一个Ack响应消息。
5. Leader接收到超过**半数以上**Follower的Ack响应消息后，即认为消息发送成功，可以发送commit消息。
6. Leader向所有Follower广播commit消息，同时自身也会完成事务提交。收到commit的服务器把本地数据文件中的数据写到内存中。
7. Zookeeper采用Zab协议的核心，就是只要有一台服务器提交了Proposal，就要确保所有的服务器最终都能正确提交Proposal。

### 崩溃恢复
一旦Leader服务器出现崩溃或者由于网络原因导致Leader服务器失去了与过半 Follower的联系，那么就会进入崩溃恢复模式。

Zab协议崩溃恢复要求满足以下两个要求：
1. 确保已经被Leader提交的提案Proposal，必须最终被所有的Follower服务器提交。 （**已经产生的提案，Follower必须执行**）
2. 确保丢弃已经被Leader提出的，但是没有被提交的Proposal。（**丢弃胎死腹中的提案**）

崩溃恢复主要包括两部分：Leader选举和数据恢复。
#### Leader选举
Leader建立完成后，Leader周期性地向Follower发送心跳，当Leader崩溃后，Follower发现socket通道关闭，于是Follower开始进入到Looking状态，重新回到Leader选举状态，**此时集群不能对外提供服务**

根据上述要求，Zab协议需要保证选举出来的Leader需要满足以下条件：
1. 新选举出来的Leader不能包含未提交的Proposal。即新Leader必须都是已经提交了Proposal的Follower服务器节点。
2. 新选举的Leader节点中含有最大的zxid。这样做的好处是可以避免Leader服务器检查Proposal的提交和丢弃工作。

#### 数据恢复
1. 完成Leader选举后，在正式开始工作之前（接收事务请求，然后提出新的Proposal），Leader服务器会首先确认事务日志中的所有的Proposal 是否已经被集群中过半的服务器Commit。
2. Leader服务器需要确保所有的Follower服务器能够接收到每一条事务的Proposal，并且能将所有已经提交的事务Proposal应用到内存数据中。等到Follower将所有尚未同步的事务Proposal都从Leader服务器上同步过，并且应用到内存数据中以后，Leader才会把该Follower加入到真正可用的Follower列表中。
