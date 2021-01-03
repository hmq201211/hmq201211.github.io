---
title: SelectPollEpoll  
date: 2021-01-03  
tags: 
- 操作系统
- 网络通讯
categories:
- 操作系统
---

# Select
  把fd（文件描述符）构造成bitmap传递给os的select，操作系统由用户态转内核态，并把bitmap拷贝到内核空间进行操作，要是有事件触发了，这个fd对应的bitmap的位置会被置位，客户端则需要遍历bitmap来查看那些fd有读写事件触发了，复杂度为O（n）
    1. select的bitmap有限制 1024
    2. bitmap不可重用
    3. 需要用户态转内核态的过程
    4. 需要n复杂度的遍历才能直到哪些fd有事件

# Poll
  关键的数据结构：
  ```C
  struct pollfd{
    int fd; // 文件描述符
    short events; // 关心的事件读或者写，要是同时关注，需要与这一位运算
    short revents; // poll方法的置位操作位
  }
  ```
  传入pollfd的数组，操作系统直接修改的是数组的pollfd的reevents
    1. 解决了大小限制的问题
    2. 可以重用数组

# Epoll
  关键的数据结构：
  
      epfd：fd-events
            fd-events
            fd-events
            fd-events
            fd-events
  特点：
    1. 使用了mmap来把用户态和内核态空间指向同一块内存
    2. 使用了就绪链表来把有数据的fd放在前面，并返回有几个fd触发了事件
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
