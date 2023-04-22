---
layout: post
title:  "Redis知识盘点"
date:   2023-04-07 08:00:30 +0200
categories: redis
---

Redis重要知识盘点, 包括Redis数据类型及应用场景, 缓存的各种问题, 分布式, Redis高可用方案等

## Redis是什么
Redis是一种以Key-Value键值对的形式存储数据的非关系型内存数据库
* 提供5中基本数据类型, 可以覆盖应用开发中大部分的业务场景
* 基于内存存储,通常作为应用与数据库之间的缓存组件, 以减少数据库压力
* 非关系型数据库, 不存在表关联查询问题, 提高了应用程序的数据IO效率
* 提供了几种高可用方案, 应对企业级开发的应用场景
## Redis基本数据类型
### String
1. String是Redis最基础的数据结构类型, 是二进制安全的, 可以存储图片、序列化对象等, 最大存储512M
2. 简单使用: set key value、 get key
3. 内部编码: int/embstr/raw
4. Redis使用SDS封装字符串
* unsigned int len; 标记buf长度
* unsigned int free; 标记buf中未使用元素个数
* char buf[]; 存储元素
5. 应用场景
* 共享session
* 分布式锁
* 计数器: 播放量、浏览量等
* 限流

### Hash
1. Hash类型是指在Key-value的value中存储的是k-v结构
2. 简单使用: hset key field value、hget key field
3. 内部编码: ziplist(压缩列表)、hashtable(哈希表)
4. 应用场景: 缓存用户信息等

### Set
1. Set存储多个字符串元素, 不允许重复元素
2. 简单使用: sadd key element、smembers key
3. 内部编码: intset(整形集合)、hashtable
4. 应用场景: 用户标签、生成随机数抽奖等

### ZSet
1. ZSet存储多个已排序字符串元素, 不允许重复元素
2. 简单使用: zadd key score member、zrank key member
3. 内部编码: ziplist、skiplist(跳跃表)
4. 应用场景: 排行榜(销量榜、投票榜等等)、用户点赞等 
6. 当成员数量小于128个且每个成员的字符串长度都小于64个字节时使用ziplist

### List
1. List存储多个有序字符串, 最大2^32 - 1个元素
2. 简单使用: lpush key value、lrange key start end
3. 内部编码: ziplist、linkedlist(链表)
4. 应用场景: 消息队列、文章列表
* lpush + lpop = 栈
* lpush + rpop = 队列
* lpush + ltrim = 有限集合
* lpush + brpop = 消息队列

## Redis特殊类型
### Geospatial
存储地理位置, 用于地理位置定位, 并对存储的信息进行操作
### Hyperloglog
用来做基数统计算法的数据结构, 如网站UV
### Bitmaps
用一个比特位映射某个元素状态, 底层基于字符串类型实现, 可以看作一个以比特为单位的数组

## Redis为什么快
  <img src= "/assets/files/Redis为什么快.jpg" alt="加载错误" title="Redis为什么快"/>

