---
title: Redis  
date: 2020-12-28  
tags: 
- 中间件
- 数据库
- 消息队列
- 分布式锁
categories:
- 数据库
---

# 5种基础的数据结构：
  - string：内部表示为一个字符数组，而且是动态的字符串，可以修改的，字符串最大长度为512MB
    > 键值对

        RDM Redis Console
        连接中...
        已连接。
        101.200.121.40:0>set name codehole
        "OK"
        101.200.121.40:0>get name
        "codehole"
        101.200.121.40:0>exists name
        "1"
        101.200.121.40:0>del name 
        "1"
        101.200.121.40:0>get name
        null
        101.200.121.40:0>getrange name 1 2 // 获取字符串的字串
        "am"

    > 批量键值对

        RDM Redis Console
        连接中...
        已连接。
        101.200.121.40:0>set name codehole 
        "OK"
        101.200.121.40:0>set name1 holycoder
        "OK"
        101.200.121.40:0>mget name name1 name2
        1) "codehole"
        2) "holycoder"
        3) null
        101.200.121.40:0>mset name1 boy name2 girl name3 unknown
        "OK"
        101.200.121.40:0>mget name1 name2 name3
        1) "boy"
        2) "girl"
        3) "unknown"
        101.200.121.40:0>

    > 过期和set命令拓展

        RDM Redis Console
        连接中...
        已连接。
        101.200.121.40:0>set name codehole
        "OK"
        101.200.121.40:0>get name
        "codehole"
        101.200.121.40:0>expire name 5
        "1"
        101.200.121.40:0>get name
        "codehole"
        101.200.121.40:0>get name
        null
        101.200.121.40:0>setex name 5 codehole
        "OK"
        101.200.121.40:0>get name
        "codehole"
        101.200.121.40:0>get name
        "codehole"
        101.200.121.40:0>get name
        "codehole"
        101.200.121.40:0>get name
        "codehole"
        101.200.121.40:0>get name
        null
        101.200.121.40:0>setnx name codehole
        "1"
        101.200.121.40:0>get name
        "codehole"
        101.200.121.40:0>setnx name holycoer
        "0"
        101.200.121.40:0>get name
        "codehole"
        101.200.121.40:0>

    > 计数 如果value是整数 可以对他进行自增操作，范围是 signed long

        RDM Redis Console
        连接中...
        已连接。
        101.200.121.40:0>set age 30
        "OK"
        101.200.121.40:0>incr age 
        "31"
        101.200.121.40:0>incrby age 50
        "81"
        101.200.121.40:0>incrby age -55
        "26"
        101.200.121.40:0>set codehole 9223372036854775807
        "OK"
        101.200.121.40:0>incr codehole
        "ERR increment or decrement would overflow"
        101.200.121.40:0> 

  - list：注意是双向链表而不是数组，当列表弹出了最后一个元素后，数据结构会被自动删除，内存会回收，可以作为异步队列，需要处理的数据塞进列表，另一个线程从这个列表轮询数据进行处理
    > 右进左出 队列

        RDM Redis Console
        连接中...
        已连接。
        101.200.121.40:0>rpush books python java golang
        "3"
        101.200.121.40:0>lpop books
        "python"
        101.200.121.40:0>lpop books
        "java"
        101.200.121.40:0>lpop books
        "golang"
        101.200.121.40:0>lpop books
        null
        101.200.121.40:0>
    
    > 右进右出 栈
    
        RDM Redis Console
        连接中...
        已连接。
        101.200.121.40:0>rpush books java python golang
        "3"
        101.200.121.40:0>rpop books
        "golang"
        101.200.121.40:0>rpop books
        "python"
        101.200.121.40:0>rpop books
        "java"
        101.200.121.40:0>rpop books
        null
        101.200.121.40:0>
    
    > 慢操作
    > > lindex 相当于Java链表的get（int index） 进行了列表遍历  
    > > ltrim 保留start_index和end_index之间的值  
    > > lrange 获取范围元素
    
        RDM Redis Console
        连接中...
        已连接。
        101.200.121.40:0>rpush books python java golang
        "3"
        101.200.121.40:0>lindex books 1
        "java"
        101.200.121.40:0>lrange books 0 -1
        1) "python"
        2) "java"
        3) "golang"
        101.200.121.40:0>ltrim books 1 -1
        "OK"
        101.200.121.40:0>lrange books 0 -1
        1) "java"
        2) "golang"
        101.200.121.40:0>ltrim books 1 0 // 清空了整个列表 因为区间长度为负数了
        "OK"
        101.200.121.40:0>lrange books 0 -1

        101.200.121.40:0>llen books // 获取列表的长度
        "0"
        101.200.121.40:0>
      
    > 快速列表
      
        在列表元素比较少的情况下，使用连续内存，即ziplist，压缩列表  
        数据量多的时候改成quicklist，就是链表和ziplist的结合
  
  - hash：
    1. 无序字典，字典的值只能是字符串，rehash的过程中，redis为了追求性能采用了渐进式rehash，即同时保留两个hash结构，在后续的定时任务和hash指令的操作中一点点迁移数据，完成后新的hash结构取代老的hash结构
    2. 移除hash最后一个元素后，数据结构自动删除，内存回收
    3. 缺点是浪费网路流量和存储消耗
    
    ```
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>hset books java "thinking in java"
    "1"
    101.200.121.40:0>hse
    "ERR unknown command 'hse'"
    101.200.121.40:0>hset books golang "concurrency in go"
    "1"
    101.200.121.40:0>hset books python "python cookbook"
    "1"
    101.200.121.40:0>hgetall books // 获取所有的key value
    1) "java"
    2) "thinking in java"
    3) "golang"
    4) "concurrency in go"
    5) "python"
    6) "python cookbook"
    101.200.121.40:0>hlen books
    "3"
    101.200.121.40:0>hget books java
    "thinking in java"
    101.200.121.40:0>hset books golang "learn go programming"
    "0"
    101.200.121.40:0>hget books golang
    "learn go programming"
    101.200.121.40:0>hmset books java "effective java" python "learning python" golang "modern golang programming" // 批量set
    "OK"
    101.200.121.40:0>hgetall books
    1) "java"
    2) "effective java"
    3) "golang"
    4) "modern golang programming"
    5) "python"
    6) "learning python"
    101.200.121.40:0>hset user age 19
    "1"
    101.200.121.40:0>hincrby user age
    "ERR wrong number of arguments for 'hincrby' command"
    101.200.121.40:0>hincrby user age 1 // 对hashmap的键值对的数字value自增
    "20"
    101.200.121.40:0>
    ```
  - set：
    - 无序 唯一
    - 相当于hashmap 但是所有的value都是null
    - 最后一个元素被移除，数据结构自动删除，内存回收
    
    ```
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>sadd books python
    "1"
    101.200.121.40:0>sadd books python
    "0"
    101.200.121.40:0>sadd books java golang
    "2"
    101.200.121.40:0>smembers books // 获取所有元素
    1) "golang"
    2) "java"
    3) "python"
    101.200.121.40:0>sismember books java // 元素是否在set中
    "1"
    101.200.121.40:0>SISMEMBER books rust
    "0"
    101.200.121.40:0>scard books // 获取set的长度
    "3"
    101.200.121.40:0>spop books // 弹出一个
    "golang"
    101.200.121.40:0>
    ```
  - zset：有序列表 
    - set 保证了唯一性
    - 给每个value赋予了score作为排序权重，保证了有序性
    - set的最后一个value被移除后，数据结构自动删除，内存回收
  
    ```
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>zadd books 9.0 java
    "1"
    101.200.121.40:0>zadd books 8.9 python
    "1"
    101.200.121.40:0>zadd books 8.8 golang
    "1"
    101.200.121.40:0>zrange books 0 1 // 按score排序 参数区间为排名范围
    1) "golang"
    2) "python"
    101.200.121.40:0>zrange books 0 0
    1) "golang"
    101.200.121.40:0>zrange books -1 -1
    1) "java"
    101.200.121.40:0>zrevrange books 0 -1 // 按score逆序排序 参数区间为排名范围
    1) "java"
    2) "python"
    3) "golang"
    101.200.121.40:0>zcard books
    "3"
    101.200.121.40:0>zscore books java // 获取value的分数
    "9"
    101.200.121.40:0>zrank  books java // 获取value的排名
    "2"
    101.200.121.40:0>zrangebyscore books 8.88 9.0 // 根据分数的区间来遍历zset
    1) "python"
    2) "java"
    101.200.121.40:0>zrangebyscore books -inf 9.0 withscores // 根据分数区间遍历zset 同时返回分数值
    1) "golang"
    2) "8.8000000000000007"
    3) "python"
    4) "8.9000000000000004"
    5) "java"
    6) "9"
    101.200.121.40:0>zrem books java // 删除value
    "1"
    101.200.121.40:0>zrange books 0 -1
    1) "golang"
    2) "python"
    101.200.121.40:0>
    ```
    > 跳跃列表：列表分成多层，所以每个元素可能身兼数职，通过层级的帮助定位来快速查找
    
  - 容器型数据结构（list set hash zset）规则：
    1. create if not exists
    2. drop if no elements
  - 过期时间：
    1. 以对象为单位，而不是以key为单位
    2. 如果字符串设置了过期时间，然后通过set方法修改了，则过期时间消失
    
    ```
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>set codehole yoyo
    "OK"
    101.200.121.40:0>expire codehole 600
    "1"
    101.200.121.40:0>ttl codehole // 生存时间值
    "595"
    101.200.121.40:0>set codehole yoyo
    "OK"
    101.200.121.40:0>ttl codehole
    "-1"
    101.200.121.40:0>
    ```
    
