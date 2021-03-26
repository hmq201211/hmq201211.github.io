---
title: RedisPrinciple  
date: 2021-01-02  
tags: 
- 中间件
- 数据库
- 消息队列
- 分布式锁

categories:
- 数据库
---


# 线程IO模型
  
## 非阻塞IO
- 阻塞式IO
  1. 读取多少个字节后返回，要是没读取完成，就会阻塞
  2. 写多少字节返回，要是写缓冲区满了，那么写方法就会阻塞，直到缓冲区有空间空出来
- 非阻塞式IO Non_Blocking true
  - 读写方法都不会阻塞
  - 能读多少取决于读缓冲区的已有字节数
  - 能写多少取决于写缓冲区的空闲字节数
  - 读写都会返回已经操作的字节数

## 事件轮询
用来解决线程如何知道何时继续读和写的问题（读缓冲区数据来了，或者写缓冲区有数据空了）
- 例如select方法可以阻塞timeout事件，来监听任何事件，要是有事件到来了会返回，或者超时了也会返回
- epoll比select效率要高

## 指令队列
会为每个客户端套接字关联一个指令队列，客户端的指令通过队列顺序处理

## 响应队列
同样会为每个客户端关联一个响应队列，通过这个队列来返回结果给客户端
- 要是队列为空，需要获取写事件，就把当前的客户端fd从write_fds中移除
- 等有数据了，再把当前的fd加入进去
- 避免select返回了写事件，但是发现没有数据可以写，造成的cpu浪费

## 定时任务
防止响应IO事件时错过定时任务，Redis使用了最小堆来处理
- 每个循环周期中，对最小堆中要到期的时间点的任务进行处理
- 将最快要执行的任务还需要等待的时间记录下来，作为selct调用的timeout
  
# 通讯协议
Redis使用了文本协议
  
## RESP Redis序列化协议
5种最小单元类型，每个单元结束后加上回车换行符
- 单行字符串以+开头
- 多行字符串以$开头加上字符串的长度
- 整数以：开头
- 错误信息以-开头
- 数组以*开头加上数组的长度

# 持久化

## 快照 RDB
- 全量备份
- 内存数据的二进制序列化
- 在进行rdb持久化的时候调用系统的glibc的函数fork来产生子进程，由子进程完全负责持久化的进行
- 子进程和父进程完全共享代码段和数据段
- 采用copy on write的机制，父进程在对内存进行修改的时候复制出要修改的内存的那一页，这样最多有2倍的数据内存大小
- 子进程在进程产生的一瞬间，数据就不会变化，只需要遍历持久化就行
  
## AOF
- 存储的是顺序指令序列，只存修改的操作
- 恢复的时候通过重放这些指令来实现数据恢复
- 但是随着Redis的运行，AOF会越来越大，AOF恢复会很耗时
- AOF重写 bgrewriteaof
  - 开辟一个新的子进程对内存进行遍历
  - 转化成一系列的Redis的操作指令
  - 序列化到一个新的aof日志文件中
  - 再把这段操作时间中的增量AOF日志追加到这个文件中
- fsync
  - AOF日志是文件形式，当程序对日志进行写操作实际是把内容写到了内核为文件描述符分配的一个内存缓存里面，内核会异步将脏数据刷回硬盘
  - 要是突然宕机这时日志就会丢失
  - glibc中的fsync函数可以将指定文件的内容强制刷回硬盘
  - 但是这是IO操作，很慢，实际生产中一般1s执行一次fsync
  - ***sync方法是将修改过的快缓存区排入写队列，不管磁盘操作是否执行完成，直接返回***
- 从机进行持久化，这是因为从机没有客户端请求的压力，但是要注意数据不一致的情况
- 混合持久化：RDB恢复加AOF增量日志重放

# 管道
管道并不是服务器提供的特有的技术，而是客户端提供的  
2次指令对应请求为写读写读，为2次网络请求， 通过调整指令顺序为写写读读后，减少为1次网络请求

## 压测命令
redis-benchmark -t set -q

## 深入了解管道本质
- write操作将数据写入内核写缓冲区然后就返回了，要是缓冲区满了才阻塞；  
- read操作从内核读缓冲区读数据，要是缓冲区没数据才阻塞； 
- 对于管道来说，连续的写操作根本不耗时，第一个读操作会等待一个网络请求的时间，之后的读操作都是从内核的读缓冲中取数据
  
# 事务（弱事务）

