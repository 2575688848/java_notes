## Kafka 操作

#### 调整kakfaheap内存的配置

修改kafka-server-start.sh的配置

 

#### 创建一个16个分区，2个副本的 hisearch-search topic

./kafka-topics.sh --create --bootstrap-server 10.20.0.22:9092 --replication-factor 2 --partitions 16 --topic hisearch-search

 

#### kafka启动 先启动zookeeper，再启动kafka。

如有没有zookeeper环境的话，kafka有自带打包和配置好的Zookeeper。

**./zookeeper-server-start.sh -daemon ../config/zookeeper.properties**

**./kafka-server-start.sh -daemon ../config/server.properties**



#### 查看消费 Group 列表：

./kafka-consumer-groups.sh --bootstrap-server 10.20.0.22:9092 --list

./kafka-consumer-groups.sh --bootstrap-server 10.20.0.22:9092 --group test_group --describe 

```
TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST 
test            0          5               5               0               -               -              
 
# CURRENT-OFFSET: 当前消费者群组最近提交的 offset，也就是消费者分区里读取的当前位置
# LOG-END-OFFSET: 当前最高水位偏移量，也就是最近一个读取消息的偏移量，同时也是最近一个提交到集群的偏移量
# LAG：消费者的 CURRENT-OFFSET 与 broker 的 LOG-END-OFFSET 之间的差距
```



#### 查看topic相关信息

./kafka-topics.sh --list --bootstrap-server 10.20.0.22:9092

./kafka-topics.sh --topic hisearch-search --describe --bootstrap-server 10.20.0.22:9092

通过 CMAK 删除的topic 数据文件也会被同时删除。 

注：如果被删除的topic此时仍有消费者连接的话，是无法完全删除的。

 

#### 打开kafka生产者控制台（可在控制台模拟产生消息）

./kafka-console-producer.sh --broker-list 10.20.0.22:9092 --topic hisearch-search

 

#### 打开kafka消费者控制台（可在控制台观察到消息被消费）

./kafka-console-consumer.sh --bootstrap-server 10.20.0.22:9092 --topic hisearch-search --from-beginning

 

#### 查看topic里的消息量

./kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 10.20.0.22:9092 --topic fex_node --time -1

-1 表示查看的是历史所有的消息量，包括已经被删除的

-2 表示查看的是已经被删除的消息量

两者相减得出当前topic的消息量。