### 基于内存的存储
内存读写速度比磁盘读写快很多
### 高效的数据结构
1. SDS(简单动态字符串)
* 在字符串长度处理上, O(1)时间复杂度
* 空间预分配: 字符串修改越频繁, 内存分配也越频繁, 就会消耗性能. SDS在修改和空间扩充上会额外分配未使用空间, 减少性能损耗
* 惰性空间回收: SDS在缩短时, 不是回收多余空间, 而是在free中记录多余空间, 后续变更直接使用free中记录的空间, 减少内存分配
2. 字典
* 即哈希表, 所有键值用字典来存储,  O(1)时间复杂度即可取到值
3. 跳跃表
* Redis特有数据结构, 由zskiplist(保存表头/尾节点、长度)和zskiplistNode(保存表节点)两个结构组成.在链表的基础上增加了多级索引提升查询效率. 支持O(logN) - O(n)的时间复杂度. 
### 合理的数据编码
Redis每种基本数据类型都对应多个数据结构,编码. 根据存储数据的类型, 大小选择相应编码
### 合理的线程模型-IO多路复用
#### 概念
核心思想是通过单个线程去监听多个连接, 一旦某个连接就绪, 就通知应用程序, 获取连接进行读写操作
1. IO: 在操作系统中, 数据在内核态和用户态之间的读写
2. 多路: 大部分情况是指多个TCP连接(多个Socket或Channel)
3. 复用: 一个或多个线程处理多个TCP连接, 无需创建和维护过多线程
#### 模型
1. select
* 数据结构: 数组
* 最大连接数: 1024
* FD拷贝: 每次调用select拷贝
* 效率: 轮询, O(n)
2. poll
* 数据结构: 链表
* 最大连接数: 无限制
* FD拷贝: 每次调用poll拷贝
* 效率: 轮询, O(n)
3. epoll
* 数据结构: B+数
* 最大连接数: 无限制
* FD拷贝: FD首次调用epoll_ctl拷贝, 每次调用epoll_wait不拷贝
* 效率: 回调, O(1)

## 缓存异常场景
一般情况下, 用户请求会先从缓存中查找数据, 如果缓存中有, 则直接返回结果; 如果没有,就从数据库中查找数据, 查找到后, 将数据更新到缓存, 并返回结果; 数据库中也没有数据则返回空。缓存就是为了缓解数据库压力, 提高请求响应效率
### 缓存击穿
1. 热点key在某时间点过期, 而此时正好有大量对该key的请求过来, 导致大量请求打到db
2. 解决:
* 使用互斥锁方案: 缓存失效时不立即加载db数据, 而是使用某些带有成功返回的原子操作去操作, 成功的时候再加载db数据并设置缓存, 否则重试获取缓存
* 永不过期
* 做好熔断、降级

### 缓存穿透
1. 请求访问的是一个不存在数据,即缓存和数据库中都没有, 导致每次请求都会打到db
2. 解决
* 数据库中没有就在缓存设置个空值或默认值, 同时设置短暂过期时间
* 对应恶意攻击, 可以在API入口加参数校验, 过滤非法请求, 加鉴权, 加IP黑名单
* 布隆过滤器
  * 结构: 一个很长的二进制向量和一组hash映射函数组成
  * 作用: 检索一个元素是否存在一个集合中, 缺点是可能误判
  * 原理: 将一个集合中的元素通过n个hash散列函数映射到一个长度为x位的数组的不同位置上, 这些位置上的二进制数均设置为1. 若待检查元素通过n个hash散列函数映射后, 发现其在数组的某个位置上的二进制数为1, 则该元素很可能属于集合, 反之一定不属于集合
  * 减少误判: 多加hash函数映射, 降低hash碰撞概率; 增加数组长度, 增大hash函数生成的数据范围, 降低hash碰撞概率

### 缓存雪崩
1. 大量缓存数据在同一时间过期或者Redis宕机, 因此大量请求都打到db上,导致db压力过大甚至宕机
2.  解决:
* 均匀设置缓存过期时间, 如一个较大固定值 + 一个较小随机值设置过期时间
* 建设高可用Redis集群

### 缓存预热
### 缓存降级

## 热点key问题
热点key即访问频率高的key
1. 产生原因: 
* 用户消费数据大于生产数据,如秒杀、热搜等读多写少的场景
* 请求分片集中, 超过单redis服务器性能瓶颈, 如固定名称的key, 因为hash落入同一台服务器, 瞬间访问量极大, 从而产生热点key问题
2. 解决:
* 将热点key分散到不同服务器中
* Redis集群扩容, 增加分片副本, 均衡读取流量
* 使用二级缓存

