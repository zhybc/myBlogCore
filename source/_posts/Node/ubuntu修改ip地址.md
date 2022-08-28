---
title: ubuntu修改ip地址
date: 2022年8月28日15:59:27
updated: 2022年8月28日15:59:33
tags: [笔记,ubuntu]
categories:	#【可选】文章分类
 - [笔记,ubuntu]
keywords: ubuntu修改ip地址
description: ubuntu修改ip地址
cover: false	#【可选】文章缩略图(如果没有设置top_img,文章页顶部将显示缩略图，可设为false/图片地址/留空#  
---

# ubuntu修改ip地址

~~~shell
vi /etc/netplan/00-installer-config.yaml

## 修改以下内容 ##
##########################################
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens160:
        addresses: [192.168.33.117/24]
        dhcp4: false
        optional: true
        gateway4: 192.168.33.1
        # dhcp4: true
        nameservers:
                addresses: [192.168.33.1,114.114.114.114]

  version: 2
~~~

> 修改完成后，保存退出，然后输入命令重启网卡

~~~ shell
netplan apply
~~~

~~~ shell
# 修改主机名
$ hostnamectl --static set-hostname docker

# 修改网络信息
$ nano /etc/sysconfig/network-scripts/ifcfg-ens192

# 修改 root 用户的密码
$ passwd root

# 完成以上操作后，重启系统即可
$ reboot
~~~
