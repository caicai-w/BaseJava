## Redis怎么保证高并发和高可用？

Redis不能支撑高并发的瓶颈就在于单机，一般来说单机的redisQPS很难超过10万，面对更高的QPS，redis可以有的架构如下：

### 1.读写分离主从架构

架构做成主从架构，一主多从，主负责写，并且将数据同步到其他slave结点，从结点负责读，所有的读请求全部走从结点，如果QPS再增加，也很简单，就继续增加redis slave就可以了。

#### 1.1 Redis replication的核心机制

(1)redis采用异步方式复制数据到slave结点，不过redis2.8开始，slave node会周期性的确认自己每次复制的数据量

(2)一个master node是可以配置多个slave node，slave node 也可以连接其他slave node。slave node做复制的时候，是不会block master node正常工作的

(3)slave node在做复制的时候，也不会block对自己的查询操作，它会用旧的数据集来提供服务，但是复制完成的时候需要删除旧数据集，加载新数据集，这个时候就会暂停对外服务了。

(6)slave node主要用来进行横向扩容，做读写分离，扩容的slave node可以提高读的吞吐量。

#### 1.2 master做持久化对于主从架构的意义

第一，如果采用主从架构建议必须给master node做持久化，不建议用slave node作为master node的数据热备份，因为如果你关掉master的持久化可能在master宕机重启之后是空的，然后一经过复制，salve node数据也丢了。

如果master把RDB和AOF都关掉了，宕机重启后是没有数据可以恢复的，然后直接认为自己的数据是空的，master就会把空数据集同步到slave上面去，所有的slave的数据全部清空。第二，master的各种备份方案，万一本地数据都丢了，要从备份中挑一份rdb去恢复redis，这样才能确保redis在启动的时候时候是有数据的。即使采用了后续讲解的高可用机制，slave node可以自动接管master node，但是也可能sentinal还没有检测到master failure，master node就自动重启了，还是可能导致上面的所有slave node数据清空故障。

#### 1.3 主从架构的核心原理

当启动一个slave node的时候，它会发送一个PSYNC命令给master node，如果这是slave node重新连回master，那么master node仅仅会复制给slave部分缺少的数据; 否则如果是slave node第一次连接master node，那么会触发一次full resynchronization。

开始full resynchronization的时候，master会启动一个后台线程，开始生成一份RDB快照文件，同时还会将从客户端收到的所有写命令缓存在内存中。RDB文件生成完毕之后，master会将这个RDB发送给slave，slave会先写入本地磁盘，然后再从本地磁盘加载到内存中。然后master会将内存中缓存的写命令发送给slave，slave也会同步这些数据。

 slave node如果跟master node有网络故障，断开了连接，会自动重连。master如果发现有多个slave node都来重新连接，仅仅会启动一个rdb save操作，用一份数据服务所有slave node。 

#### 1.4 主从复制的断点续传

从redis 2.8开始，就支持主从复制的断点续传，如果主从复制过程中，网络连接断掉了，那么可以接着上次复制的地方，继续复制下去，而不是从头开始复制一份。

Master node会在内存中创建一个backlog，master和slave都有一个replica offset和一个master id，offset就是保存在backlog中的，如果master和slave网络连接断掉了，slave会让master从上次的replica offset开始继续复制。但是如果没有找到对应的offset，就会执行一次全量复制。 

#### 1.5 无磁盘化复制

master在内存中创建rdb，然后发送给slave，不会在自己本地落盘了，是直接写到slave 的socket里面去。如果要落盘的话，就是先把RDB保存到file。

 repl-diskless-sync  no

repl-diskless-sync-delay 5s，等待一定时长再开始复制，因为要等更多slave重新连接过来

 如果是落盘，就是master开启一个子进程写到磁盘，再发给slave，如果是master无磁盘复制，就是把master的数据写到slave的进程里面去。

#### 1.6 过期key处理

 slave不会过期key，只会等待master过期key。如果master过期了一个key，或者通过LRU淘汰了一个key，那么会模拟一条del命令发送给slave。

#### 1.7 完整复制流程

（1）slave node启动，仅仅保存master node的信息，包括master node 的host和ip，但是复制流程还没开始master host和ip是从哪来的，redis.conf里配置的。

（2）slave node内部有个定时任务，每s会检查是否有新的master node要连接和复制，如果发现就跟master node建立socket网络连接。

（4）slave node 发送ping给master node

  (5)  口令认证，如果master设置了requirepass，那么slave node必须发送masterauth的口令过去进行认证

 (6) master node后续持续写命令，异步复制给slave node。

#### 1.8 数据同步的核心

指的是第一次slave连接master的时候，执行的是全量复制，关于这个的一些细节：

（1）master和slave都会维护一个offset，master会在自身不断的累加，slave也会，slave每s都会上报自己的offset给master，同时master也会保存每个slave的offset。

（2）backlog，master node有个backlog，默认1MB，master node给slave node复制数据时，也会将数据。backlog是做中断后的增量复制的。

（3）run id，info server，可以看到master run id，如果根据host+ip定位master node，是不靠谱的，比如说，数据出现了错误，我要把数据恢复到20小时之前的数据，那我用一份rdb去应用到master，这个时候run id就应该变了，run id不同，slave就会全量复制。
如果需要不更改run id重启redis，可以使用redis-cli debug reload命令。

（4）psync

从节点使用psync从master node进行复制，psync runid offset
master node会根据自身的情况返回响应信息，如果run id就不一样，就全量复制，可能是FULLRESYNC runid offset触发全量复制，如果是发现之前断掉的，可能是CONTINUE触发增量复制。



#### 1.9 全量复制

（1）master执行bgsave，在本地生成一份rdb快照文件
（2）master node将rdb快照文件发送给salve node，如果rdb复制时间超过60秒（repl-timeout），那么slave node就会认为复制失败，可以适当调节大这个参数
（3）对于千兆网卡的机器，一般每秒传输100MB，6G文件，很可能超过60s
（4）master node在生成rdb时，会将所有新的写命令缓存在内存中，在salve node保存了rdb之后，再将新的写命令复制给salve node
（5）client-output-buffer-limit slave 256MB 64MB 60，如果在复制期间，master内存缓冲区持续消耗超过64MB，或者一次性超过256MB，那么停止复制，复制失败。就是说写的太多了。
（6）slave node接收到rdb之后，清空自己的旧数据，然后重新加载rdb到自己的内存中，同时基于旧的数据版本对外提供服务。
（7）如果slave node开启了AOF，那么会立即执行BGREWRITEAOF，重写AOF。

rdb生成->rdb通过网络拷贝->slave旧数据的清理->slave aof rewrite，很耗费时间

如果复制的数据量在4G~6G之间，那么很可能全量复制时间消耗到1分半到2分钟

#### 1.10 增量复制

（1）如果全量复制过程中，master-slave网络连接断掉，那么salve 就会用上一次的pysnc runid offset，重新连接master时，会触发增量复制.
（2）master直接从自己的backlog中获取部分丢失的数据，发送给slave node，默认backlog就是1MB
（3）msater就是根据slave发送的psync中的offset来从backlog中获取数据的

heartbeat：主从节点互相都会发送heartbeat信息

master默认每隔10秒发送一次heartbeat，salve node每隔1秒发送一个heartbeat

#### 1.11 异步复制

master每次接收到写命令之后，现在内部写入数据，然后异步发送给slave node。

### 2. 主从架构如何做到高可用性