## 缓存更新策略
### 先更新db, 在更新缓存
1. 缺陷
* 脏数据问题: 线程1更新db -> 线程2更新db -> 线程2更新缓存 -> 线程1更新缓存, 导致db和缓存数据不一致
* 写多读少时: 数据还没读, 但是缓存却频繁更新, 浪费性能
* 写缓存之前要进行复杂计算: 若每次更新db后, 都要进行复杂计算后更新缓存, 也浪费性能

### 先删除缓存, 再更新db
1. 缺陷
* 线程1写db前删除缓存 -> 线程2读缓存, 发现缓存不存在 -> 线程2读db, 并将旧数据写入缓存 -> 线程1将新数据写入db
2. 解决
* 延迟双删: 写请求 -> 先删除缓存 -> 再更新数据库 -> 休眠 -> 再次删除缓存(休眠时间 = 读业务数据的耗时/主从同步时间 + 几百毫秒, 一般为1s)
* 延迟双删导致吞吐量降低, 可以将第二次删除作为异步, 另起线程删除, 此时就不需要休眠

### 先更新db, 再删除缓存
该策略和上述延迟双删都可能出现删除缓存失败问题, 因此引入重试机制
1. 重试机制
* 写请求更新数据库 -> 缓存删除若失败 -> 将失败key加入消息队列 -> 消费消息队列消息, 获取失败key -> 重试删除缓存
2. 读binlog异步删除
* 上述重试机制会对业务代码造成入侵问题, 因此引入订阅程序订阅数据库binlog, 在非业务代码中进行操作
* 写请求更新数据库 -> 数据库将操作写入binlog日志 -> 订阅(ali中间件canal)binlog提取key和操作数据 -> 非业务代码获取该信息进行删除 -> 缓存删除若失败 -> 将失败key加入消息队列 -> 消费消息队列消息, 获取失败key -> 重试删除缓存

## Redis过期策略
### 定时过期
设置key的过期时间, 会创建一个定时器, 到期立即清除key. 对内存友好, 但是会占用大量CPU资源处理过期数据, 从而影响缓存响应时间和吞吐量
### 惰性过期
只有当访问key时, 才会判断该key是否到过期时间, 到过期时间就立即清除. 对CPU友好, 但是如果key已过期,也没有访问, 可能浪费大量内存
### 定期过期
每隔一段时间, 会扫描一定数量的key, 并清除其中过期的key. 通过调整扫描时间间隔和每次扫描限定耗时, 可以在不同情况下使CPU和内存达到最优平衡效果

## Redis内存淘汰策略
当Redis使用的内存达到maxmemory(默认为当前服务器最大内存)参数配置的阈值时,就会根据配置的淘汰策略把访问频率不高的key移除
1. volatile-lru, 从设置了过期时间的key中使用LRU算法(最近最少使用)进行淘汰
2. volatile-lfu, 从设置了过期时间的key中使用LFU算法进行淘汰
3. volatile-random, 从设置了过期时间的key中, 随机淘汰
4. volatile-ttl, 从设置了过期时间的key中,根据过期时间的早晚淘汰, 过期时间越早越优先淘汰
5. allkeys-lru, 从所有key中使用LRU算法(最近最少使用)进行淘汰
6. allkeys-lfu, 从所有key中使用LFU算法进行淘汰
7. allkeys-random, 从所有key中, 随即淘汰
8. noeviction, 默认策略, 当内存不足以容纳新写入数据时, 新写入操作报错

## Redis持久化策略
### RDB(默认)
通过快照方式来实现持久化, 可以在指定的时间间隔或者执行特定命令时将当前系统中的数据保存备份, 以二进制形式写入磁盘
1. 触发命令:
* save命令触发: 同步快照, 即在该命令执行期间, 客户端请求阻塞
* bgsave命令: 异步快照,通过fork一个子进程处理保存工作, 父进程继续响应客户端请求
* redis.config配置自动化
2. 优势
* 数据备份
* 灾难恢复
* 恢复大数据集比AOF快
3. 劣势
* 不能做到实时保存, 可能丢失数据
* fork()可能非常耗时