## 基本用法
- multi 事务开始
- exec 事务执行
- discard 事务丢弃

```shell
RDM Redis Console
Connecting...
Connected.
101.200.121.40:0>multi
"OK"
101.200.121.40:0>incr books
"QUEUED"
101.200.121.40:0>incr books
"QUEUED"
101.200.121.40:0>exec
1) "OK"
2) "1"
3) "OK"
4) "2"
5) "OK"
101.200.121.40:0>
```
  
所有指令在exec之前不执行，只是缓存在事务队列中
  
## ACID
不具备原子性（要么全部成功，要么全部失败），但具备隔离性（当前执行的事务不被其他事务打断）

```shell
RDM Redis Console
Connecting...
Connected.
101.200.121.40:0>multi
"OK"
101.200.121.40:0>set book string
"QUEUED"
101.200.121.40:0>incr book
"QUEUED"
101.200.121.40:0>set node yes
"QUEUED"
101.200.121.40:0>exec
1) "OK"
2) "OK"
3) "OK"
4) "ERR value is not an integer or out of range"
5) "OK"
6) "OK"
7) "OK"
101.200.121.40:0>get book
"string"
101.200.121.40:0>get node
"yes"
101.200.121.40:0>
```

## discard 丢弃
在exec执行之前，作用是丢弃事务缓存队列里面的所有指令

```shell
RDM Redis Console
Connecting...
Connected.
101.200.121.40:0>get books
null
101.200.121.40:0>multi
"OK"
101.200.121.40:0>incr books
"QUEUED"
101.200.121.40:0>incr books
"QUEUED"
101.200.121.40:0>discard
"OK"
101.200.121.40:0>get books
null
101.200.121.40:0>
```
 
## 优化
考虑到事务需要执行的指令比较多，那么可以考虑使用管道来减少IO操作
  
## watch
watch会在事务开始前盯住一个或者多个关键变量，在exec执行之前会检查自watch之后是否修改了，如果修改了，会给客户端返回失败  
利用这种机制可以实现乐观锁  
Redis禁止在multi和exec之间执行watch，必须在multi之前

```shell
RDM Redis Console // shell 1
Connecting...
Connected.
101.200.121.40:0>flushdb
"OK"
101.200.121.40:0>set books 100
"OK"
101.200.121.40:0>watch books
"OK"
101.200.121.40:0>multi 
"OK"
101.200.121.40:0>incr books
"QUEUED"
101.200.121.40:0>exec

101.200.121.40:0>get books
"500"
101.200.121.40:0>
```
-----------------
```shell
RDM Redis Console // shell 2 在shell 1 执行exec之前执行修改watch的值
Connecting...
Connected.
101.200.121.40:0>set books 500
"OK"
101.200.121.40:0>
```
  
通过watch来模拟乐观锁实现账户余额翻倍

```python
import redis

def key_for(uid):
    return "account_{}".format(uid)


def double_account(redis_client, uid):
    key = key_for(uid)
    while True:
        redis_client.watch(key)
        value = int(redis_client.get(key))
        value *= 2
        pipeline = redis_client.pipeline(transaction=True)
        pipeline.multi()
        pipeline.set(key, value)
        try:
            pipeline.execute()
            break
        except redis.WatchError:
            print("err")
            continue
    return int(redis_client.get(key))


client = redis.StrictRedis(host="101.200.121.40", port=6379, password="++++++++")
user_id = "abc"
client.setnx(key_for(user_id), 10)
print(double_account(client, user_id))
```
  
# PubSub 消息多播
运行生产者生产一次消息，由中间件来负责将消息复制到多个消息队列

## 生产者消费者模型

- 普通消费者
  ```python
  import time
  import redis

  client = redis.StrictRedis(host="101.200.121.40", port=6379, password="+++++++")
  p = client.pubsub()
  p.subscribe("hello")
  while True:
      msg = p.get_message()
      if not msg:
          time.sleep(1)
          continue
      print(msg)
  ```
- 普通生产者
  ```python
  import time
  import redis

  client = redis.StrictRedis(host="101.200.121.40", port=6379, password="")
  client.publish("hello", "1")
  client.publish("hello", "2")
  client.publish("hello", "3")
  ```
- 阻塞消费者
  ```python
  import time
  import redis

  client = redis.StrictRedis(host="101.200.121.40", port=6379, password="H")
  p = client.pubsub()
  p.subscribe("hello")
  for msg in p.listen():
      print(msg)
  ```

