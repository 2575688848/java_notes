### 日志文件写入流程

写入顺序：
	undo（数据回滚，MVCC） -> redo（事务回滚，数据同步到真实磁盘数据页） -> binlog（数据备份，主从同步）

​	数据写入磁盘 是通过异步，用一个后台线程把 redolog 里的数据同步到聚簇索引上的数据页上。这时候，磁盘上的真正存储的数据可能还没有更新，此时这种数据页叫做脏页，脏页需要丢弃的时候，要将脏页对应的 redolog 里的数据刷新到磁盘的真实数据页中。

​	写入 redolog 磁盘文件之前会把数据写到缓冲池中（相当于是在内存的InnoDB 索引）。思考一下此时如果有其它事务来操作这个数据的话，会不会造成数据的脏读？

​	答：是不会的，因为有行锁的限制，读取和更新操作都不会在此数据行上生效。所以，虽然数据已经在缓冲池中了，但是其它的事务并不会读取到这个数据行。

​	如果最后事务提交成功，则会保留缓冲池的数据，如果提交不成功，则缓冲池数据丢弃。

​	uodolog： 当事务提交成功后，将此事务更新的数据信息，记录在 uodo 中，用来实现 MVCC。



redo ：logbuffer（mysql缓存），oscache（操作系统），redolog file（磁盘）

![image-20211216102741806](.images/image-20211216102741806.png)

​	**两阶段提交：**

​	prepare：成功写入redolog （涉及到 redolog 崩溃）
​	commit：说明 binlog 已经写入成功

![image2020-7-10 11_6_11](.images/image2020-7-10 11_6_11.png)

MySQL 有一个参数：innodb_flush_log_at_trx_commit 能够控制事务提交时，刷 redo log 的策略。

 目前有**三种策略**：

![image2020-7-10 11_6_43](.images/image2020-7-10 11_6_43.png)



binlog：binlogCache
	binlog 写入磁盘文件成功之后，将 redolog 状态置为 commit



### redolog

InnoDB 的 redo log 是固定大小的，比如可以配置为一组 4 个文件，每个文件 的大小是 1GB，那么这块“粉板”总共就可以记录 4GB 的操作。从头开始写，写到末尾就 又回到开头循环写，WAL 技术，WAL 的全称 是 Write-Ahead Logging 先写到内存 redolog 中

如下面这个图所示：

![image-20200928185007272](.images/image-20200928185007272.png)

redolog 文件中有两个指针：checkpoint 和 write pos

write pos 和 checkpoint 之间的是“粉板”上还空着的部分，可以用来记录新的操作。如果 write pos 追上 checkpoint，表示“粉板”满了，这时候不能再执行新的更新，得停下来先 擦掉一些记录，把 checkpoint 推进一下。



### binlog

mysql 自带的日志，不能实现 crash-safe 能力。

1. redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都 可以使用。 
2. redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志， 记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。 
3. redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。



更新时，先更新 redolog（prepare），然后再更新 binlog，最后执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。

更新时两阶段提交：prepare 和 commit。

```bash
## 时间点1: 此时 redolog 和 binlog都没写日志，所以不会出现问题
1、先进入commit prepare 阶段，这个阶段事务中新生成的redo log 会被刷到磁盘，并将回滚段置为prepared状态。

## 时间点2: 此时虽然 redolog 有日志，但处于prepare阶段，但在服务器故障恢复时会判断 binlog 的完整性。因为binlog没有，所以回滚。
2、commit阶段：innodb释放锁，释放回滚段，设置redo log提交状态，binlog持久化到磁盘，然后存储引擎层提交。

## 时间点3: 此时 redolog 和 binlog都已经刷入磁盘

```



#### binlog格式

**statement：**

这种格式会带来一些问题：

1. 如果 delete 语句使用的是索引 a，那么会根据索引 a 找到第一个满足条件的行，也就是说删除的是 a=4 这一行； 

2. 但如果使用的是索引 t_modified，那么删除的就是 t_modified='2018- 11-09’也就是 a=5 这一行。 

