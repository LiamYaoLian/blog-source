---

title: "Redis Summary"
date: 2022-06-14 20:43:32
tags:
---

## redis-cli
* select 0 // select database No. 0
* set name lily // 设置key=name, value=lily
* get hello // 获得key=hello结果
* keys he* // 根据Pattern表达式查询符合条件的Key
* dbsize // 返回key的总数
* exists a // 检查key=a是否存在
* del a // 删除key=a的数据
* expire hello 20 // 设置key=hello 20秒后过期
* ttl hello // 查看key=a的过期剩余时间
* keys * // 线程阻塞
* scan // 无阻塞，有重复概率，客户端去重，花费的时间会比直接用keys指令长, TODO
* SETEX key seconds value
* lrange // TODO
* zset // TODO
* other // TODO
要了解 MC 和 Redis 的常用命令，例如原子增减、对不同数据结构进行操作的命令等。
了解 MC 和 Redis 在内存中的存储结构，这对评估使用容量会很有帮助。
了解 MC 和 Redis 的数据失效方式和剔除策略，比如主动触发的定期剔除和被动触发延期剔除
要理解 Redis 的持久化、主从同步与 Cluster 部署的原理，比如 RDB 和 AOF 的实现方式与区别。
不管你有没有电商经验我觉得你都应该知道秒杀的具体实现，以及细节点。



## Data Type
* string
* hash
* list: MQ; pagination
* Set: remove duplicates
* Sorted Set
* Bitmap: BloomFilter // TODO
* HyperLogLog: 供不精确的去重计数功能，比较适合用来做大规模数据的去重统计，例如统计 UV；// TODO
* Geospatial: 可以用来保存地理位置，并作位置距离计算或者根据半径计算位置等。有没有想过用Redis来实现附近的人？或者计算最优地图路径？ //TODO
* pub/sub // TODO
* Pipeline // TODO
* Lua // TODO

## TPS
* roughly 100k


## Distributed Lock


## Redis async queue
* list: rpush to produce, lpop to consume; 当lpop没有消息的时候，要适当sleep一会再重试。list还有个指令叫blpop，在没有消息的时候，它会阻塞住直到消息到来
* 使用pub/sub主题订阅者模式，可以实现 1:N 的消息队列。在消费者下线的情况下，生产的消息会丢失，得使用专业的消息队列如RocketMQ等
* 延时队列：使用sortedset，拿时间戳作为score，消息内容作为key调用zadd来生产消息，消费者用zrangebyscore指令获取N秒之前的数据轮询进行处理

## Persistency
* RDB做镜像全量持久化，AOF做增量持久化。因为RDB会耗费较长时间，不够实时，在停机的时候会导致大量丢失数据，所以需要AOF来配合使用。在redis实例重启时，会使用RDB持久化文件重新构建内存，再使用AOF重放近期的操作指令来实现完整恢复重启之前的状态。AOF持久化开启且存在AOF文件时，优先加载AOF文件；AOF关闭或者AOF文件不存在时，加载RDB文件；加载AOF/RDB文件成功后，Redis启动成功； AOF/RDB文件存在错误时，Redis启动失败并打印错误信息
* AOF定时sync，防止机器断电

Redis 提供了 RDB 和 AOF 两种持久化方式，RDB 是把内存中的数据集以快照形式写入磁盘，实际操作是通过 fork 子进程执行，采用二进制压缩存储；AOF 是以文本日志的形式记录 Redis 处理的每一个写入或删除操作。

RDB 把整个 Redis 的数据保存在单一文件中，比较适合用来做灾备，但缺点是快照保存完成之前如果宕机，这段时间的数据将会丢失，另外保存快照时可能导致服务短时间不可用。

AOF 对日志文件的写入操作使用的追加模式，有灵活的同步策略，支持每秒同步、每次修改同步和不同步，缺点就是相同规模的数据集，AOF 要大于 RDB，AOF 在运行效率上往往会慢于 RDB。

