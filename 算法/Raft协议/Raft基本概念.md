### 为什么会有 Raft？

在分布式系统中，为了消除单点提高系统可用性，通常会使用副本来进行容错，但这会带来另一个问题，即如何保证多个副本之间的一致性？

所谓的一致性并不是指集群中所有节点在任一时刻的状态必须完全一致，而是指一个目标，即让一个分布式系统看起来只有一个数据副本，并且读写操作都是原子的。（强一致性）

共识算法（Consensus Algorithm）就是用来做这个事情的，它保证即使在小部分（≤ (N-1)/2）节点故障的情况下，系统仍然能正常对外提供服务。共识算法通常基于状态复制机（Replicated State Machine）模型，也就是所有节点从同一个 state 出发，经过同样的操作 log，最终达到一致的 state。

共识算法是构建强一致性分布式系统的基石，Paxos 是共识算法的代表，而 Raft 则是其作者在博士期间研究 Paxos 时提出的一个变种，主要优点是容易理解、易于实现。



### 基本概念

Raft 核心算法将一致性问题拆分为三个子问题。

- **Leader election**
  - 集群中必须存在一个 leader 节点。
- **Log replication**
  - Leader 节点负责接收客户端请求，并将请求操作序列化成日志同步。
- **Safety**
  - 包括 leader 选举限制、日志提交限制等一系列措施，来确保 state machine safety。

除核心算法外还有集群成员变更、日志压缩等。



### 选举相关概念

每开始一次新的选举，称为一个 term，每个 term 都有一个严格递增的整数与之关联。

节点的状态切换如图所示: 

<div align=middle><img src=".images/d9f2d1f2d3674a50900eda504ecaa326~tplv-k3u1fbpfcp-watermark.image" width="70%" height="70%" /></div>

具体说明如下：

- **Starts up**
  - 节点刚启动时自动进入 follower 状态。
- **Times out, starts election**
  - 进去 follower 状态后开启一个选举定时器，到期时切换为 candidate 并发起选举，leader 节点的心跳会部分重置这个定时器。
- **Times out, new election**
  - 进入 candidate 状态后开启一个超时定时器，如果到期时还未选出新的 leader，就保持 candidate 状态并重新开始下一次选举。
- **Receives votes from majority of servers**
  - Candidate 状态节点收到半数以上选票，切换状态成为新的 leader。
- **Discovers current leader or new term**
  - Candidate 状态节点收到 leader 或更高 term 号（本文后续有介绍）的消息，表示已经有 leader 了，切回 follower。
- **Discovers server with higher term**
  - Leader 节点收到更高 term 号的消息，表示已经存在新 leader 了，切回 follower。这种切换一般发生在网络分区时，比如旧 leader 宕机后恢复。



### Term相关概念

每当 candidate 触发 leader election 时都会增加 term，如果一个 candidate 赢得选举，他将在本 term 中担任 leader 的角色，但并不是每个 term 都一定对应一个 leader，比如上述的 “times out, new election” 情况（对应下图中的t3），可能在选举超时时都没有产生一个新的 leader，此时将递增 term 号并开始一次新的选举。

<div align=middle><img src=".images/f3e463285d0e4d6299fca19c7ef7e334~tplv-k3u1fbpfcp-watermark.image" width="60%" height="60%" /></div>



节点间通过 RPC 来通信，主要有两类 RPC 请求:

**RequestVote RPCs:** 用于 candidate 拉票选举

**AppendEntries RPCs:** 用于 leader 向其它节点复制日志以及同步心跳