# 分布式锁
  常用于解决逻辑处理的并发问题（多个线程的非原子性操作，例如get ++ set）
  1. setnx lock true 当lock不存在时候来占坑，业务执行完成释放锁，但是可能有问题，即删除指令没有执行，造成死锁
  2.  

         setnx lock true
         expire lock time
         setnx和expire是非原子操作，要是expire也没有执行，则也会造成死锁
  3. redis2.8以后加入set lock true ex time nx 命令来保证了原子性 但是超时问题还是存在
  4. 可重入锁，对set方法，并引入threadlocal变量存储当前线程的持有锁的计数
  5. 锁冲突问题：
    1. 直接抛出异常，用户来决定什么时候来重试
    2. sleep一会，然后重试
    3. 请求转移到消息队列，然后等一会再重试
  
# 消息队列
  redis的list或者zset都可以拿来作单组消费者的消息队列，但是需要注意，没有ack等机制的保证
  ```shell
  RDM Redis Console
  连接中...
  已连接。
  101.200.121.40:0>llen fruit
  "3"
  101.200.121.40:0>rpush fruit orange 
  "4"
  101.200.121.40:0>llen fruit
  "4"
  101.200.121.40:0>lpop fruit
  "apple"
  101.200.121.40:0>llen fruit
  "3"
  101.200.121.40:0>lpop fruit
  "banana"
  101.200.121.40:0>llen fruit
  "2"
  101.200.121.40:0>lpop fruit
  "pear"
  101.200.121.40:0>llen fruit
  "1"
  101.200.121.40:0>lpop fruit
  "orange"
  101.200.121.40:0>llen fruit
  "0"
  101.200.121.40:0>lpop fruit
  null
  101.200.121.40:0>llen fruit
  "0"
  101.200.121.40:0>
  ```
  问题：
    1. 队列空了：需要线程sleep
    2. sleep导致消息延迟增大->阻塞读取 blpop/brpop key timeout
      ```
      101.200.121.40:0>blpop fruit 1000
      1) "fruit"
      2) "apple"
      101.200.121.40:0>
      ```
  延时队列：
    通过zset来实现，消息序列化为一个字符串，作为value，到期处理时间作为score  
    同时使用多个线程来处理，为了保证可用性，关键点在于zrem返回的是1表示当前线程抢到了消息

