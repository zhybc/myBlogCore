---
title: hexo 常用命令
date: 2022-4-9
updated: 2022-4-10
categories: 
  - [教程,Hexo]
  - [教程,命令]
tags: [教程,Hexo,命令]
author: 大白菜
description: hexo 常用命令
group: post_Node
---

# 1.安装

安装

```go
npm install hexo -g
```

升级

```go
npm update hexo -g
```

初始化

```go
hexo init
```



## 2.发表文章

新建文章

```go
hexo n "我的博客" == hexo new "我的博客"
```

发表草稿

```go
hexo p == hexo publish
```

生成静态文件

```go
hexo g == hexo generate
```

启动服务预览

```go
hexo s == hexo server
```

部署到远程

```go
hexo d == hexo deploy
```

联合使用   清除缓存+生成静态文件+运行

```go
hexo clean && hexo g && hexo s
```



## 3.清理缓存

清除缓存文件 (db.json) 和已生成的静态文件 (public)

```go
hexo clean
```

## 4.版本

查看Hexo运行版本

```go
hexo version
```

## The End.

