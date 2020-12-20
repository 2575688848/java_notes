### Follower状态转移

集群刚开始启动时，都是处于 follower 身份，每个 follower 有一个随机的选举超时时间，如果过了这个时间还没有找到主节点，那么就会转变为 Candidate 身份，并且自己当前的 term 加 1 ，发起新一轮的选举。然后它会向集群其它节点发送“请给自己投票”的消息（RequestVote RPC）。



### Candicate状态转移 

Follower 切换为 candidate 并向集群其他节点发送“请给自己投票”的消息后，接下来会有三种可能的结果。

**1. 选举成功**

当candicate从整个集群的**大多数**（N/2+1）节点获得了针对同一 term 的选票时，它就赢得了这次选举，立刻将自己的身份转变为 leader 并开始向其它节点发送心跳来维持自己的权威。

每个节点针对每个 term 只能投出一张票，并且按照先到先得的原则。这个规则确保只有一个 candidate 会成为 leader。



**2. 选举失败**

Candidate 在等待投票回复的时候，可能会突然收到其它自称是 leader 的节点发送的心跳包（AppendEntries RPC），如果这个心跳包里携带的 term **不小于** candidate 当前的 term，那么 candidate 会承认这个 leader，并将身份切回 follower。这说明其它节点已经成功赢得了选举，只需立刻跟随即可。但如果心跳包中的 term 比自己小，candidate 会拒绝这次请求并保持选举状态。



*S4、S2 依次开始选举*

<div align=middle><img src=".images/d4ce0c7fdf3b4039a0e4b0b200af731a~tplv-k3u1fbpfcp-watermark.image" width="40%" height="40%" /></div>



*S4 成为 leader，S2 在收到 S4 的心跳包后，由于 term 不小于自己当前的 term，因此会立刻切为 follower 跟随S4*

<div align=middle><img src=".images/2535cfeb9cfb44d4a0a04506f09f7485~tplv-k3u1fbpfcp-watermark.image" width="40%" height="40%" /></div>

**3. 选举超时**

第三种可能的结果是 candidate 既没有赢也没有输。如果有多个 follower 同时成为 candidate，选票是可能被瓜分的，如果没有任何一个 candidate 能得到大多数节点的支持，那么每一个 candidate 都会超时。此时 candidate 需要增加自己的 term，然后发起新一轮选举。如果这里不做一些特殊处理，选票可能会一直被瓜分，导致选不出 leader 来。这里的“特殊处理”指的就是前文所述的**随机化选举超时时间**。



### Leader 切换状态转移过程

有这样的一个场景：当 leader 节点发生了宕机或网络断连，此时其它 follower 会收不到 leader 心跳，首个触发超时的节点会变为 candidate 并开始拉票（由于随机化各个 follower 超时时间不同），由于该 candidate 的 term 大于原 leader 的 term，因此所有 follower 都会投票给它，这名 candidate 会变为新的 leader。一段时间后原 leader 恢复了，收到了来自新leader 的心跳包，发现心跳中的 term 大于自己的 term，此时该节点会立刻切换为 follower 并跟随的新 leader。



**S4 作为 term2 的 leader；S4 宕机，S5率先超时**

<div align=middle><img src=".images/33f858652afc4be6bb1d7b1f1dc33eaa~tplv-k3u1fbpfcp-watermark.image" width="40%" height="40%" /></div>

**S5当选为 leader**

<div align=middle><img src=".images/1418b09699fe4614a594a565c55055d5~tplv-k3u1fbpfcp-watermark.image" width="40%" height="40%" /></div>



**S4 恢复后，收到来自 S5的心跳，发现 S5的 term 大于自己的 term。于是选择跟随**

<div align=middle><img src=".images/31b9745a2fa94fb693a7ea1257aa5f7f~tplv-k3u1fbpfcp-watermark.image" width="40%" height="40%" /></div>