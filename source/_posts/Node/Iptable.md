---
title: Iptable 详解！
date: 2022-5-15
categories: 
  - [笔记,Iptable,Linux]
tags: [笔记,Iptabl,Linux]
author: 大白菜
group: post_Node
description: Iptable 详解！
---



# Iptable

> iptables  [-t table]  [COMMAND]  [chain]  [CONDITION]  [-j ACTION]
>
> iptables  [-t 表名]  [命令选项]［链名条件匹配］ ［-j 目标动作或跳转］

```C#
-t table   是指'操作的表',filter、nat、mangle或raw,'默认使用filter'

COMMAND    '子命令',定义'对规则的管理',PREROUTING、POSTROUTING、OUTPUT

chain      指明'链路'

CONDITION  匹配的'条件或标准',用于指定对符合什么样 条件的数据包进行处理；

ACTION     匹配后'操作动作',用于指定数据包的处理方式（比如允许通过、拒绝、丢弃、跳转（Jump）给其它链处理。

```



## **-t   table**

```C#
#----------------------------
规则表：
    1）filter表——三个链：INPUT、FORWARD、OUTPUT
作用：过滤数据包 内核模块：iptables_filter.
    2）Nat表——三个链：PREROUTING、POSTROUTING、OUTPUT
作用：用于网络地址转换（IP、端口） 内核模块：iptable_nat
    3）Mangle表——五个链：PREROUTING、POSTROUTING、INPUT、OUTPUT、FORWARD
作用：修改数据包的服务类型、TTL、并且可以配置路由实现QOS内核模块：iptable_mangle(别看这个表这么麻烦，
咱们设置策略时几乎都不会用到它)
   4）Raw表——两个链：OUTPUT、PREROUTING
#-----------------------------
规则链：
   1）INPUT——进来的数据包应用此规则链中的策略
   2）OUTPUT——外出的数据包应用此规则链中的策略
   3）FORWARD——转发数据包时应用此规则链中的策略
   4）PREROUTING——对数据包作路由选择前应用此链中的规则
（记住！所有的数据包进来的时侯都先由这个链处理）
   5）POSTROUTING——对数据包作路由选择后应用此链中的规则
（所有的数据包出来的时侯都先由这个链处理）
```



## **iptables命令的管理控制选项**

```C#
-A 在指定链的末尾添加（append）一条新的规则
-D 删除（delete）指定链中的某一条规则，可以按规则序号和内容删除
-I 在指定链中插入（insert）一条新的规则，默认在第一行添加
-R 修改、替换（replace）指定链中的某一条规则，可以按规则序号和内容替换
-L 列出（list）指定链中所有的规则进行查看（默认是filter表，如果列出nat表的规则需要添加-t，即iptables -t nat -L）
-E 重命名用户定义的链，不改变链本身
-F 清空（flush）
-N 新建（new-chain）一条用户自己定义的规则链
-X 删除指定表中用户自定义的规则链（delete-chain）
-P 设置指定链的默认策略（policy）
-Z 将所有表的所有链的字节和数据包计数器清零
-n 使用数字形式（numeric）显示输出结果
-v 查看规则表详细信息（verbose）的信息
-V 查看版本(version)
-h 获取帮助（help）
```



## **在iptables中，'-j 参数'用来指定要'进行的处理动作'**

```javascript
常用的'处理动作'包括：
      'ACCEPT'、接受
      'REJECT'、明示拒绝
      'DROP'、悄悄丢弃
      'REDIRECT'、重定向：主要用于实现端口重定向
      'MASQUERADE'、源地址伪装
      'LOG'、
      'DNAT'、
      'SNAT'、
      'MIRROR'、
      'QUEUE'、
      'RETURN'、
      'MARK'

 '一条链上有多个规则' --> '如果先匹配,后面的规则就不再处理' --> '跳转到下一个链条'
```



## **用到的命令**