# 位图Bitmap：
  通过bit这个最小单位来保存数据，注意位数组的高低位顺序和字符的高低位顺序是相反的
  - 零存整取
    ```shell
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>setbit string 1 1
    "0"
    101.200.121.40:0>setbit string 2 1
    "0"
    101.200.121.40:0>setbit string 4  1
    "0"
    101.200.121.40:0>setbit string  9 1
    "0"
    101.200.121.40:0>setbit string 10 1
    "0"
    101.200.121.40:0>setbit string 13 1
    "0"
    101.200.121.40:0>setbit string 15 1
    "0"
    101.200.121.40:0>get string
    "he"
    101.200.121.40:0>
    ```
  - 零存零取
    ```shell
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>setbit s 1 1
    "0"
    101.200.121.40:0>setbit s 2 1
    "0"
    101.200.121.40:0>setbit s 4 1
    "0"
    101.200.121.40:0>getbit s 1
    "1"
    101.200.121.40:0>getbit s 2
    "1"
    101.200.121.40:0>getbit s 4
    "1"
    101.200.121.40:0>getbit s 5
    "0"
    101.200.121.40:0>
    ```
  - 整存零取
    ```shell
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>set w h
    "OK"
    101.200.121.40:0>getbit w 1
    "1"
    101.200.121.40:0>getbit w 2
    "1"
    101.200.121.40:0>getbit w 4
    "1"
    101.200.121.40:0>getbit w 5
    "0"
    101.200.121.40:0>
    ```
  - 统计和查找
    - bitcount 用来统计一定范围内的1的个数
    - bitpos 但是很遗憾的是这是字节索引
    ```shell
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>set sign sign
    "OK"
    101.200.121.40:0>bitcount sign
    "19"
    101.200.121.40:0>bitcount sign 0 0
    "5"
    101.200.121.40:0>bitcount sign 0 1
    "9"
    101.200.121.40:0>bitpos sign 1
    "1"
    101.200.121.40:0>bitpos sign 0
    "0"
    101.200.121.40:0>bitpos sign 1 1
    "9"
    101.200.121.40:0>bitpos sign 0 1
    "8"
    101.200.121.40:0>bitpos sign 0 1 1
    "8"
    101.200.121.40:0>bitpos sign 1 1 1
    "9"
    101.200.121.40:0>
    ```
  - 魔法指令bitfield 一次操作多个位
    有三个子指令 get、set、incrby 但是只能最多处理64个位 超过64位只能使用多个子指令
    ```shell
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>set w hello
    "OK"
    101.200.121.40:0>bitfield w get u4 0  // 从index0开始取4位作为无符号数
    1) "6"
    101.200.121.40:0>bitfield w get u3 2 // 从index2开始取3位作为无符号数
    1) "5"
    101.200.121.40:0>bitfield w get i4 0 // 从index0开始取4位作为有符号数
    1) "6"
    101.200.121.40:0>bitfield w get i3 3 // 从index3开始取3位作为有符号数
    1) "2"
    101.200.121.40:0>bitfield w get i3 2 // 从index2开始取3位作为有符号数
    1) "-3"
    101.200.121.40:0>
    ```
    > 一次执行多个子指令
    ```shell
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>bitfield w get u4 0 get u3 2 get i4 0 get i3 2
    1) "6"
    2) "5"
    3) "6"
    4) "-3"
    101.200.121.40:0>
    ```
    > 使用set子指令来将第二个字符改成a
    ```shell
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>bitfield w set u8 8 97 // 从index8开始将97的无符号数替换接下来的8个位置
    1) "101"
    101.200.121.40:0>get w
    "hallo"
    101.200.121.40:0>
    ```
    > incrby 指定范围自增，默认溢出策略是折返，即将溢出的符号位丢掉
    ```shell
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>set w hello
    "OK"
    101.200.121.40:0>bitfield w incrby u4 2 1
    1) "11"
    101.200.121.40:0>bitfield w incrby u4 2 1
    1) "12"
    101.200.121.40:0>bitfield w incrby u4 2 1
    1) "13"
    101.200.121.40:0>bitfield w incrby u4 2 1
    1) "14"
    101.200.121.40:0>bitfield w incrby u4 2 1
    1) "15"
    101.200.121.40:0>bitfield w incrby u4 2 1 // 这里溢出了
    1) "0"
    101.200.121.40:0>get w
    "@ello"
    101.200.121.40:0>
    ```
    > overflow子指令 可以选自溢出行为 默认为wrap折返，可选fail失败不执行、sat饱和截断（保持最大值），命令只生效一次，下一次就变成了默认的wrap  
    > sat
    ```shell
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>set w hello
    "OK"
    101.200.121.40:0>bitfield w overflow sat incrby u4 2 1
    1) "11"
    101.200.121.40:0>bitfield w overflow sat incrby u4 2 1
    1) "12"
    101.200.121.40:0>bitfield w overflow sat incrby u4 2 1
    1) "13"
    101.200.121.40:0>bitfield w overflow sat incrby u4 2 1
    1) "14"
    101.200.121.40:0>bitfield w overflow sat incrby u4 2 1
    1) "15"
    101.200.121.40:0>bitfield w overflow sat incrby u4 2 1
    1) "15"
    101.200.121.40:0>bitfield w overflow sat incrby u4 2 1 // 截断 保持最大值
    1) "15"
    101.200.121.40:0>bitfield w overflow sat incrby u4 2 1 // 截断 保持最大值
    1) "15"
    101.200.121.40:0>
    ```
    > fail
    ```shell
    RDM Redis Console
    连接中...
    已连接。
    101.200.121.40:0>set w hello
    "OK"
    101.200.121.40:0>bitfield w overflow fail incrby u4 2 1
    1) "11"
    101.200.121.40:0>bitfield w overflow fail incrby u4 2 1
    1) "12"
    101.200.121.40:0>bitfield w overflow fail incrby u4 2 1
    1) "13"
    101.200.121.40:0>bitfield w overflow fail incrby u4 2 1
    1) "14"
    101.200.121.40:0>bitfield w overflow fail incrby u4 2 1
    1) "15"
    101.200.121.40:0>bitfield w overflow fail incrby u4 2 1 // 失败 不执行
    1) null
    101.200.121.40:0>
    ```
    
