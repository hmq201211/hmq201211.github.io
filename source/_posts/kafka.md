---
title: kafka  
date: 2020-12-21  
tags:
- Kafka  

categories:
- Kafka
---

# 优势
1. 天生支持分布式
2. 磁盘存储数据 持久化（但是有保存策略，包括时间和空间）
3. 流处理模式

# 为什么选择：
1. 多生产者和多消费者模式
2. 天生持久化
3. 高伸缩性
4. 高性能

# 应用场景：
1. 活动追踪
2. 传递消息
3. 收集指标和日志
4. 提交日志
5. 流处理

# 基础部分
1. 消息: 字符数组
2. 键:  可选项
3. 批次: 提高效率 (权衡: 时间延迟与吞吐量之间的关系)
4. 主题: 数据库中的表
5. 分区: 相当于分表
6. 生产者: 写消息
7. 消费者: 拉取消息
8. 偏移量: 每个分区消费到的index
9. 消费群组: 每个分区只能被消费群组的一个消费者消费, 但是一个消费者可以消费多个分区. 如果消费者down了, 则由其他的消费者分担
10. broker和集群: 集群复制, 消息首先传递到每个Broker的特定主题的特点分区的首领, 然后其他的Broker的相同主题的相同分区获得一份复制, 如果首领节点down了, 则其他节点充当首领
![kafkaBroker与集群][1]

# Kafka对硬件的要求
1. 磁盘：影响最大的是生产者，读写速度和空间大小（每个分区，多个目录，保存几天，几个副本）
2. 内存：页面缓存，影响到消费者性能
3. 网路：影响生产者写入和消费者读取，影响副本的复制
4. CPU：计算压缩，要求不高

# Kafka配置
- broker.id 集群需要配置标示broker的唯一标示
- zookeeper.connect 链接zk的地址，集群可以用‘，’隔开
- zookeeper.connect.timeout.ms 超时时间，毫秒
- log.dirs 数据存储位置, 多个用‘，’隔开, 并且遵循最少使用原则
- num.recovery.threads.per.data.dir 每个目录的恢复线程数量 在启动或者关闭Kafka的时候有用
- auto.create.topics.enable 自动创建主题
- num.partitions 新建主题的默认分区数量，创建时只能比这个数大，不能小
- log.retention.hours 日志保留的小时数，超过会被删除, 跟最后修改时间有关
- log.retention.bytes 每个分区的最大大小，超过的数据会被删除，一般不设置
- log.segment.bytes 日志片段的大小，超过会新建分片
- log.retention.check.interval.ms 检查间隔
- message.max.bytes 消息的最大字节数 消费者和生产者要保持一致, 切不要大于fetch.message.max.bytes.
- fetch.message.max.bytes 消费者所能获取的最大字节数
- delete.topic.enable 主题是否能被删除
- unclean.leader.election.enable 是否运行非完全选举(从机数据不全) 

# 负载均衡
- 消息带key，按照key值来取模均衡
- key值固定，不负载均衡
- key为空，分区器散列

# 生产者Java API
![生产者JavaAPI][2]
失败重试机制：
  1. 连接错误
  2. 首领down了  
  
非失败重试的情况，直接返回异常

发送的方式：
  1. 发送并忘记（还是会重试，但是有时候会丢失消息）
  2. 同步发送
  3. 异步发送
  
生产者参数设置：
- acks：消息确认机制：
  - 1 必须首领也确认
  - 0 不需要确认
  - all 需要首领确认，从机也确认
- batch.size: 批次大小 默认为16384  
- linger.ms: 批次发送前的等待时间，默认为0
- max.request.size: 生产者默认发送请求的最大大小，不能超过message.max.bytes, 否则发送数据丢失
- max.in.flight.requests.per.connection: 由于失败重试机制导致的消息顺序不能保证，将此值设置为1时，可以确保失败重试时，其他消息不能发送 *严重损耗生产者性能，除非非要保证消息顺序一致性*
- bufffer.memory: 生产者内存缓冲区大小  
- retries：失败重试次数，默认为Integer的最大值
- request.timeout.ms: 客户端等待请求响应的最大时间  
- max.block.ms: 最长阻塞时间，超过则抛出异常
- compression.type: 压缩类型 none gzip snappy 默认无压缩
- partitioner.class 自定义分区器的全限定名

自定义序列化器和反序列化器:
- 序列化器需要实现serilizer接口
- 反序列化器需要实现deserilizer接口

# 消费者
消费者群组的意义:
  1. 消费速率的问题 消费者往往比生产者慢

