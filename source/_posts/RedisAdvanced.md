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
- 可以挂载多个消费者组，每个消费者组都有一个last_delivered_id