# HyperLogLog
  提供不精确的去重计数方案，标准误差0.81%
  ```shell
  RDM Redis Console
  连接中...
  已连接。
  101.200.121.40:0>pfadd count user1
  "1"
  101.200.121.40:0>pfadd count user2
  "1"
  101.200.121.40:0>pfcount count
  "2"
  101.200.121.40:0>
  ```
  ```python
  import redis

  client = redis.StrictRedis(host="101.200.121.40", port=6379, password="*********")

  for i in range(1000):
      client.pfadd("user_count", "user-%d" % i)
      total = client.pfcount("user_count")
      if total != i + 1:
          print(total, i + 1)
          break
  ```
  - pfmerge destination-key source-keys 合并多个pf计数器

# Bloom Filter 布隆过滤器
  说某个值存在时，这个值可能不存在；说某个值不存在时，这个值一定不存在
  - bf.add key memeber 向过滤器中添加元素 bf.madd 添加多个
  - bf.exists 查看元素是否存在 bf.mexists 检查多个是否存在
  - bf.reserve 可以在add之前就创建自定义布隆过滤器
    - key 布隆过滤器的key
    - error_rate 错误率 设置的越低，需要的空间越大
    - initial_size 预计存入的数量，实际数量超过预设的时候，误判率会升高；但是设置的过大会浪费存储空间
  - add流程，多个hash方程，value值对这几个hash计算后取数组长度的模，然后在数组相应的几个位置设置成1
  - exist流程 同样计算数组的这几个位置，要是存在0的，那一定不存在；要是都是1，那么有可能不存在，因为可能其他的value把这些位置设置成了1
  - 应用场景：
    1. 爬虫
    2. nosql数据库过滤不存在的row
    3. 邮箱的垃圾邮件
  
