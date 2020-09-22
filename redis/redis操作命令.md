### 连接命令

```bash
# 查询当前连接信息
redis-cli -h 10.20.252.1 -c -p 7101 client list
 
# 监视器
redis-cli -h 10.20.252.1 -c -p 7101 monitor
 
# 查看内存
redis-cli -h 10.20.252.1 -p 7101 -c info memory

# 查看大 Key
redis-cli -h 10.20.252.1 -p 7101 -c --bigkeys
```



### Redis Cluster 相关命令

```bash
集群(cluster)
cluster info 打印集群的信息
cluster nodes 列出集群当前已知的所有节点(node)，以及这些节点的相关信息节点
cluster meet 将ip和port所指定的节点添加到集群当中，让它成为集群的一份子
 
 
cluster forget 从集群中移除node_id指定的节点
cluster replicate 将当前节点设置为node_id指定的节点的从节点
cluster saveconfig 将节点的配置文件保存到硬盘里面
 
 
cluster slaves 列出该slave节点的master节点
cluster set-config-epoch 强制设置configEpoch槽(slot)
 
 
cluster addslots [slot …] 将一个或多个槽(slot)指派(assign)给当前节点
cluster delslots [slot …] 移除一个或多个槽对当前节点的指派
cluster flushslots 移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点
 
 
cluster setslot node 将槽slot指派给node_id指定的节点，如果槽已经指派给另一个节点，那么先让另一个节点删除该槽，然后再进行指派
cluster setslot migrating 将本节点的槽slot迁移到node_id指定的节点中
cluster setslot importing 从node_id 指定的节点中导入槽slot到本节点
 
 
cluster setslot stable 取消对槽slot的导入(import)或者迁移(migrate) 键
cluster keyslot 计算键key应该被放置在哪个槽上
cluster countkeysinslot 返回槽slot目前包含的键值对数量
cluster getkeysinslot 返回count个slot槽中的键
 
 
其它
cluster myid 返回节点的ID
cluster slots 返回节点负责的slot
cluster reset 重置集群，慎用
```

