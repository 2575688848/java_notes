## Boss 直聘



### 一面

#### 1、介绍下搜索项目的整体流程

**索引建立过程：**

##### 全量抓取：

两种方式：

1、通过 jsp 的控制台页面手动启动全量抓取。

2、每隔12小时会全量更新一次。

直接从数据库表中分批次的获取全量数据存储到 memcached 中和 redis 索引操作队列。INDEXMESSAGE 中。

全量抓取会根据已经定义好的类型，每个类型都对应一个表。来从各个表中全量的构建抓取任务。

任务名叫：DataPumpJob  （每 500 个数据一个 job ）放到 ConcurrentLinkedQueue<Job> 队列中（一个基于链表的无界线程安全队列）。

创建 DataPumpTaskNum （25）数量的线程 DataPumpTask。

DataPumpTask 被提交到线程池来消费上面的那些 DataPumpJob。

在 DataPumpTask 执行 run 方法，从 ConcurrentLinkedQueue 中获取之前构建的 DataPumpJob。

在 DataPumpJob 中，根据 type 获取对应 page 和 size 大小的数据，进行数据的存储操作。

这这里：先判断数据是否是删除状态，如果是则：memcached 数据删除，redis 索引队列插入 goods:345222:delete （345222 为 goodId）。

如果不是删除数据，则：memcached 存储数据，并且 redis 索引队列插入 goods:345222:insert 。



##### 增量抓取：

方式：

1、通过 jsp 的控制台页面手动启动增量抓取。

2、启动完之后就会一直以常驻线程的方式一直运行。

增量同步会开启配置的数据库的数量的 CDCPumpTask 线程，每个线程对应一个数据库。

每个线程将自己订阅的增量数据，序列化后写到 key 为 CANALMESSAGE 的redis list 列表中。

（⚠️：这样就能保证每个库的增量数据在redis中也是有序的）。

之后会启动 CDCProcessTaskNum（100）数量的 CDCProcessTask 线程来处理上面这些序列化后的数据。

在处理的过程中会根据序列化的数据中的 数据库+数据表 获取到不同的处理类，来处理这个数据。

如果类型是插入：则根据 key 从数据库获取最新的消息，然后 memcached 数据插入，redis 索引队列插入 goods:345222:insert

如果类型是更新：先判断是否是删除状态，如果是删除则直接删除。如果不是删除，则也是从数据库获取最新的消息，然后 memcached 数据插入，redis 索引队列插入 goods:345222:insert

如果类型是删除：则 memcached 数据删除，redis 索引队列插入 goods:345222:delete

索引处理：

打开 DataIndexTaskNum 数量的线程 DataIndexTask ，提交到线程池中。来消费 redis 中 INDEXMESSAGE 列表里的消息。

根据从 INDEXMESSAGE 列表里取到信息 goods:345222:insert ，从 memcached 中获取 model 数据。然后构建 SolrInputDocument 写到 Solr 主节点中。

主从同步时间：5 秒钟一次。



**search 过程：**

1、客户端请求打到 search-api 服务，首先根据 search-api  本身已有的配置文件进行同义词转换、加入品牌词扩展、加入商品的权重处理，拼接成一个关键词字符串。然后根据这个关键词去请求 solr。

2、请求 solr 前，构建 SolrQuery  查询参数。

setParam： qt 参数：对应的是 core  solrconfig.xml 中的某个 requestHandler 例：/SearchV03

setQuery(keyword) ：设置查询关键词。

addFilterQuery ： 添加过滤条件。

addSort：添加排序策略。

setFields：获取那些字段信息。

setStart：设置开始行

setRows：设置返回结果行数

最后把这个请求转发到 solr 某个 core 的 requestHandler 上。

3、请求转发到 solr 的 requestHandler ，里面的一些配置。

qf:  查询字段，具体到哪些字段，如果缺省默认为df。例如：qf="fieldOne^2.3 fieldTwo fieldThree^0.4"

bf : boost function（加强功能） 例如：recip(rord(myfield),1,2,3)^1.5，对查询结果打分做更强的处理。

spellcheck：拼写检查｜ 搜索建议。

fq：solr core 本身配置的过滤条件。



#### 2、倒排索引如何建立

elasticsearch 文件夹下的 lucene 文件。



#### 3、solr core的配置

schema.xml：配置 core 存储数据的字段和类型。

solrconfig.xml：core 实例配置文件。主从节点的同步信息，请求的处理，过滤，权重排序。

**solr 的 requestHandler ，里面的一些配置：**

qf:  查询字段，具体到哪些字段，如果缺省默认为df。例如：qf="fieldOne^2.3 fieldTwo fieldThree^0.4"

bf : boost function（加强功能） 例如：recip(rord(myfield),1,2,3)^1.5，对查询结果打分做更强的处理。

spellcheck：拼写检查｜ 搜索建议。

fq：solr core 本身配置的过滤条件。



**扩展词、停止词、同义词配置**

扩展词：相当于是一个完整的 term，不再进行分词处理。在搜索引擎 Solr 里的 ext.dic 文件配置。（配置完需要重启 Solr）

停止词：排除这些词，直接忽略。在配置文件里的 schema.xml 文件配置。（不需要重启，到控制台 reload 一下每个对应 core 即可）

同义词：对这样的词进行分词会得到一个同义词词组里的所有词。在配置文件里的 schema.xml 文件配置。（不需要重启，到控制台 reload 一下每个对应 core 即可）



#### 4、es 的 query 过程

elasticsearch 文件夹的 「es是如何查询的」 文件



#### 5、redis如何实现最近10条查询请求

redis 的 list 结构实现，做一个只有 10 个数据的 FIFO 的队列。当有新请求来的时候，判断当前 list 是否等于 10 。

如果等于 10 的话，就出队列一个，再 push 进去这个新请求。



#### 6、jvm 优化

solr 文件夹下的 「搜索 JVM 优化」。



