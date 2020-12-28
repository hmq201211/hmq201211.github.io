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
  
  