消费者群组订阅规则:  
![消费者群组][3]
  1. 每个分区只能被一个消费者消费
  2. 要是消费者比分区多, 没订阅的消费者会得不到信息
  3. 要是有消费者down了, 没订阅的消费者会替代down了消费者
  4. 多个群组分开看, 互不干扰

提交和偏移量： 
  从分区copy一份消息，然后消费，然后给特殊的consumer_offsets主题发生偏移量消费消息
  
**消费者多线程不安全，需要保证每一个线程中都有一份KafkaConsumer实例**

分区和再均衡:
  1. 消费者发生了变化, down了的消费者会被等待的消费者替代
  
         群组协调器：涉及到群主概念, 第一个加入消费者群组的消费者作为群主, 保留全部分区到组里消费者的映射关系
        
  2. 主题分区变多了的话, 等待的消费者会分配新的分区 会发生Stop The World 
  
         再均衡监听器：再均衡开始之前和消费者停止读取消息之后生效
         分区发生变化, 监听器会调用onPartitionsRevoked方法
         消费者被分配之后, 监听器会调用onPartitionsAssigned方法
         consumer.seek(TopicPartition, Offset) 指定消费者的消费偏移量
  
配置：
  - auto.offset.reset 消费者在读取到一个没有偏移量或者偏移量无效的分区的情况下，如何处理
     1. latest 当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据
     2. earliest 当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
     3. none topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常
  - enable.auto.commit 消费者每次拉取是否自动提交偏移量 默认true
  - max.poll.records 每次拉取返回的记录数量 默认500
  - partition.assignment.strategy 分区分给消费者的策略
    1. Range 连续分区分给同一个消费者 默认生效
    2. RoundRobin 你一个我一个 随机轮询
 
提交偏移量的方式:
  - 自动提交:
    1. 重复消费
    2. 消息丢失
  - 手动提交(同步)
  ```
  enable.auto.commit = false
  consumer.commitSync();
  ```
  - 手动提交(异步)
  ```
  enable.auto.commit = false
  consumer.commitASync(new OffsetCommitCallback(){
    public void onComplete(...){
    ...
    }
  );
  ```
  - 同步和异步组合
  ```
  enable.auto.commit = false
  consumer.commitASync(new OffsetCommitCallback(){
    public void onComplete(...){
    ...
    }
  );
  finally{
  consumer.commitSync();
  }
  ```
  - 特定提交(处理重复消费和数据丢失)
  ```
  new一个map<TopicPartition, OffsetAndMetadata>, 然后业务中可以往这个map中put值, 并采用同步或者异步的方式来进行手动提交
  在业务结束时,再同步提交一次
  finally{
  consumer.commitSync();
  }
  ```
独立消费者:
  没有消费者组的概念, 但是需要消费主题和分区
  ```
  1. consumer.partitionsFor(TOPIC); 获取主题的分区列表
  2. 选取要消费的主题 放入list
  3. consumer.assign(List<TopicPartition>)
  ```
消费者优雅退出:
  其他的线程调用消费者的wakeup方法
 
# 集群
![集群][4]

控制器：
  多个Broker的管理者
复制相关：
  - 首领副本 集群的某个分区的管理者，对接生产者和消费者 分配首领遵循最少使用原则
  - 跟随副本 非首领副本，获取首领副本的拷贝
  - 优先副本 首领down了之后 该副本会优先成为首领
    
        auto.leader.rebalance.enable
        
工作机制：
  - 同步副本： 跟首领副本保持一致，最新，保证数据不丢失
  - 请求队列和响应队列：使用NIO的通信模式
  - 生产者和消费者与集群进行消息发送和消费之前，会先从任意Broker里面获取元数据，并缓存起来，以后发生数据到首领副本所在的Broker
  
        metadata.max.age 元数据最长缓存时间，超过后会刷新 默认30s
  - 生产者批量发送，消费者也是批量消费，或者指定的等待时间到了也会拉取
  - ISR列表: zk维护了一个跟leader信息一致的follower的列表， 用于leader选举
  ![ISR][5]
  ISR加ACKS=all 保证消息不丢失，但是损失效率
  - 消息格式 推荐使用压缩
  ![消息格式][6]
  - 索引：普通索引和时间戳索引，Kafka自己不维护
  - 超时数据清理(压缩)
  同一个key对于的不同value, 在压缩之后只保留这个key的最新value
 
 
[1]: ../../../../images/picture/Broker与集群.png
[2]: ../../../../images/picture/KafkaJavaApi.png
[3]: ../../../../images/picture/KafkaConsumerGroup.png
[4]: ../../../../images/picture/Kafka集群.png
[5]: ../../../../images/picture/ISR.png
[6]: ../../../../images/picture/MessageFormat.png
