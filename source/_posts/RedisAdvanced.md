---
title: RedisAdvanced  
date: 2021-01-09  
tags: 
- 中间件
- 数据库
- 消息队列
- 分布式锁
categories:
- 数据库
---

# Stream

## 基本构成
- 消息链表， 每个消息都有唯一的id和内容，并且是持久化的
- 每个消息链表都有名称，即为redis的key，首次使用xadd的时候自动创建
- 可以挂载多个消费者组，每个消费者组都有一个last_delivered_id，表示该消费者组消费到什么地方了
- 消费者组不会自动创建，需要指令xgroup create来创建，并指定开始的消息id来初始化last_delivered_id
- 每个消费者组的状态都是独立的，互不影响
- 同一个消费者组里的消费者是竞争关系，任意一个消费者读取了消息都会使得游标last_delievred_id移动，每个消费者在消费者组里都是唯一的
- 消费者内部有一个状态变量pending_ids（PEL），记录了当前已被客户端读取但是没有ack的消息，它确保了客户端至少消费了信息一次，而不会在网路传输中丢失了而没有被处理

## 消息ID
- 形式：timestampInMillis-sequence 表示为时间的毫秒数-该毫秒内的第几条消息
- 也可以自定义，但是需要保证后面的消息比前面的消息大

## 消息内容
hash的键值对

## 增删改查
- xadd 向Stream中追加信息
- xdel 从Stream中删除信息，至少把标志位设置为删除，但是不影响消息的总长度
- xrange 获取Stream中的消息，自动过滤已被删除的消息
- xlen 获取Stream消息长度
- del 删除整个消息列表的所有消息
- xread 把stream当成普通的消息队列（list）来处理，忽略了消费组的存在（用户自己记住消费到哪里了）
  - block 0 表示永远阻塞直到消息到来，block 1000 表示只阻塞1s，要是没有消息返回nil
- xinfo 查看消费者组的信息
- xreadgroup 可以进行消费者组的组内消费
- xack 通知服务器，消息处理完毕，该id会从PEL中移除

## Stream消息过多的问题
xadd提供了maxlen定长参数，就可以将老的消息干掉，确保链表不超过指定长度

## 消息如果忘记ACK
会导致PEL的内存占有增大

## PEL如何保证消息不丢失
PEL只能够保存了已经发出去的消息ID，只要客户端重启后xreadgroup其实消息ID是任意有效的消息ID，一般设置成0-0，表示读取所有的PEL消息和last_delivered_id之后的消息

## Stream高可用
也是建立在主从复制的基础上，因为指令复制是异步的，所以在故障转移时可能会丢失极小部分数据

## 分区
原生不支持分区，想要分区，必须分配多个Stream

# Info 获取Redis内部的运行参数
- 包括
  - Server 服务器相关
  - Clients 客户端相关
  - Memory 服务器运行内存统计数据
  - Persistence 持久化信息
  - Stats 通用统计数据
  - Replication 主从复制相关信息 
  - CPU CPU的使用情况
  - Cluster 集群信息
  - KeySpace 
  
  
  
  
  
  
  
  
  



