# 简单限流：
  定宽的滑动窗口，设置key为userId加Action，value为唯一性就行，score为时间戳，获取窗口内的次数，如果超过了max则限流
  ```python
  import time
  import redis

  client = redis.StrictRedis(host="101.200.121.40", port=6379, password="********")


  def is_action_allowed(user_id, action_id, period, limits):
      key = 'history:%s:%s' % (user_id, action_id)
      now_ts = int(time.time() * 1000)
      with client.pipeline() as pip:
          pip.zadd(key, {now_ts: now_ts})
          pip.zremrangebyscore(key, 0, now_ts - period * 1000)
          pip.zcard(key)
          pip.expire(key, period + 1)
          _, _, count, _ = pip.execute()
      return count <= limits


  for _ in range(200):
      print(is_action_allowed("user1", "get", 60, 10))
  ```
  
# 漏斗限流
  - capacity 漏斗的总容量
  - capacity_left 漏斗的可用容量
  - losing_rate 漏斗的水流失速率
  - last_check_time 上次检查时间
  - 主要思想是漏斗一直漏水，要是空出来的剩余空间可以支持一次请求，那么这次请求被允许，要是不够，这次请求就会被拒绝
  
  ```python
  import time
  
  class Funnel:
      def __init__(self, capacity, losing_rate):
          self.capacity = capacity
          self.capacity_left = capacity
          self.losing_rate = losing_rate
          self.last_check_time = time.time()

      def make_space(self):
          now_time = time.time()
          time_passed = now_time - self.last_check_time
          capacity_lost = self.losing_rate * time_passed
          if capacity_lost < 1:
              return
          self.capacity_left += capacity_lost
          self.last_check_time = now_time
          if self.capacity_left > self.capacity:
              self.capacity_left = self.capacity

      def watering(self, volume):
          self.make_space()
          print('capacity_left:{%s}, volume:{%s}' % (self.capacity_left, volume))
          if self.capacity_left >= volume:
              self.capacity_left -= volume
              return True
          return False


  gateway = {}

  fixed_capacity = 10
  fixed_losing_rate = 10000


  def is_action_allowed(user_id, api, volume):
      key = '%s:%s' % (user_id, api)
      funnel = gateway.get(key)
      if not funnel:
          funnel = Funnel(fixed_capacity, fixed_losing_rate)
          gateway[key] = funnel
      return funnel.watering(volume)


  for _ in range(1000):
      print(is_action_allowed("user", "get", 1))
  ```
  
  - Redis4.0新指令 cl.throttle 原子性的限流指令 
  - cl.throttle user:get 15 30 60 1
    - user:get 是key
    - 15 初始容量
    - 30 多少词操作
    - 60 多少时间
    - 30次和60s构成了漏水速率
    - 1 是volume，每次操作占用的空间
  - 返回值
    - 0 0表示允许，1表示拒绝
    - 15 漏斗容量
    - 14 漏斗剩余容量
    - -1 如果被拒绝了，需要多少时间重试
    - 2 多少时间后漏斗完全空出来
  
