---
title: .net Core Api Swagger Ui 添加注释显示
date: 2022-4-9
updated: 2022-4-10
categories: 
  - [笔记,.net]
  - [笔记,Swagger]
  - [笔记,配置]
tags: [笔记,.net,Swagger,配置]
author: 大白菜
description: .net Core Api 添加注释显示
group: post_Node
---

# Net Core Api Swagger 添加显示注释

## StartUp中配置

> 在 ConfigureServices 中添加 

相关代码：

```c#
var basePath = AppContext.BaseDirectory;

var xmlPath = Path.Combine(basePath, "Gmzta.ControlPlane.Api.xml");
options.IncludeXmlComments(xmlPath, true);//显示控制器层注释
//options.OrderActionsBy(s => s.RelativePath);//对Action的名称进行排序
```

如图：

![image-20220410020403079](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/SwaggerUi-image-1.png)

##  设置属性-勾选文档生成

> 错误代码取消1591显示

![image-20220410020605065](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/swagger-image-2.png)

The End.
