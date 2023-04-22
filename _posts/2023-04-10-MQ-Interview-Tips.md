---
layout: post
title:  "消息队列知识盘点"
date:   2023-04-10 09:00:30 +0200
categories: mq
---

消息队列重要知识盘点, 如常见中间件、应用场景、高可用、消息可靠性、消息幂等


## 消息队列是什么
MQ是一种应用间的通信方式, 包括producer、broker、consumer三部分
* Producer: 负责生产消息, 即业务信息的承载者——消息的实例化
* Broker: 负责消息存储、投递、及其他功能
* Consumer: 负责从队列取出消息并做各种业务逻辑处理

## 应用场景
### 异步处理
1. 用户注册短信
2. 下单通知、发优惠券

### 应用解耦
1. 订单系统-库存系统
* 传统方式: 订单系统直接调用库存系统, 若库存系统不可用, 订单系统也不可用, 导致下单失败, 两个系统存在耦合
* 同时,订单系统作为上游系统, 每当下游新增业务,加入其他服务(如:营销系统), 订单系统就要修改, 扩展性差
* 引入MQ中间件: 只要统一MQ, 订单系统只管发消息, 其他应用消费即可

### 流量削峰
1. 秒杀业务, 流量会突然暴增, 但应用处理不过来, 可能造成宕机
2. 在应用前加入消息队列, 将用户请求放入消息队列里, 应用从消息队列中取出其可承受范围的请求数量处理

### 消息通讯
### 远程调用

## 常见消息队列中间件
1. ActiveMQ
2. RabbitMQ
3. Kafka
4. RocketMQ

## RabbitMQ
基于AMQP协议实现的分布式消息中间件. 生产者把消息发送到RabbitMQ Broker上的Exchange交换机上. Exchange交换机把收到的消息根据路由规则发给绑定的队列, 最后再把消息投递给订阅了这个队列的消费者, 从而完成消息的异步通讯. 
### 核心组件
1. Product
2. Consumer
3. Broker
4. Connection: TCP长连接, Product生产消息,Consumer消费消息都要和Broker建立连接
5. Channel: 虚拟连接, 在已经连接好的TCP长连接中创建和释放Channel可以减少资源损耗
6. Queue: 在Broker里, 用于存储消息
7. Exchange: 交换机, 负责消息路由, 即根据规则分发消息到不同Queue中
8. Binding: Exchange与Queue之间的绑定关系
9. VHost: 类似于namespace, 解决不同业务系统间的消息隔离

### 工作原理
1. Product与Broker建立连接, 将消息发送给Exchange, 并携带一个特殊标识
2. Exchange与Queue建立绑定关系, 并给每个队列一个特殊标识
3. 当两个标识匹配时, Exchange就会把消息发给符合的Queue
4. Consumer与Broker建立连接, 从Queue取出消息并消费

### 消息路由策略
1. 生产者发送消息给Exchange时携带RoutingKey(路由键)
2. Exchange拿到RoutingKey后与BindingKey(Exchange与Queue建立绑定关系时产生)进行匹配
3. 匹配规则根据ExchangeType决定
* direct: 完全匹配方式
* fanout: 广播方式, 把消息广播给绑定到当前Exchange上的所有队列
* topic: 正则匹配, BindingKey支持2个通配符: #(匹配0或n个单词), *(匹配1个单词); 例如: BindingKey = #.produce/produce.#/*.produce/produce.*

### 消息可靠性保证
1. 丢失情况
* Product生产消息发送到Rabbit Server过程中丢失
* Rabbit Server持久化消失时宕机导致消息丢失
* Consumer收到消息还没处理就宕机, 导致Rabbit Server认为消息已消费
2. 解决
* Confirm(消息确认)机制 + 持久化(创建Queue时设置为持久化/发送消息时设置持久化投递)

### 高可用
#### 普通集群
集群中只有一个节点存储消息队列全部的内容, 其他节点只同步队列的元数据(队列名称、交换机名称及属性、交换机与队列的绑定关系等), 生产和消费消息时不管请求在哪个节点上, 最终都会通过元数据定位到队列所在节点去存取数据. 无法保证队列的高可用, 但提高了MQ的吞吐量
#### 镜像集群
集群中每个节点都存储了消息队列的数据, 能保证队列高可用, 但由于数据副本的同步降低了性能
#### 负载均衡(HAProxy + Keepalived实现)
1. 负载均衡的组件需要满足:
* 本身有负载功能, 可以监控集群中节点的状态, 如果某个节点出现异常或者发生故障,就把它剔除掉
* 能够支持部署多个服务, 能够自主完成Master选举
* Master节点需要对外提供一个虚拟IP, 客户端只需要连接到这一个虚拟IP就可以自动完成真实IP的路由

