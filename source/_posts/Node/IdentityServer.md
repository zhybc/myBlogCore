---
title: skoruba-Duende.IdentityServer.Admin
date: 2022-4-10
updated: 2022-4-10
categories: 
  - [笔记,.net]
  - [笔记,IdentityServer]
  - [笔记,ids4]
tags: [笔记,.net,IdentityServer,ids4]
author: 大白菜
description: skoruba-Duende.IdentityServer.Admin
group: post_Node
---

# Duende IdentityServer 和 Asp.Net 核心身份的管理

Github:[GitHub - skoruba/Duende.IdentityServer.Admin：Duende IdentityServer 和 Asp.Net Core Identity ⚡ 的管理](https://github.com/skoruba/Duende.IdentityServer.Admin)

## 添加Api客户端

Login.Api 为例

Ids4配置地址：https://localhost:44303/Configuration/Clients

### 必须添加Api Scope

![image-20220410190548460](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/IdentityServer-image-20220410190548460.png)

### 添加客户端

添加

![image-20220410190906111](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/IdentityServer-image-20220410190906111.png)

基本设置

![image-20220410191004732](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/IdentityServer-image-20220410191004732.png)

如下图，其中作用设置必须已有的。

![image-20220410191109566](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/IdentityServer-image-20220410191109566.png)

###  Appsetting

![image-20220410191235559](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/IdentityServer-image-20220410191235559.png)

## 添加Vue

### Vue

![image-20220410191528315](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/IdentityServer-image-20220410191528315.png)

### 添加客户端

![image-20220410191623757](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/IdentityServer-image-20220410191623757.png)

<div style="color:red;">
    基础设置注意作用域，需要在配置中设置。
</div>

![image-20220410192040067](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/IdentityServer-image-20220410192040067.png)

![image-20220410191815352](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/IdentityServer-image-20220410191815352.png)

<div style="color:red;">
    注意配置跨域来源、注销地址
</div>
