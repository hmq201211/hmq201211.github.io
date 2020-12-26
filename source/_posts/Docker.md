---
title: Docker  
date: 2020-12-25  
tags: 
- 容器
- 虚拟化
categories:
- 开发环境
---


# 思想：
Linux 为系统内核 加 文件系统  
Docker 复用了Linux的系统内核，加定制的文件系统，组成了虚拟化容器

# 组件：
  - Images： 
  Docker的镜像 用来生成Docker容器的模版
  - Container:
  Docker的实际运行的容器
  - Client：
  客户端，通过docker api与docker守护进程进行通讯
  - Host：
  主机，用来运行docker的守护进程和doucker容器
  - Registry：
  仓库用来保存镜像
  - Machine：
  虚拟化指令（命令行工具），简化docker的安装
  
# 架构：
![架构][1]

# docker 命令：
  - docker pull 从仓库拉去镜像
  - docker run 启动一个容器 
    - d 后台运行
    - name 容器名称
    - 再加 镜像名称：版本
  
  
  
  [1]: ../../../../images/picture/DockerFramework.png