# GeoHash 地理位置
  - 用来解决数据库保存地理位置带来的查询，排序慢的问题
  - 基本思路是把二维坐标映射到一维坐标上，然后通过这条线来查找最近的点就可以了
    - 以常见的二刀法为例
      1. 第一次二刀将平面分成四份分别编号00，01，10，11
      2. 取左上角00块为例子，继续二刀切分，那么这就细分为0000，0001，0010，0011
      3. 其他的区块也是这么细分，最终每个点都可以近似表示成一个定长的整数
      4. 从而完成了从二维坐标转成一维数字的过程
    - GeoHash还会对这个整数做一次base32编码（0-9，a-z，去除a,i,l,o)
    - 在redis里面使用52位整数进行编码，放入zset里面，value是元素的key，score是元素的geohash52位整数值
  - geoadd +集合名称 +经度 +维度 +member 向集合名称中添加元素
  
  ```shell
  List of unsupported commands: DUMP, RESTORE, AUTH 
  Connecting ...

  Connected.
  101.200.121.40:0>geoadd company 116.48 39.99 juejin
  "1"

  101.200.121.40:0>geoadd company 116.51 39.90 ireader
  "1"

  101.200.121.40:0>geoadd company 116.49 40.00 meituan
  "1"

  101.200.121.40:0>geoadd company 116.56 39.78 jd 116.33 40.02 xiaomi
  "2"

  101.200.121.40:0>zrange company 0 -1
   1)  "jd"
   2)  "xiaomi"
   3)  "ireader"
   4)  "juejin"
   5)  "meituan"
  101.200.121.40:0>
  ```
  
  - geodist 查询集合中的元素之间的距离 携带集合名称 两个元素名称 和距离单位(m,km,ml,ft)
  
  ```shell
  101.200.121.40:0>geodist company xiaomi jd km
  "33.1323"

  101.200.121.40:0>
  ```
  
  - geopos可以获取经纬度坐标
  
  ```shell
  101.200.121.40:0>geopos company jd
   1)    1)   "116.55999809503555298"
    2)   "39.77999878782433285"

  101.200.121.40:0>geopos company xiaomi ireader
   1)    1)   "116.3299986720085144"
    2)   "40.01999886079634194"

   2)    1)   "116.51000171899795532"
    2)   "39.90000009167092543"

  101.200.121.40:0>
  ```
  
  - geohash可以获取hash值, 解码地址:http://geohash.org/${hash}
  
  ```shell
  101.200.121.40:0>geohash company jd
   1)  "wx4fk3srkc0"
  101.200.121.40:0>
  ```
  
  - georadiusbymember
  ```shell
  101.200.121.40:0>georadiusbymember company ireader 20 km count 3 asc // company集合中在20km中里ireader最近的3个公司，包括了自己，按距离正排
   1)  "ireader"
   2)  "juejin"
   3)  "meituan"
  101.200.121.40:0>georadiusbymember company ireader 20 km count 3 desc // company集合中在20km中里ireader最远的3个公司，按距离倒排
   1)  "jd"
   2)  "meituan"
   3)  "juejin"
  101.200.121.40:0>georadiusbymember company ireader 20 km count 3 desc withhash
   1)    1)   "jd"
    2)   "4069153314994473"

   2)    1)   "meituan"
    2)   "4069887165561330"

   3)    1)   "juejin"
    2)   "4069886973713703"

  101.200.121.40:0>georadiusbymember company ireader 20 km count 3 desc withhash withdist withcoord // 三个附加参数，分别是带hash，带距离，带坐标
   1)    1)   "jd"
    2)   "14.0136"
    3)   "4069153314994473"
    4)      1)    "116.55999809503555298"
     2)    "39.77999878782433285"


   2)    1)   "meituan"
    2)   "11.2526"
    3)   "4069887165561330"
    4)      1)    "116.48999780416488647"
     2)    "39.99999991084916218"


   3)    1)   "juejin"
    2)   "10.3322"
    3)   "4069886973713703"
    4)      1)    "116.47999852895736694"
     2)    "39.99000043587556519"
   ```
   - georadius 跟georadiusbymember差不多，但是提供了经纬度坐标来替换member
   
   ```shell
   101.200.121.40:0>georadius company 116.51 39.90 20 km count 3 a101.200.121.40:0>sc withhash withdist withcoord
   1)    1)   "ireader"
    2)   "0.0001"
    3)   "4069885997742760"
    4)      1)    "116.51000171899795532"
     2)    "39.90000009167092543"


   2)    1)   "juejin"
    2)   "10.3322"
    3)   "4069886973713703"
    4)      1)    "116.47999852895736694"
     2)    "39.99000043587556519"


   3)    1)   "meituan"
    2)   "11.2526"
    3)   "4069887165561330"
    4)      1)    "116.48999780416488647"
     2)    "39.99999991084916218"
   ```
   
   - 注意：
    1. 因为在集群环境中，集合可能从一个节点迁移到另一个节点，如果单个key过大，会造成集群迁移产生卡顿现象
    2. geo数据建议单独实例部署
    3. 要是数据量过大，可以考虑按省市进行拆分

