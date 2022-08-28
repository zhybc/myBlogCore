---
title: docker和docker-dompose安装
date: 2022年7月11日19:22:55
updated:
tags: [docker,docker-compose]
categories:	
  - [笔记,docker]
  - [笔记,docker-compose]
keywords: docker安装、docker-compose安装、Linux
description: docker以及docker-dompose安装
top_img:
cover: false
---

# docker安装

​		安装Docker：新装的Ubuntu系统，需要安装Docker之后，才能使用Dockerfile，已经安装Docker的系统可跳过该步。

Docker安装参考步骤：https://developer.aliyun.com/article/762674

​		*备注：Docker安装后，一般需要管理员权限，因此运行Docker命令需要加sudo，也可以根据文章的说明修改为非管理员权限运行Docker。

Docker主要安装过程，依次键入以下命令，并等待安装结束：

1. 首先，更新软件包索引，并且安装必要的依赖软件，来添加一个新的 HTTPS 软件源：

```shell
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

2. 使用下面的 curl 导入源仓库的 GPG key：

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

3. 将 Docker APT 软件源添加到你的系统：

```shell
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

4. 现在，Docker 软件源被启用了，你可以安装软件源中任何可用的 Docker 版本。

安装 Docker 最新版本，运行下面的命令。

```shell
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```

# docker-compose 安装

## 方法 - 1

```shell
apt-get install docker-compress
```

## 方法 - 2 

1) 拉取

tips：v2.9.0 为版本，可能会失效，以git官网为准。

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/v2.9.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

2. 添加权限

```shell
sudo chmod +x /usr/local/bin/docker-compose
```



# The End.