## MQ如何保证消息不丢失
### Producer保证不丢失
1. 采用同步发送方式, send方法返回成功, 即表示消息到达Broker, 否则重试.
2. 事务消息

### Broker保证不丢失
1. 消息持久化
* 同步刷盘: 生产者一发来消息就持久化到磁盘, 然后返回一个成功的ACK响应
* 异步刷盘: 将消息写入PageCache, 就返回一个成功的ACK响应, 提供了MQ性能, 但还是又丢失可能
* 同步复制: 主从节点都写入成功, 才返回一个成功的ACK响应
* 异步复制: 主节点写入成功, 就返回一个成功的ACK响应

### Consumer保证不丢失
消费成功, 给Broker发送成功反馈

## MQ如何保证幂等,避免重复消费
### 什么是幂等
一个方法被重复调用时所期望的结果和第一次执行所期望的结果一致, 即多次调用也不影响最终一致性
* 重复提交或恶意攻击
* 超时重试机制

### 幂等解决办法
1. 使用数据库唯一约束
2. 使用Redis的setNX指令, 例如将消息用该命令写入Redis, 当该消息消费过就不会再消费
3. 使用状态机, 状态只会向前变更
4. ...


## MQ如何处理消息积压


## MQ技术选型

<table class="table table-bordered table-striped">
	<caption>MQ技术选型</caption>
	<thead>
		<tr>
            <th>对比项</th>
			<th>Kafka</th>
			<th>RocketMQ</th>
			<th>RabbitMQ</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>语言</td>
			<td>Scala/Java</td>
			<td>Java</td>
            <td>Erlang</td>
		</tr>
        <tr>
			<td>维护者</td>
			<td>Apache</td>
			<td>Alibaba</td>
            <td>Mozilla/Spring</td>
		</tr>
		<tr>
			<td>单机吞吐量</td>
			<td>17.3w/s</td>
			<td>11.6w/s</td>
            <td>2.6w/s</td>
		</tr>
        <tr>
			<td>订阅形式</td>
			<td>基于topic, 按照topic进行正则匹配的发布订阅模式</td>
			<td>基于topic/messageTag, 按照消息类型、属性进行正则匹配的发布订阅模式</td>
            <td>direct/topic/headers/fanout模式</td>
		</tr>
		<tr>
			<td>持久化</td>
			<td>支持大量堆积</td>
			<td>支持大量堆积</td>
            <td>支持少量堆积</td>
		</tr>
        <tr>
			<td>顺序消息</td>
			<td>支持</td>
			<td>支持</td>
            <td>不支持</td>
		</tr>
        <tr>
			<td>集群方式</td>
			<td>Leader-Slave无状态集群</td>
			<td>Master-Slave, 需手动切换</td>
            <td>主从复制</td>
		</tr>
        <tr>
			<td>性能稳定性</td>
			<td>较差</td>
			<td>一般</td>
            <td>较好</td>
		</tr>
	</tbody>
</table>

## Kafka
kafka的基础集群架构由多个broker节点组成, 创建一个topic时, 划分为多个partition, 每个partition存放一部分数据, 分别存在于不同的broker上. 并且提供了复制品副本机, 即每个partition都会同步到多台机器上, 形成多个副本. 然后所有副本选出一个leader, 这个leader跟producer和consumer交互, 其他副本都follower. 写数据时, leader负责把数据同步给所有follower, 读数据时, 直接读leader数据. 当某个broker宕机, 这个broker上的partition在其他机器上都有副本.

### 数据存储原理
#### Topic主题
用来存储消息的队列,是一个逻辑的概念,可以理解为一组消息的集合. 生产者和Topic以及Topic和消费者的关系都是多对多
#### Partition分区
为了实现横向扩展, 它会把不同的数据存放在不同的Broker上, 同时为了降低单台服务器的访问压力, 又把一个Topic中的数据分隔成多个Partition, 每个Partition都有一个物理目录
#### Replica副本
为了提高分区的可靠性, 设计了副本机制. 创建Topic的时候,通过指定replication-factor副本因子来确定Topic的副本数. 副本因子数必须小于等于节点数,否则会报错
去了备份的意义
#### Segment分段
为了防止Log不断追加导致文件过大,导致检索消息效率变低, 一个Partition超出一定大小的时候,就被切割为多个Segment来组织数据. 在磁盘上,每个Segment由一个log文件和2个index文件组成
* index是用来存储Consumer的Offset(偏移量)的索引文件
* timeindex是用来存储消息时间戳的索引文件
* log是用来存储具体的数据文件