# scan 查找key的指令
  简单粗暴的keys指令加正则字符串规则来查找符合的key
  
  ```shell
  List of unsupported commands: DUMP, RESTORE, AUTH 
  Connecting ...

  Connected.
  101.200.121.40:0>set a a
  "OK"

  101.200.121.40:0>set b b
  "OK"

  101.200.121.40:0>set c c
  "OK"

  101.200.121.40:0>keys *
   1)  "b"
   2)  "a"
   3)  "c"
  101.200.121.40:0>set k1 1
  "OK"

  101.200.121.40:0>set k2 2
  "OK"

  101.200.121.40:0>keys k*
   1)  "k2"
   2)  "k1"
  101.200.121.40:0>set 1k 1
  "OK"

  101.200.121.40:0>set 2k 2
  "OK"

  101.200.121.40:0>keys *k
   1)  "1k"
   2)  "2k"
  101.200.121.40:0>
  ```
  
  - 缺点是:
    1. 没有offset, limit参数不能分批次查找
    2. 是O(n)的遍历算法, 很慢, 加上redis是单线程程序, 其他指令必须等到keys结束之后才能执行
  - scan指令
    - 复杂度也是n, 但是是游标分步进行, 不会阻塞线程
    - 提供limit参数, 控制返回的key的数量, 只是一个hint, 实际数量可多可少
    - 提供正则匹配
    - 服务器不保存游标状态，返回给了客户端一个游标整数
    - 可能有重复
    - 查找的过程对key进行修改，不一定会在查找过程中体现出来
    - 返回为空不代表遍历结束，还要看游标值
  
  
  
  
  
  
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  



  
  
  
  
  
  
  
  
  
  
  
    
    