## Sync
第一次同步时，主节点做一次bgsave，并同时将后续修改操作记录到内存buffer，待完成后将RDB文件全量同步到复制节点，复制节点接受完成后将RDB镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程。后续的增量数据通过AOF日志同步即可



## Cache avalanche
* 缓存挂掉，所有的请求都会穿透到 DB
* 解决方法：
  - 使用快速失败的熔断策略，减少 DB 瞬间压力；
  - 使用主从模式和集群模式来尽量保证缓存服务的高可用。


## Cache penetration
* frequenly query a data (e.g. ID = -1) that does not exist in both DB and cache, crush the DB
* solve:
  - validate (e.g. ID cannot be -1);
  - 从缓存取不到的数据，在数据库中也没有取到，这时也可以将对应Key的Value对写为null或类似的标记，有效期设置较短，如30s
  - Bloom Filter // TODO

## Hotspot Invalid
* solution
  - set hotspot to "never expire"
  - lock (may need LUA): https://juejin.cn/post/6844903986475057165 可以使用互斥锁更新，保证同一个进程中针对同一个数据不会并发请求到 DB，减小 DB 压力。
  - 使用随机退避方式，失效时随机 sleep 一个很短的时间，再次查询，如果失败再执行更新。
  - 失效时间太集中，缓存没什么数据，此时有大量访问，导致数据库有大量访问：失效时间应加随机值

## Type
* local cache
* distributed cache: pro: big data; con: remote request, slower
* 多级缓存，本地缓存只保存访问频率最高的部分热点数据，其他的热点数据放在分布式缓存中。

## Cache elimination strategy
* noeviction
* allkeys-lru
* volatile-lru
* allkeys-random
* volatile-random
* volatile-ttl


## VS Memcache
* 多线程异步 IO （Redis: single thread, async)
* 失效的策略采用延迟失效，就是当再次使用数据时检查是否失效；
* 当容量存满时，会对缓存中的数据进行剔除，剔除时除了会对过期 key 进行清理，还会按 LRU 策略对数据进行剔除。
* key 不能超过 250 个字节；
* value 不能超过 1M 字节；
* key 的最大失效时间是 30 天；
* 只支持 K-V 结构，不提供持久化和主从同步功能。

## Sentinel
* 哨兵必须用三个实例去保证自己的健壮性
集群监控：负责监控 Redis master 和 slave 进程是否正常工作。
消息通知：如果某个 Redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员。
故障转移：如果 master node 挂掉了，会自动转移到 slave node 上。
配置中心：如果故障转移发生了，通知 client 客户端新的 master 地址。

## Expire
* Redis 的 key 可以设置过期时间，过期后 Redis 采用主动和被动结合的失效机制，一个是和 MC 一样在访问时触发被动删除，另一种是定期的主动删除。

## Data consistency
* 更新 DB 后，更新 Redis 因为网络原因请求超时；或者是异步更新失败导致
* 解决的办法是，如果服务对耗时不是特别敏感可以增加重试；如果服务对耗时敏感可以通过异步补偿任务来处理失败的更新，或者短期的数据不一致不会影响业务，那么只要下次更新时可以成功，能保证最终一致性就可以。

## 加分项
最好可以了解缓存使用中可能产生的问题。比如 Redis 是单线程处理请求，应尽量避免耗时较高的单个请求任务，防止相互影响；Redis 服务应避免和其他 CPU 密集型的进程部署在同一机器；或者禁用 Swap 内存交换，防止 Redis 的缓存数据交换到硬盘上，影响性能。再比如前面提到的 MC 钙化问题等等。
要了解 Redis 的典型应用场景，例如，使用 Redis 来实现分布式锁；使用 Bitmap 来实现 BloomFilter，使用 HyperLogLog 来进行 UV 统计等等。
知道 Redis4.0、5.0 中的新特性，例如支持多播的可持久化消息队列 Stream；通过 Module 系统来进行定制功能扩展等等。


https://juejin.cn/post/6844903989184577550
https://juejin.cn/post/6844903991562747912
