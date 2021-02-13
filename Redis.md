# Redis

###### 基本数据结构

- 字符串（string） ~简单来说就是三种：int、string、byte[]

  是二进制安全的，最多可容纳数据长度是512M

  注意：

  1. 字符串append(每次都会开出一片内存用于存储，会使用更多的内存)
  2. 整数共享(尽量使用整数，配置redis内存或LRU，整数共享可能会失效)
  3. 整数精度问题(精度大概能保证16)

- 散列（hash）~ Map或Pojo Class

- 列表（list）~ LinkedList

- 集合（set）set，不重复的list

- 有序集合（sorted set）~每个成员都会有一个分数与之关联，通过分数排序

- Bitmaps ~不是一个真实的数据结构

  String类型上的一组面向bit操作的集合

- Hyperloglogs ~HLL

  可以用总内存倒推计算，但不是很准确

- GEO

  用户给定的地理位置(经度和围读)信息储存起来，并对这些信息进行操作

###### Java原生框架

- Redis的Java客户端-Lettuce

- Redis 的 Java 客户端-Redission(分布式版本)

###### Spring Data Redis

​	核心是RedisTemplate、可配置于Jedis，Lettuce，Redisson

###### Spring Boot 与 Redis集成

​	引入spring-boot-starter-data-redis

###### Spring Cache 与Redis集成

​	默认使用全局的CacheManager自动集成

​	使用ConcurrentHashMap或echache时，不需要考虑序列化问题

​	redis的话需要1、默认使用Java的对象序列化，对象需要实现Serializable；2、自定义配置，可以修改为其他序列化方式

###### Redis事务

- Redis事务命令

  开启事务：multi

  命令入队

  执行事务：exec

  撤销事务：discard

- Watch实现乐观锁

  watch 一个key,发生变化则事务失败

- Redis Lua

  单线程->脚本执行具有原子性，操作不会被打断，值不会被修改

  每个脚本执行，都能保证事务

- Redis管道技术(pipeline)

  合并操作批量处理，且不阻塞前序命令

###### Redis性能优化

- 内存优化~10G/20G

  https://redis.io/topics/memory-optimization

  hash-max-ziplist-value 64

  zset-max-ziplist-value 64

- CPU优化~单线程，线上千万不能用keys *

  不要阻塞，特别是lua脚本

  谨慎使用范围操作

  SLOWLOG get 10 //默认10毫秒，默认只保留最后的128条

###### Redis分区

- 容量~多个业务系统，共用一个redis，还是应该分开？

  规划好key，特别是前缀

- 分区

  类似垂直拆分

###### Redis使用的一些经验

- 性能

  client~线程数(4~8)与连接数(redis ~ 10000)

  监控系统读写比和缓存命中率(N:1，90%+)

- 容量

  做好容量评估，合理使用缓存资源；监控注重增量变化

- 资源管理和分配

  尽量每个业务集群单独使用自己的Redis，不混用

  控制Redis资源的申请与使用，规范环境和Key的管理

  监控CPU 100%,优化高延迟的操作

###### 使用场景

- 通用数据缓存，string，int，list，map等*
- 实时热数据，最新500条数据*
- 会话缓存，token缓存等*
- 非严格一致性要求的数据：评论，点击
- 业务数据去重：订单处理的幂等校验等。
- 业务数据排序：排名，排行榜等。
- 全局流控计数
- 秒杀的库存计算*
- 抢红包*
- 全局ID生成*
- id去重，记录访问ip等全局bitmap操作
- UV、PV等访问量==>非严格一致性要求
- 发布订阅与Stream
- 分布式锁*
  1. 获取锁--单个原子性操作
  2. 释放锁--lua脚本(保证原子性+单线程，从而具有事务性)

###### 可能会出现的问题

- 缓存穿透

  大量并发查询不存在的KEY，导致都直接将压力透传到数据库

  解决办法：

  1. **缓存空值**的KEY，这样第一次不存在也会被加载记录，下次拿到有这个KEY
  2. **Bloom过滤**或**RoaringBitmap**判断KEY是否存在
  3. 完全以缓存为准，使用**延迟异步加载**的策略，这样就不会触发更新

- 缓存击穿

  某个KEY失效的时候，正好有大量并发请求访问这个KEY

  解决办法：

  1. KEY的更新操作添加**全局互斥锁**
  2. 完全以缓存为准，使用**延迟异步加载**的策略，这样就不会触发更新

- 缓存雪崩

  当某一时刻发生大规模的缓存失效的情况，会有大量的请求进来直接打在数据库，导致数据库压力过大升值宕机

  解决办法：

  1. 更新策略在时间上做到比较均匀
  2. 使用的热数据尽量分散到不同的机器上
  3. 多台机器做主从复制或者多副本，实现高可用
  4. 实现熔断限流机制，对系统进行负载能力控制

###### 过期策略

- 按FIFO或LRU
- 按固定时间过期
- 按业务时间加权：例如3+5x

###### 数据备份与回复

- RDB ~ frm(全量)

  备份

  ​	执行 save 即可在redis数据目录生成数据文件 dump.rdb 

  ​	也可以异步执行 bgsave 

  2.恢复

  ​	将备份文件 (dump.rdb) 移动到 redis 数据目录并启动服务即可 

  ​	查看文件夹 CONFIG GET dir 

- AOF ~ binlog(增量)

  备份

  ​	如果 appendonly 配置为 yes，则以 AOF 方式备份 Redis 数据，那么此时 Redis 会按 

  ​	照配置，在特定的时候执行追加命令，用以备份数据。 

  ​	appendfilename "appendonly.aof" 

  ​	# appendfsync always 

  ​	# appendfsync everysec 

  ​	# appendfsync no...... 

  ​	AOF 文件和 Redis 命令是同步频率的，假设配置为 always，其含义为当 Redis 执行 

  ​	命令的时候，则同时同步到 AOF 文件，这样会使得 Redis 同步刷新 AOF 文件，造成 

  ​	缓慢。而采用 evarysec 则代表每秒同步一次命令到 AOF 文件。 

  恢复

  ​	自动加载