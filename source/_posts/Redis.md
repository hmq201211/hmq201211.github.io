---
title: Redis  
date: 2020-12-28  
tags: 
- 缓存
- 数据库
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
    101.200.121.40:0>hgetall books
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
    101.200.121.40:0>hmset books java "effective java" python "learning python" golang "modern golang programming"
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
    101.200.121.40:0>hincrby user age 1
    "20"
    101.200.121.40:0>
    ```

