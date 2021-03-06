# 五种数据结构
字符(string)，集合(set)，有序集合(sort set)，队列(queue)，哈希(hash)

集合成员是唯一的，不能重复

有序集合成员唯一，score 可以重复

# hash
使用数组存储，哈希冲突使用拉链法存储，当扩容的时候 rehash 是渐进性，需要查找两个数组。

# 分布式锁
setnx，redlock

# 延迟队列
有序集合实现延迟队列

# 位图
redis中所有数据都是二进制形式存储的。redis支持一个setbit和getbit操作，它支持在某个key的value上直接对某个二进制位操作，每个二进制位都只有0和1两种状态，正好可以表示用户是否活跃两种状态。

// 用户id123456是活跃用户

setbit a 123456 1

// 用户id234567不是活跃用户

setbit a 234567 0

getbit a 123456

# 持久化

  - AOF
  
    - AOF 日志是连续的增量备份。
    
    - Linux 的 glibc 提供了fsync(int fd)函数可以将指定文件的内容强制从内核缓存刷到磁盘。只要 Redis 进程实时调用 fsync 函数就可以保证 aof 日志不丢失。
  
  - RDB（快照）
  
    - 快照是内存数据的二进制序列化形式。
    
    - Redis 使用操作系统的多进程 COW(Copy On Write) 机制来实现快照持久化。fork创建出的子进程，与父进程共享内存空间。也就是说，如果子进程不对内存空间进行写入操作的话，内存空间中的数据并不会复制给子进程，这样创建子进程的速度就很快了！(不用复制，直接引用父进程的物理空间)。
    
  - 启动先用快照恢复，在从aof恢复，加快恢复速度
  
# 哨兵模式(Redis Sentinel)
它负责持续监控主从节点的健康，当主节点挂掉时，自动选择一个最优的从节点切换为主节点。客户端来连接集群时，会首先连接 sentinel，通过 sentinel 来查询主节点的地址，然后再去连接主节点进行数据交互。当主节点发生故障时，客户端会重新向 sentinel 要地址，sentinel 会将最新的主节点地址告诉客户端。如此应用程序将无需重启即可自动完成节点切换。

# 集群模式(RedisCluster)
   - Redis Cluster 将所有数据划分为 16384 的 slots

   - Cluster 默认会对 key 值使用 crc16 算法进行 hash 得到一个整数值，然后用这个整数值对 16384 进行取模来得到具体槽位。
   
   - 故障迁移，对 slots 进行迁移。
   
# codis
   - 分为 1024 个哈希槽，利用代理层计算 key 落在哪个哈希槽。
   
   - zookeep 记录分布式系统的信息
   
   - server 里面分为 group, 也是使用哨兵模式，故障可以进行切换
   
   - 具有联邦的概念，联邦其实就是路由转发的实现
   
   - 将流量经过代理层，计算落在哪个哈希槽上，请求服务对应的group去读取数据。