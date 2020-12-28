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
- 中间件
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
        101.200.121.40:0>

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
    
  
  
  
  
  
  
  
  
  
  
  
  
  
  
    
    