### AOF
通过存储执行指令实现实时保存, 即客户端执行一个数据变更操作, 它就会把这个执行追加到AOF缓冲区, 然后从缓冲区把数据写入磁盘的AOF文件
1. 触发配置
* always, 每次发送数据修改时就立即记录到磁盘文件, 数据完整性好, 但IO开销大
* everysec, 每秒进行同步
* no, 默认配置, 不使用AOF持久化方案
2. 重写机制
当AOF文件大小达到阈值时,就会把这个文件中的相同指令进行压缩, 只保留最新指令
* 流程: 根据当前Redis内存数据, 重新构建AOF文件 -> 读取数据, 写入新AOF文件 -> 重写完成后用新文件覆盖旧文件
* 重写过程由于比较耗时, 因此由子进程完成, 主进程依然可以处理客户端请求
* 为避免重写过程中, 主进程数据变化, 导致AOF文件与Redis内存中数据不一致, 主进程会把数据变更追加到AOF缓冲区, 等重写完成再把缓冲区内容追加到AOF文件

### 如何选择
1. 数据不能丢失, RDB+AOF混用
2. 只做缓存, 可以容忍数据丢失, 使用RDB

## Redis高可用方案
高可用即数据不丢失、服务不中断。前者可以使用RDB/AOF保证
### 主从复制模式
  <img src= "/assets/files/Redis主从复制.jpg" alt="加载错误" title="Redis主从复制流程"/>

#### 步骤
1. 从库向主库发送psync命令, 准备同步
2. 主库收到命令后响应FULLRESYNC命令(第一次复制采用全量复制), 并带上主库runId和主库复制进度(偏移量)offset
3. 主库执行bgsave命令, 生成RDB文件, 发送给从库
4. 从库加载RDB文件
5. 新来的写操作会记录到replication buffer内存区间里
6. 主库完成RDB发送后, 会把replication buffer中的修改操作发给从库, 从库重新执行这些操作,完成同步

#### 一主多从, 全量复制时主库压力问题
如果每个从库都要和主库进行全量复制, 主库fork进程生成RDB的过程会阻塞主线程处理正常请求, 同时传输过大的RDB文件也会占用主库网络带宽
可以使用**主-从-从模式**解决, 即在从库中再建立主从关系
#### 主从断连
主从库复制完成后,会维护一个网络长连接, 用于主库后续新的写命令传输到从库.
当主从库断开连接后, 主库会把断连期间收到的写命令写入replication buffer, 同时也会写入repl_backlog_buffer缓冲区, 它是一个环形缓冲区, 主库会记录自己写到的位置, 从库会记录自己读到的位置, 主从重连后, 利用缓冲区实现增量复制

#### 主从数据一致性
1. 全量同步: 一般用于第一次建立主从关系,或主从断开时间较久的场景
2. 增量同步: 从库发起同步时, 带上上次同步的offset,然后跟主库的offset比较, 如果相差的数据能够在主库的**积压缓存**中找到, 则只需同步该部分数据
3. 指令同步: 主库输入的指令异步同步给从库

### 哨兵模式
主从模式中一旦主节点故障就不能提供服务, 需要人工将从节点晋升为主节点, 同时通知应用方更新主节点地址, 多数业务场景都不能接受此种故障处理方式

#### 哨兵作用: 监控、自动选主切换、通知
1. 监控: 哨兵是运行在特殊模式下的Redis进程, 哨兵运行期间, 监视所有Redis主从节点, 周期性发送`ping`命令, 检测主从库是否挂掉
* 如果从库没有在规定时间响应, 哨兵就会标记其为下线状态
* 如果主库没有在规定时间响应, 哨兵就会标记其为下线状态, 并开始选主任务
2. 选主任务: 从多个从库中, 选出一个当主库
3. 通知: 选出主库后, 哨兵将新主库连接信息发送给其他从库及其他客户端, 从而将请求发到新主库