3. 由于 statement 格式下，记录到 binlog 里的是语句原文，因此可能会出 现这样一种情况：在主库执行这条 SQL 语句的时候，用的是索引 a，在备库用的索引是 t_modified，因此， MySQL 认为这样写是有风险的。

   

```bash
## 查看binlog 文件里的内容
mysql> show binlog events in 'master.000001';
```

![image-20201019172412049](.images/image-20201019172412049.png)



**row：**（多用于恢复数据的场景，也是经常被使用的格式）

row 格式的 binlog 里没有了 SQL 语句的原 文，而是替换成了两个 event：Table_map 和 Delete_rows。

1. Table_map event，用于说明接下来要操作的表是 test 库的表 t; 
2. Delete_rows event，用于定义删除的行为。

当 binlog_format 使用 row 格式的时候，binlog 里面记录了 真实删除行的主键 id，这样 binlog 传到备库去的时候，就肯定会删除 id=4 的行，不会有主备删除不同行的问题。

![image-20201019173439935](.images/image-20201019173439935.png)

使用 mysqlbinlog 工具查看详细信息：

```bash
mysqlbinlog  ‑vv data/master.000001 ‑‑start‑position=8900;
```

![image-20201019174511238](.images/image-20201019174511238.png)



**mixed：**前两种格式的混合。

为什么会有这种格式：
因为有些 statement 格式的 binlog 可能会导致主备不一致，所以要使 用 row 格式。 

但 row 格式的缺点是，很占空间。比如你用一个 delete 语句删掉 10 万 行数据，用 statement 的话就是一个 SQL 语句被记录到 binlog 中，占 用几十个字节的空间。但如果用 row 格式的 binlog，就要把这 10 万条 记录都写到 binlog 中。这样做，不仅会占用更大的空间，同时写 binlog 也要耗费 IO 资源，影响执行速度。 

所以，MySQL 就取了个折中方案，也就是有了 mixed 格式的 binlog。 mixed 格式的意思是，MySQL 自己会判断这条 SQL 语句是否可能引起 主备不一致，如果有可能，就用 row 格式，否则‘就用 statement 格式。



**双主架构（双M）的循环复制问题**

双M：两台mysql服务互为主备。

MySQL 在 binlog 中记录了这个命令第一次执 行时所在实例的 server id。因此，我们可以用下面的逻辑，来解决两个节 点间的循环复制的问题： 

1. 规定两个库的 server id 必须不同，如果相同，则它们之间不能设定为 主备关系；
2. 一个备库接到 binlog 并在重放的过程中，生成与原 binlog 的 server id 相同的新的 binlog； 
3. 每个库在收到从自己的主库发过来的日志后，先判断 server id，如果跟 自己的相同，表示这个日志是自己生成的，就直接丢弃这个日志。



#### 如果没有两阶段提交导致的问题？

**先写 redo log 直接提交，然后写 binlog**

假设写完 redo log 后，机器挂了，binlog 日志没有被写入，那么机器重启后，这台机器会通过 redo log 恢复数据，但是这个时候 bingog 并没有记录该数据，后续进行机器备份的时候，就会丢失这一条数据，同时主从同步也会丢失这一条数据。



**先写binlog，然后写 redo log** 

假设写完了 binlog，机器异常重启了，由于没有 redo log，本机是无法恢复这一条记录的，但是 binlog 又有记录，那么和上面同样的道理，就会产生数据不一致的情况。



如果采用 redo log 两阶段提交的方式就不一样了，写完 binglog 后，然后再提交 redo log 就会防止出现上述的问题，从而保证了数据的一致性。那么问题来了，有没有一个极端的情况呢？

假设 redo log 写完了，binglog 也已经写完了，这个时候发生了异常重启会怎么样呢？ 这个就要依赖于 MySQL 的处理机制了，MySQL 的处理过程如下：

* 判断 redo log 是否是 commit 状态，如果是，就立即提交。
* 如果 redo log 是预提交但不是 commit 状态，这个时候就会去判断 binlog 是否完整，如果完整就提交 redo log,  不完整就回滚事务。

这样就解决了数据一致性的问题。



