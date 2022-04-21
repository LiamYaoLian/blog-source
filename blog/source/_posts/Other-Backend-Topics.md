---
title: Other Backend Topics
date: 2022-04-20 18:49:54
tags:
---

## Docker
* a container to install the program and its environment
### Container
* 镜像类似于Java中的类，而容器就是实例
* 容器可修改，镜像不可修改
* 同一个镜像可生成多个容器独立运行，他们之间没有任何干扰
### Client
* client提供给用户一个终端，用户输入 Docker 提供的命令来管理本地或远程的服务器
### Daemon
* Daemon：服务端守护进程，接收 Client 发送的命令并执行相应的操作
### Steps
* download images
  - docker pull [OPTIONS] NAME[:TAG]
  - docker images [OPTIONS] [REPOSITORY[:TAG]]
* docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
