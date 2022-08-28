---
title: docker-rootless
date: 2022年8月28日15:08:31
updated: 2022年8月28日15:08:36
tags: [笔记,docker,rootless]
categories:
 - [笔记,docker]
 - [笔记,rooless]
keywords: docker，rootless
description: 待补充...
cover: false
---

# 参考 

>blog: https://www.cnblogs.com/cocoalatte/p/15919366.html
>
>zhihu:  https://zhuanlan.zhihu.com/p/496021016

# 前景提要

> 客户要求进程不要使用root用户，好在咱们的东西都跑在docker上，这样岂不是建个用户加个权限启动docker就解决了么，但是经过了解，docker的非root用户模式还有点费劲，所以记录一下

-----

# 创建用户

1. 使用`管理员` 创建一个名为`dockeruser` 的用户

~~~shell
sudo -i

sudo useradd dockeruser -d /home/dockeruser -m
~~~

2. `管理员`配置密码

~~~shell
sudo passwd dockeruser
~~~

3. `管理员`解决远程登录

~~~shell
vim /etc/passwd
~~~

> 修改以下内容
>
> ```
> dockeruser:x:1001:1001::/home/dockeruser:/bin/sh
> # 替换为 ↓↓↓↓↓↓↓
> dockeruser:x:1001:1001::/home/dockeruser:/bin/bash
> ```

# 开始安装

1. `管理员`安装rootless的必备组件： `newuidmap`  和  `newgidmap` ，复制以下命令并执行：

~~~shell
sudo sh -eux <<EOF
# Install newuidmap & newgidmap binaries
apt-get install -y uidmap
EOF
~~~

2. 登录`dockeruser`用户

~~~shell
ssh dockeruser@localhost
~~~

> 1. 要么通过系统登录界面，要么ssh登录
> 2. 不能使用su 方式切换到rootless用户,不然后面会提示检测不到systemd, 导致dockerd不能被systemd管理, 只能使用dockerd-rootless.sh脚本管理dockerd
> 3. ·    [There's no login session](https://link.zhihu.com/?target=https%3A//www.redhat.com/sysadmin/sudo-rootless-podman)

3. `管理员`  复制并执行

~~~ shell
curl -fsSL https://get.docker.com/rootless | sh
~~~

# 成功提示

```shell
[INFO] Installed docker.service successfully.
[INFO] To control docker.service, run: `systemctl --user (start|stop|restart) docker.service`
[INFO] To run docker.service on system startup, run: `sudo loginctl enable-linger dockeruser`
 
[INFO] Creating CLI context "rootless"
Successfully created context "rootless"
 
[INFO] Make sure the following environment variables are set (or add them to ~/.bashrc):
 
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/1001/docker.sock
```

#  将docker.sock添加到环境变量

1. `dockeruser`复制并执行:

```
$ vim ~/.bashrc 
```

>`dockeruser`添加以下内容``(``根据安装成功后的提示添加，参考成功提示10,11行)
>
>~~~ shell
>export PATH=/usr/bin:$PATH
>export DOCKER_HOST=unix:///run/user/1001/docker.sock
>~~~

2. 添加完以后使用`source ~/.bashrc` 命令使环境变量生效。

~~~shell
source ~/.bashrc
~~~

# 使用systemctl启动服务

1. 使用`dockeruser` 执行

```shell
systemctl --user start docker
```

- user 使用当前用户管理服务

2. `dockeruser`设置开始自启(不设置指定用户退出后，服务将停止)

```shell
loginctl enable-linger dockeruser
```

# rootless使用国内仓库

> rootless 与docker不同，docker在 `/etc/docker/daemon.json`
>
> rootless 在 `~/.config/docker/daemon.json`

1. 使用 `dockeruser` 编辑，复制并执行以下命令：

```shell
vim ~/.config/docker/daemon.json
```

2. 填写以下内容

```
{
   "registry-mirrors": ["http://hub-mirror.c.163.com","https://registry.docker-cn.com","https://3laho3y3.mirror.aliyuncs.com","http://f1361db2.m.daocloud.io","http://f1361db2.m.daocloud.io"]
}
```

3. 重启服务

```
systemctl --user restart docker
```

4. `docker info`  查看详情