#### 哨兵模式示意图
  <img src= "/assets/files/Redis哨兵模式.jpg" alt="加载错误" title="Redis哨兵模式"/>

* 哨兵之间通过发布订阅机制组成集群

#### 哨兵如何判断主库下线
1. 主观下线
* 哨兵进程向主从库发送`ping`命令, 主从库若没有在规定时间(`down-after-milliseconds`选项设置的值)内响应, 哨兵就会把其标记为**主观下线**
2. 客观下线
* 当主库被标记为主观下线时, 则监视该主库的所有哨兵每秒1次的频率来确认主库是否真的进入主观下线, 当**多数**哨兵都认定主库进入主观下线, 则该主库就会被标记为**客观下线**, 以防止误判造成不必要切换开销

#### 哨兵如何选主
  <img src= "/assets/files/Redis哨兵选主.jpg" alt="加载错误" title="Redis哨兵选主"/>

1. 过滤: 从库下线的直接过滤, 从库网络不好总超时的过滤掉
2. 打分: 过滤后再将剩下从库打分
* 从库优先级, 优先级高的打分高, 优先级可通过`slave-priority`配置
* 从库复制进度, 从库复制进度快的打分高
* 从库ID号, 从库ID号小的打分高

#### 由哪个哨兵执行主从切换
1. Leader哨兵负责主从切换
2. Leader哨兵产生条件
* 拿到num(sentinels)/2 + 1的赞成票
* 拿到的票数要大于等于哨兵配置的quorum值
3. 具体流程
由标记客观下线的哨兵发起Leader选举
  <img src= "/assets/files/Redis哨兵Leader选举.jpg" alt="加载错误" title="Redis哨兵Leader选举"/>

* t1时, 哨兵A1判断主库为客观下线，它想成为主从切换Leader, 给自己投票后分别向A2和A3发起投票命令
* t2时, 哨兵A判断主库为客观下线, 它也想成为主从切换Leader, 给自己投票后分别向A1和A2发起投票命令
* t3时, 哨兵A1收到A3发起的投票命令, 但A1已投给自己, 因此对A3时反对票
* t4时, 哨兵A2收到A3发起的投票命令, 因为A2之前未投票, 因此会给第一个向它发出投票请求的哨兵投赞成票
* t5时, 哨兵A2收到A1发起的投票命令, 但A2已投A3, 对A1只能反对票
* t6时, 哨兵A1获得1票, 哨兵A3获得2票, 因此A3成为Leader
* 若因网络等原因都未能选出Leader, 则等待一段时间后会重新选举

#### 选出主库后的故障转移
  <img src= "/assets/files/Redis哨兵模式故障迁移.jpg" alt="加载错误" title="Redis哨兵模式故障迁移"/>

### 集群模式
哨兵模式基于主从模式, 实现读写分离和自动切换, 但其每个节点数据都一样, 浪费内存, 且不好在线扩容。因此引入Redis Cluster(切片集群实现方案), 实现了Redis的分布式存储, 对数据进行分片, 每个Redis节点上的数据不同,实现了数据解耦, 和容量有限问题, 还提供复制和故障转移功能

#### 客户端如何确定访问的数据在哪个Redis节点上
##### 哈希槽(Hash Slot)
1. 一个切片集群分为16384个Slot, 每个进入Redis的键值对根据**key散列**(CRC16算法计算出1个16bit值再对16384取模)分配到这16384个slot中
2. 每个键都属于这16384个slot中的一个, 集群中每个节点都能处理16384个slot
3. 集群中每个节点负责一部分slot