### Kafka为什么快
1. 磁盘顺序读写: Kafka的Message是不断追加到本地磁盘文件末尾的,而不是随机的写入,使得其写入吞吐量得到了显著提升
2. 稀疏索引: Kafka插入一批消息才会产生一条索引记录. 后续利用二分查找, 可以大大提高检索效率
3. 批量文件压缩: Kafka默认不会删除数据，它会把所有的消息都变成一个批量的文件, 把相同的Key合并为最后一个Value, 减少网络IO损耗
4. 零拷贝机制: Kafka中文件传输最终调用的是NIO库里的transferTo方法,不经过用户态, 提升文件传输性能

### Kafka消息的无序性
在kafka的架构里面, 用Partition分区机制来实现消息的物理存储, 在同一个topic下面,可以维护多个partition来实现消息的分片; 生产者在发送消息的时候, 会根据消息的key进行取模, 来决定把当前消息存储到哪个partition里面, 并且消息是按照先后顺序有序存储到partition里面的. 但消费者消息的消费顺序可能不是按照发送顺序来实现的, 从而导致乱序的问题
### 如何保证消息的有序性
* 一般是自定义消息分区路由的算法，然后把指定的key都发送到同一个Partition里面. 接着指定一个消费者专门来消费某个分区的数据,这样就能保证消息的顺序消费
* 消费端使用阻塞队列, 把获取的线程存放到阻塞队列中, 然后异步线程获取消息进行消费

### Kafka选举Leader原理
早期直接用zookeeper完成选举, 之后换了实现方式. 不是所有repalica都参与leader选举, 而是由其中一个broker统一指挥, 这个broker角色就是controller控制器
1. 先从所有broker中选出唯一controller, 所有broker都会尝试在zk中创建临时节点controller, 谁先创建成功谁就是controller
2. ontroller确定以后, 接下来确定找候选人, 但不是所有的Replica都有竞选资格, 只有在ISR里的副本才有资格(ISR是集合列表, 保存与Leader节点数据最接近的副本. 如果某个副本数量落后Leader太多就会被踢出列表)
3. 确定候选人后, 开始选举, 默认是让ISR中第一个Replica变成Leader

## 事务消息如何实现, MQ如何保证数据一致性
### 普通消息的生产消费流程
1. 生产者产生消息, 发送到MQ服务器
2. MQ服务器收到消息后, 将消息持久化到存储系统, 并返回ACK给生产者
3. MQ服务器把消息push给消费者
4. 消费者消费消息, 并返回ACK给MQ服务器
5. MQ服务器收到ACK, 认为消息消费成功, 从存储系统中删除消息 

### 事务消息
1. 生产者产生消息, 发送一条半事务消息到MQ服务器
2. MQ收到消息后, 将消息持久化到存储系统, 这条消息的状态是待发送状态
3. MQ服务器返回ACK确认到生产者, 此时MQ不会触发消息推送事件
4. 生产者执行本地事务
5. 如果本地事务执行成功, 即commit执行结果到MQ服务器; 如果执行失败, 发送rollback
6. 如果是commit, MQ服务器更新消息状态为可发送; 如果rollback,即删除消息
7. 如果消息状态更新为可发送, 则MQ服务器会push消息给消费者,消费者消费完给MQ发送ACK反馈
8. 如果MQ服务器长时间没有收到生产者commit或rollback,它会反查生产者,然后根据查询到的结果执行最终状态(进行回滚或重发消息)

## 如何设计一个MQ框架
此来问题具有相似性,例如: 如何设计一个mybatis框架等, 需要了解技术框架的基本结构、工作原理
### 思考角度
1. MQ的整体流程
2. RPC设计: 服务发现、序列化协议等
3. Broker持久化问题: 存文件系统还是数据库
4. 消费关系保存问题： 点对点、广播等
5. 消息可靠性问题：消息重复、幂等
6. MQ的高可用问题：多副本、集群
7. 消息事务特性
8. 伸缩性、可扩展性等：消息积压、资源不足、快速扩容、吞吐量等等

## 阻塞队列
### 阻塞队列
是一种特殊队列, 在普通队列基础上提供2个附加功能
* 队列为空, 消费者线程阻塞, 唤起生产者线程
* 队列已满, 生产者线程阻塞, 唤起消费者线程

### 有界无界
1. 有界: 设置有固定大小的队列, 如ArrayBlokingList
2. 无界: 没设置固定大小, 但并不是无限, 而是元素存储量非常大, 一般感知不到, 如LinkedBlokingQueue默认队列长度为Integer.MAX_VALUE
