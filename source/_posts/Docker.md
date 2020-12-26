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
Docker 容器使用的是最小定制 例如有ls 但是没有ll

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

# docker 安装:
  - yum仓库配置路径: /etc/yum.repos.d/
  - wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  - yum install docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io -y

# docker 命令：
  - systemctl start docker 启动docker
  - systemctl enable docker 开机自动启动docker
  - systemctl deamon-reload 重新加载systemctl所管理的服务的配置文件
  - systemctl restart docker 重启docker
  - docker pull 从仓库拉去镜像
  - docker run 启动一个容器 
    - d 后台运行
    - name 容器名称
    - 再加 镜像名称：版本
    - v 宿主机目录：要替代的容器内部文件目录
  - docker images 查看已有的镜像
  - docker ps 查看容器状态
  - docker info 查看docker信息
  - docker version 查看docker版本
  - docker inspect +容器名称或者id 查看容器信息
  - docker rm -fv +容器名称或者id 删除容器
    - f 强制删除
    - v 附带的数据也删除
  - docker exec -it 容器的名称或者id /bin/bash
    - i 即使没有附加, 也保持stdin打开
    - t 分配一个伪终端
    - d 分离模式, 在后台运行
    - exit 退出
  
  [1]: ../../../../images/picture/DockerFramework.png