##### MOVED重定向
1. 客户端向节点发送数据读写操作时, 节点会计算出要处理的键属于哪个slot, 并检查是否再该节点上, 如果是就直接执行命令
2. 如果不是, 则节点会向客户端发送MOVED重定向, 返回正确节点ip:port 指引客户端转向正确节点进行重试
  <img src= "/assets/files/Redis集群模式MOVED重定向.jpg" alt="加载错误" title="Redis集群模式MOVED重定向"/>

##### ASK重定向
一般发生于集群伸缩(重新分片), 此时会导致slot迁移, 当去源节点访问时, 但数据已经迁移到新节点, 就会使用ASK重定向
  <img src= "/assets/files/Redis集群模式ASK重定向.jpg" alt="加载错误" title="Redis集群模式ASK重定向"/>

##### 节点对请求处理过程
1. 通过slot映射，检查当前key否存在当前节点
2. 若slot不由自身节点负责,就返回MOVED重定向
3. 若哈希槽确实由自身负责,且key在slot 中，则返回该key对应结果
4. 若key不存在此slot中,检查该slot是否正在迁出
5. 若key正在迁出,返回ASK错误重定向客户端到迁移到目标服务器上
6. 若slot未迁出，检查哈希槽是否导入中
7. 若slot导入中且有ASKING标记,则直接操作,否则返回MOVED重定向

#### 集群节点通信协议: Gossip
每个节点周期性的随机从节点列表选择一些节点, 将本节点信息传递给这些节点, 这些节点收到信息后会做出相同处理, 直到所有节点信息一致
1. 交换的信息包括节点出现故障、新节点加入、主从节点变更信息、slot信息
2. Gossip的消息类型
* ping: 节点每秒会向集群中其他节点发送ping消息,消息中带有自己已知的两个节点的地址、槽、状态信息、最后一次通信时间等
* pong: 当接收到ping、meet消息时,作为响应消息回复给发送方确认消息正常通信,消息中同样带有自己已知的两个节点信息
* meet: 通知新节点加入,消息发送者通知接收者加入到当前集群,meet消息通信正常完成后,节点会加入到集群中并进行周期性ping、pong消息交换
* fail: 当节点判定集群内另一个节点下线时,会向集群内广播一个fail消息,其他节点接收到fail消息之后把对应节点更新为下线状态
3. 每个节点是通过cluster bus(集群总线)与其他节点进行通信的, 通信端口是该节点对外服务端口号+10000

#### 故障转移
1. 故障发现: 通过ping/pong消息实现故障发现, 这个环节包括**主观下线和客观下线**
* 主观下线: 某个节点判断另一节点不可用, 即为下线
  <img src= "/assets/files/Redis集群主观下线.jpg" alt="加载错误" title="Redis集群主观下线"/>
* 客观下线: 集群中多数节点认为某节点不可用, 从而标记该节点真正下线, 如果是持有slot的主节点故障, 则需要故障转移

2. 故障恢复
故障发现后,若下线节点未主节点, 则需要在它的从节点中选一个替换:
* 资格检查: 检查从节点是否具备替换故障主节点的条件
* 准备选举时间: 资格检查通过后,更新触发故障选举时间
* 发起选举: 到了故障选举时间, 进行选举
* 选举投票: 只有持有槽的主节点才有票,从节点收集到足够的选票(大于一半)
* 触发替换主节点操作

#### 为什么slot为16384

## Redis分布式锁

### Redisson原理
解决锁过期释放, 而业务还没执行完的问题. 当线程加锁成功,就启动一个watch dog, 它是一个后台线程, 每10s检查当前线程是否还持有锁, 如果还持有就延长过期时间, 以防提前释放
  <img src= "/assets/files/Redisson.jpg" alt="加载错误" title="Redisson实现流程"/>

