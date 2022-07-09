---
title: adb连接雷电模拟器修改hosts
date: 2022-3-20
updated: 2022-4-10
categories: 
  - [笔记,网络]
  - [笔记,雷电模拟器]
  - [笔记,配置]
tags: [笔记,网络,雷电模拟器,配置]
author: 大白菜
description: adb连接雷电模拟器修改hosts
group: post_Node
---

# **adb连接雷电模拟器修改hosts**

下面很多都是转载,背景简介:我的模拟器有2个以上devices,按照网上查询的很多都不能使用,在下面才找到解决方案

1. 如果找到[adb](https://so.csdn.net/so/search?q=adb&spm=1001.2101.3001.7020)？

雷电安装模拟器自带了一份，当然熟悉的朋友，喜欢用自己珍藏的版本也是可以的。雷电自带的版本再安装目录下，名字就叫adb.exe，如本人的电脑adb全路径为：F:\mnq\dnplayer\adb.exe.

2. 如何链接设备？

一步步看命令行

```go
cmd
cd F:\mnq\dnplayer
adb.exe kill-server //(很多时候连不上，就是因为没有kill)
adb.exe devices
```

这步很重要，完成之后，会list出所有设备

备注:我使用的是配置到环境变量中的adb,可以直接使用 adb devices

3. 多开的情况下如何指定操作哪个模拟器？

这一步至关重要，所以提前说明，后面的所有操作都是单开为例，多开的情况，请参考这部分，切记切记！！！

adb devices会获取模拟器列表，指定模拟器只需要在adb后面加上" -s 模拟器标识"即可！

比如说：

```go
127.0.0.1:5555
127.0.0.1:5557
```

（对，雷电的adb端口是有规律的，规律就是 5555 + index * 2）

下面图片是我本地的

![20201204141422735](http://reqj6ffi8.hn-bkt.clouddn.com/blogimages/20201204141422735.png)

4. 重新挂载模拟器   

```go
adb -s emulator-5556 remount
```

5. 将模拟器hosts pull到本地目录C:\Users\Administrator

   ![20201204141707851](http://reqj6ffi8.hn-bkt.clouddn.com/blogimages/20201204141707851.png)

6. 在本地用记事本打开hosts文件(在C:\Users\Administrator文件中 ) 修改hosts文件，然后保存

   ![6a5efc7b62c6a2d51c73a18d8fc7ce6f](http://reqj6ffi8.hn-bkt.clouddn.com/blogimages/6a5efc7b62c6a2d51c73a18d8fc7ce6f.png)

7. 将修改后的hosts文件上传到模拟器

![image-20220320122731087](http://reqj6ffi8.hn-bkt.clouddn.com/blogimages/image-20220320122731087.png)

上传成功

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

> 备注:上面的过程我看了是可以把hosts上传到模拟器上的,但是hosts的配置不起作用,也不知道什么问题,今天刚刚我把雷电模拟器启用了网络桥接,然后又重新配置了一下hosts,使用第5步把

- 重新配置的hosts上传到模拟器上,结果现在可以了

![image-20220320112458296](http://reqj6ffi8.hn-bkt.clouddn.com/blogimages/image-20220320112458296.png)

- 下面的是我的桥接配置

![image-20220320112655602](http://reqj6ffi8.hn-bkt.clouddn.com/blogimages/image-20220320112655602.png)

=================================================================================================

2021年4月22日17:27:23

今日重新配置了一遍,发现了几个问题

1. 我下载下来的hosts修改完以后(我这边修改了两行分别是 

127.0.0.1 aa.com 

127.0.0.1 bb.com

结果aa.com可以访问到本地,bb不可以,然后我又加了一行cc.com.bb才可以访问.好像是最后一行最好需要回车一下要不然上传到模拟器以后格式会出现问题)

2. 修改完以后电脑最好重启一下