> iptables [-t 表名] [命令选项] ［链名］［[条件匹配](https://www.cnblogs.com/justdoit-qing/articles/3990911.html)］［-j 目标动作或跳转］
>
> iptables [] [-A] [INPUT] [] [-j NFQUEUE --queue-num 1] #  -t table 默认为：filter

> `参数`
>
> iptables [-t nat] [-A] [PREROUTING] [-d `host` -p tcp --dport `dport`] [-j DNAT --to-destination `dest`]
>
> iptables [-t nat] [-D] [POSTROUTING] [-d `deshost`  -p tcp --dport `desport`] [-j SNAT --to `host` ]



## **Info**

```java
# 命令选项
-A 在指定链的末尾添加（append）一条新的规则
-D 删除（delete）指定链中的某一条规则，可以按规则序号和内容删除
# 链名
PREROUTING——对数据包作路由选择前应用此链中的规则（记住！所有的数据包进来的时侯都先由这个链处理）
POSTROUTING——对数据包作路由选择后应用此链中的规则（所有的数据包出来的时侯都先由这个链处理）
# 条件匹配
 1、Match：-d, --dst, --destination
    Examp：iptables -A INPUT 【-d 192.168.1.1】
    Exp：以IP目的地址匹配包。地址的形式和 -- source完全一样。
 2、Match：-p, --protocol 
    Examp：iptables -A INPUT 【-p tcp】
    Exp：匹配指定的协议。指定协议的形式有以下几种：
    1、名字，不分大小写，但必须是在/etc/protocols中定义的。
    2、可以使用它们相应的整数值。例如，ICMP的值是1，TCP是6，UDP是17。
    3、缺省设置，ALL，相应数值是0，但要注意这只代表匹配TCP、UDP、ICMP，而不是/etc/protocols中定义的所有协议。
    4、可以是协议列表，以英文逗号为分隔符，如：udp,tcp
    5、可以在协议前加英文的感叹号表示取反，注意有空格，如: --protocol ! tcp 表示非tcp协议，也就是UDP和ICMP。可以看出这个取反的范围只是TCP、UDP和ICMP。
 3、Matc：--dport, --destination-port
 	 Examp：iptables -A INPUT -p tcp 【--dport 22】
    Exp：基于TCP包的目的端口来匹配包，端口的指定形式和--sport完全一样。
# 目标动作
1、Option：--to-destination
Examp：iptables -t nat -A PREROUTING -p tcp -d 15.45.23.67 --dport 80 -j DNAT 【--to-destination 192.168.1.1-192.168.1.10】
Exp：
   指定要写入IP头的地址，这也是包要被转发到的地方。上面的例子就是把所有发往地址15.45.23.67的包都转发到一段LAN使用的
   私有地址中，即192.168.1.1到 192.168.1.10。如前所述，在这种情况下，每个流都会被随机分配一个要转发到的地址，但同一
   个流总是使用同一个地址。我们也可以只指定一个IP地址作为参数，这样所有的包都被转发到同一台机子。我们还可以在地址后指
   定一个或一个范围的端口。比如：--to-destination 192.168.1.1:80或 --to-destination 192.168.1.1:80-100。SNAT的语
   法和这个target的一样，只是目的不同罢了。要注意，只有先用--protocol指定了TCP或 UDP协议，才能使用端口。
2、Option：--to-source
Examp：iptables -t nat -A POSTROUTING -p tcp -o eth0 -j SNAT 【--to-source 194.236.50.155-194.236.50.160:1024-32000】
Exp：
    指定源地址和端口，有以下几种方式：
    1、单独的地址。
    2、一段连续的地址，用连字符分隔，如194.236.50.155-194.236.50.160，这样可以实现负载平衡。每个流会被随机分配一个IP，但对于同一个流使用的是同一个IP。
    3、在指定-p tcp 或 -p udp的前提下，可以指定源端口的范围，如194.236.50.155:1024-32000，这样包的源端口就被限制在1024-32000了。
    注意，如果可能，iptables总是想避免任何的端口变更，换句话说，它总是尽力使用建立连接时所用的端口。但是如果两台机子使用
    相同的源端口，iptables 将会把他们的其中之一映射到另外的一个端口。如果没有指定端口范围， 所有的在512以内的源端口会被
    映射到512以内的另一个端口，512和1023之间的将会被映射到 1024内，其他的将会被映射到大于或对于1024的端口，也就是说是同
    范围映射。还要注意，这种映射和目的端口无关。因此，如果客户想和防火墙外的HTTP服务器联系，它是不会被映射到FTP control所用
    的端口的。   
#
1、DNAT target
这个target是用来做目的网络地址转换的，就是重写包的目的IP地址。如果一个包被匹配了，那么和它属于同一个流的所有的包都会被自动转换，
然后就可以被路由到正确的主机或网络。DNAT target是非常有用的。比如，你的Web服务器在LAN内部，而且没有可在Internet上使用的真实IP地址，
那就可以使用这个 target让防火墙把所有到它自己HTTP端口的包转发给LAN内部真正的Web服务器。目的地址也可以是一个范围，这样的话，DNAT会为
每一个流随机分配一个地址。因此，我们可以用这个target做某种类型地负载平衡。
注意，DANT target只能用在nat表的PREROUTING和OUTPUT链中，或者是被这两条链调用的链里。但还要注意的是，包含DANT target的链不能被除此之外的
其他链调用，如POSTROUTING。
2、SNAT target
这个target是用来做源网络地址转换的，就是重写包的源IP地址。当我们有几个机子共享一个Internet 连接时，就能用到它了。先在内核里打开ip转发功能，
然后再写一个SNAT规则，就可以把所有从本地网络出去的包的源地址改为Internet连接的地址了。如果我们不这样做而是直接转发本地网的包的话，Internet上的
机子就不知道往哪儿发送应答了，因为在本地网里我们一般使用的是IANA组织专门指定的一段地址，它们是不能在Internet上使用的。SNAT target的作用就是让所
有从本地网出发的包看起来都是从一台机子发出的，这台机子一般就是防火墙。
SNAT只能用在nat表的POSTROUTING链里。只要连接的第一个符合条件的包被SNAT了，那么这个连接的其他所有的包都会自动地被SNAT,而且这个规则还会应用于这个连
接所在流的所有数据包。

```

```java
#、NFQUEUE
iptables除了可以把包转发给特定地址特定端口的监听服务外, 还可以转发到一个叫做NFQUEUE的目标上,
NFQUEUE是
iptables的一种规则目标, 它用于将网络数据包从内核传给用户态进程, 由用户态进程来裁决如何处理该数据包，并将裁决结果返回内核。传输通道为以数字标识的队列。队列由固定长度的链表实现，链表元素为数据包及元数据(kernel skb结构)。
在内核中，Netfilter框架尝试将符合规则的数据包放入队列中。若队列已满，则丢弃该数据包。因此，若用户态进程处理过慢，则会严重影响网络性能。内核与用户态进程之间基于
NFNETLINK通信，数据包需要在内核态与用户态之间进行拷贝，因而这种机制的性能比较差。
iptables -A INPUT -j NFQUEUE --queue-num 0
在包被转发到queue num=0 NFQUEUE队列之后, 某个使用libnetfilter_queue 连接队列0的进程从内核态获取到包的信息, 
对包的去向进行裁决. NFQUEUE之后的工作是在用户态完成的, 所以这里也有相应的性能损耗. fqting使用了NFQUEUE对包的内容进行
了混淆, 混淆后的包的参数使之在IDS上重组时产生错位, 重组流不为中间路由所知, 在proxy ip不被封禁的情况下可以绕过封禁内容的防火墙.
```



## **清除规则**

```java
# 清除规则
iptables [-t tables] [-F/X/Z]
-F：清除所有已制定的规则
-X：删除所有使用者自定义的 chain(其是 tables)
-Z：将所有的 chain 的计数与流量统计都清零
```