### 什么是RedLock算法
如果线程1在主节点上拿到了锁,但加锁的key还没同步到从节点,恰好这时主节点发生故障,一个从节点就会升级为
主节点. 线程2就可以获取同个key的锁，但线程1也已经拿到锁了,锁的安全性就没了. RedLock算法就是为解决此问题.
1. 核心思想
部署多个主节点,以保证它们不会同时宕掉,并且这些主节点是完全相互独立的,相互之间不存在数据同步.同时,需要确保在这多个主节点上, 与在单节点上使用相同方法来获取和释放锁.
2. 步骤
* 获取当前时间(开始获取锁的时间)
* 按顺序向全部主节点请求加锁,客户端设置网络连接和响应超时时间,并且超时时间要小于锁失效时间(假设锁自动失效时间为10s,则超时时间一般在5-50ms之间),假设超时时间为50ms,如果超时,跳过该主节点,尝试下一个主节点
* 客户端使用当前时间减去开始获取锁时的时间,得到获取锁使用的时间.当且仅当超过一半主节点都获得锁,并且使用的时间小于锁失效时间时,锁才获取成功
* 如果取到了锁,key的真正有效时间就变了,需要减去获取锁所使用的时间
* 如果获取锁失败,客户端要在所有主节点上解锁

## Redis事务机制
1. 通过MULTI、EXEC、WATCH等一组命令集合来实现事务机制。事务一次可以执行多个命令, 事务中的命令都被序列化, 在执行过程中, 按照顺序串行化执行队列中的命令, 其他客户端提交的命令不会插入到事务执行命令序列中. **即, 顺序性、一次性、排他性执行一个队列中的一系列命令**
2. 事务流程
* 开启事务(MULTI)
* 命令入队
* 执行事务(EXEC)或撤销事务(DISCARD)
* WATCH命令用于监视key, 在事务执行前, 该key被改动, 事务将被打断

## Redis中Hash冲突
Redis使用一张全局hash表保存所有键值对, hash表有多个hash桶组成, hash桶中entry元素保持key,value指针, 指向实际的键和值
### 解决方式
1. 采用链式hash, 即落在同一hash桶中的多个元素用一个单向链表保存, 元素之间依次使用指针连接. 
2. 通过rehash操作平衡负载因子(单向链表平均长度)
* 扩展: 负载因子较大时, 扩展hash表长度, 加快查询速度
* 收缩: 负载因子较小时, 收缩hash表长度, 减少内存浪费
3. 具体过程: 渐进式rehash

## Redis底层协议
### RESP
实现简单、解析速度快、可读性好

## Redis线程安全问题
### Server端
本身就是线程安全的K-V数据库, 不存在线程安全问题, 虽然Redis6之后又多线程模型, 但该多线程只是用来解决网络IO, 对于指令执行仍然时单线程, Redis Server不存在CPU瓶颈问题, 也就不必多线程执行命令
### Client端
虽然Redis Server指令执行是原子性的, 但多个Redis客户端同时执行多个指令时无法保证原子性

## Redis线上阻塞如何排查
1. 指令阻塞, Redis执行指令是单线程的, 如果让单次执行变慢就会阻塞后续的指令执行
* 获取的数据很多. 比如大数据量下执行keys、hgetall、smembers等指令
* 大Key, 单次查询的的数据过大, 也会导致单次执行变慢, 所以需要拆分大key
2. CPU负荷大, 因为Redis处理命令只用到一个cpu. 所以当cpu超过负荷的时候,也会导致阻塞
* Redis是否有牺牲cpu去换取内存的一些配置, 比如数据类型底层数据结构的配置. 
* 如果没有, 那就是需要加机器
3. 持久化阻塞
* 虽然说Redis的持久化是异步做的, 但是fork子进程的时候很慢, 那么也会导致问题
* AOF刷盘的时候由于磁盘压力, 导致操作时间久, 主线程为了数据一致性会阻塞指令
4. 其他
* 其他的程序跟Redis部署在一台机器, 导致cpu之间的相互影响
* 网络问题, 包括网络是否稳定、是否有延迟、带宽是否足够
* 客户端连接数是否达到了Redis的最大连接数