## 模式订阅
```shell
psubscribe com.* // 所有以com.*发生的消息都会收到
```

## 消息结构
- data 消息内容 一个字符串
- channel 当前订阅的主题
- type 消息的类型
  - message 普通消息
  - subscribe 订阅
  - psubscribe 模式订阅
  - unsubscribe 取消订阅
  - unpsubscribe 取消模式订阅
- pattern 模式订阅下的匹配规则
  
## 缺点
- 不保证消费者一定能收到消息(例如消费者down了,那么消息就收不到了,如果一个消费者都没有,那么会直接丢弃)
- 自身也没有做持久化,所有消息都被丢弃

  
# 小对象压缩
如果Redis内部管理的数据结构很小，会使用紧凑存储形式压缩，例如以下的双数组（一维）代替hashmap（数组加链表）
  
```python
class ArrayMap:
  def __init__(self):
      self.keys = []
      self.values = []

  def put(self, key, value):
      for i in range(len(self.keys)):
          if key == self.keys[i]:
              old = self.values[i]
              self.values[i] = value
              return old
      self.keys.append(key)
      self.values.append(value)
      return None

  def get(self, key):
      for i in range(len(self.keys)):
          if key == self.keys[i]:
              return self.values[i]
      return None

  def delete(self, key):
      for i in range(len(self.keys)):
          if key == self.keys[i]:
              old = self.values[i]
              self.keys.remove(key)
              self.values.remove(old)
              return old
      return None


collection = ArrayMap()
print(collection.put(1, 2))
print(collection.put(1, 3))
print(collection.put(2, 3))
print(collection.get(1))
print(collection.delete(1))
```

    
## ziplist 紧凑的字节数组结构
如果存储的是hash结构, 那么key和value会作为两个entry相邻存储  
如果存储的是zset结构, 那么value和score会作为两个entry相邻存储
  
- 头部:
  - zlbytes 整个压缩列表占用的字节数 4bytes
  - zltail 最后一个entry的偏移量, 便于直接定位尾部元素 4bytes
  - zllen entry的数量 2bytes
- 中间:
  - entry的紧凑排列
- 尾部:
  -zlend 魔术整数255 1byte

```shell
List of unsupported commands: DUMP, RESTORE, AUTH 
Connecting ...

Connected.
101.200.121.40:0>hset hello a 1
"1"

101.200.121.40:0>hset hello b 2
"1"

101.200.121.40:0>hset hello c 3
"1"

101.200.121.40:0>object encoding hello
"ziplist"

101.200.121.40:0>
```

## intset
紧凑的整数数组结构, 存储元素都是整数,且元素个数较少 

```shell
101.200.121.40:0>sadd hello 1 2 3 
"3"

101.200.121.40:0>object encoding hello
"intset"

101.200.121.40:0>
```

开始是uint16 -> uint32 -> uint64  

头部:
- encoding 表明value的位宽
- length 表明元素的个数
- 后面跟着...value


当set里面村的是字符串, sadd会立即升级为hashtable

```shell
101.200.121.40:0>sadd hello 1 2 3 
"3"

101.200.121.40:0>object encoding hello
"intset"

101.200.121.40:0>sadd hello yes no
"2"

101.200.121.40:0>object encoding hello
"hashtable"

101.200.121.40:0>
```

以下情况会升级为标准结构:
- hash-max-ziplist-entries 512 hash的元素个数超过512就必须用标准结构存储
- hash-max-ziplist-value 64 hash的任意元素的key/value的长度超过64就必须用标准结构存储
- list-max-ziplist-entries 512 list的元素个数超过512就必须用标准结构存储
- list-max-ziplist-value 64 list的任意元素的长度超过64就必须用标准结构存储
- zset-max-ziplist-entries 128 zset的元素个数超过128就必须用标准结构存储
- zset-max-ziplist-value 64 zset的任意元素的长度超过64就必须用标准结构存储
- set-max-intset-entries 512 set的整数元素的个数超过512就必须用标准结构存储
  
## 内存回收机制
操作系统是以页为单位回收内存的,只要有一个key在这一页上并且在使用中,这一页就不能被回收  
Redis虽然无法保证立即回收被删除的key的内存,但是可以保证重新使用那些尚未回收的空闲内存










  
  
  
  
  
  
  
  
  
 
 
 
 
 
 
 
 
 
 



    
    
    
    
    
    
    
    
    
    
    
    
    
    













