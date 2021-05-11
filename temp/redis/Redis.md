# Redis

1. Redis是单线程的吗？单线程为什么还这么快？讲一讲redis的内存模型？Redis底层实现？Redis是如何更新缓存的？Redis 多线程是什么多线程？默认开启吗？
2. Redis它的5种基础类型和6个数据结构？HyperLogLog、BitMap、GEO、Stream什么时候使用？跳表是什么？为什么使用跳表？为什么不用红黑树？全局Hash表又是什么？如何扩容的？如何渐进式rehash？
3. IO多路复用是什么？持久化方式？分别优缺点是什么？优化策略？主从复制怎么做？如何保证可用性？怎么实现分布式锁？需要注意什么问题？布隆过滤器？缓存会有哪些问题？
4. 存储方式？集群下怎么同步？怎么做限流功能？令牌桶这个限流怎么做成分布式的？淘汰策略？更新策略是什么？缓存过期策略？哨兵机制和集群的区别？
5. Redis里的CAP是怎样的？热key怎么处理？如何保证数据库与缓存双写的一致性？Redis发生主备切换会出现什么问题？缓存穿透？缓存雪崩？缓存击穿？缓存污染？
6. cluster数据分片规则？怎么做到高可用？缓存同步机制？同步策略？如何实现一个Key千万并发？Redis事务？

## Redis的事务有哪些命令？

​		一个事务以 ``MULTI`` 开始一个事务， 然后将多个命令入队到事务中， 最后由 ``EXEC`` 命令触发事务， 一并执行事务中的所有命令，从开始到执行会经历以下三个阶段：

- 开始事务。
- 命令入队。
- 执行事务。

**相关命令**

- `WATCH`：监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。
- `UNWATCH`：取消 `WATCH `命令对所有 key 的监视。
- `MULTI`：标记一个事务块的开始。
- `EXEC`：执行所有事务块内的命令。
- `DISCARD`：取消事务，放弃执行事务块内的所有命令。

**例子**

```shell
redis 127.0.0.1:7000> multi
OK
redis 127.0.0.1:7000> set a aaa
QUEUED
redis 127.0.0.1:7000> set b bbb
QUEUED
redis 127.0.0.1:7000> set c ccc
QUEUED
redis 127.0.0.1:7000> exec
1) OK
2) OK
3) OK
```

## Redis 里的 CAP 是怎样的?

**CAP**

- C（Consistency）：一致性
- A（Availability）： 可用性
- P（Partition tolerance）：  分区容错性（指系统能够容忍节点之间的网络通信的故障）。

**对于Redis来说，如果是单机的话，是CP，而如果要使用slave（主仆模式）的话就变为了AP。**

## 怎么实现分布式锁？

**实现原理**

​		Redis 锁主要利用 Redis 的 setnx 命令。（`SET lockId content PX millisecond NX `）。

- 加锁命令：`SETNX key value`，当键不存在时，对键进行设置操作并返回成功，否则返回失败。KEY 是锁的唯一标识，一般按业务来决定命名。
- 解锁命令：`DEL key`，通过删除键值对释放锁，以便其他线程可以通过 SETNX 命令来获取锁。
- 锁超时：`EXPIRE key timeout` 设置 key 的超时时间，以保证即使锁没有被显式释放，锁也可以在一定时间后自动释放，避免资源被永远锁住。

**可能产生的问题**

1. **锁误解除**

   ​		如果线程 A 成功获取到了锁，并且设置了过期时间 30 秒，但线程 A 执行时间超过了 30 秒，锁过期自动释放，此时线程 B 获取到了锁；随后 A 执行完成，线程 A 使用 DEL 命令来释放锁，但此时线程 B 加的锁还没有执行完成，线程 A 实际释放的线程 B 加的锁。

   解决方法：通过在 value 中设置当前线程加锁的标识，在删除之前验证 key 对应的 value 判断锁是否是当前线程持有。可生成一个 UUID 标识当前线程。

2. **超时解锁导致并发**

   ​		如果线程 A 成功获取锁并设置过期时间 30 秒，但线程 A 执行时间超过了 30 秒，锁过期自动释放，此时线程 B 获取到了锁，线程 A 和线程 B 并发执行。

   解决方法：将过期时间设置足够长，确保代码逻辑在锁释放之前能够执行完成。

3.  **不可重入**

   ​		当线程在持有锁的情况下再次请求加锁，如果一个锁支持一个线程多次加锁，那么这个锁就是可重入的。如果一个不可重入锁被再次加锁，由于该锁已经被持有，再次加锁会失败。Redis 可通过对锁进行重入计数，加锁时加 1，解锁时减 1，当计数归 0 时释放锁。

   解决方法：Redis Map 数据结构来实现分布式锁，既存锁的标识也对重入次数进行计数。





## Redis怎么实现高可用？

- 主从复制数据。
- 采用哨兵监控数据节点的运行情况，一旦主节点出现问题由从节点顶上继续进行服务。

## Redis有哪几种数据类型及底层实现，分别用在什么场景？

**String**

**List**

**Hash**

**Set**

**Sorted Set**
