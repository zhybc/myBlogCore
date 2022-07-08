---
title: 关于TLS经验小结
date: 2022-3-20
categories: 
  - [笔记,网络]
tags: [笔记,网络]
author: 大白菜
group: post_Node
description: 关于Tls.......
---

# **关于TLS经验小结**

## 什么是tls？

安全传输层协议（Transport Layer Security）用于在两个通信应用程序之间提供保密性和数据完整性。TLS是SSL的标准化后的产物，有有1.0 1.1 1.2 1.3四个版本，目前最常用的是tls1.2协议。

1.1 算法种类

- 非对称加密: RSA DH
- 对称加密: AES DESHash
- 算法(散列): md5 sha1 sha256

1.2 tls协议里面的算法的概念

- 密钥交换算法(Kx)
- 认证算法(Au)
- 加密算法(Enc)
- 消息摘要算法(Mac)

说了这么多，估计大家也有点懵逼了。大家肯定也有一个疑问，上述介绍的这些到底有什么用呢？ok 先上个图：

<img src="https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/Image.png" alt="Image" style="zoom: 80%;" />

​		Tips：这张图描述的是SSL通过RSA非对称加密方式的握手流程：

1. ssl握手过程:
   1. 客户端(一般为浏览器)向服务器发起send hello 其中的内容为:
      1. 协议版本号（支持的tls版本）
      2. 客户端生成的随机数（Client random） 
      3. 以及客户端支持的对称加密方法集。
   2. 服务端根据客户端send hello请求内容返回本次通信的tls版本， 具体使用哪一种对称加密方法,服务器生成的随机数（Server random） 以及服务器的证书及证书链。
   3. 客户端采用对应CA的公钥(认证算法)检查该证书是否有效、有没有 过期。如果证书没有问题。就再生成一个新随机数(Premaster secret)。 并且用服务端给的证书公钥加密这个随机数发送给服务器。
   4. 服务器用证书秘钥解密客户端发来的加密随机数(Premaster secret)。根据刚才双方协商的对称加密算法，使用之前的三个随机数去生成一个对话秘 钥，接下来的所有通信加解密都使用这个对话秘钥进行。
2. 为什么要用非对称加密？
   1. 在进行ssl握手时（此时只是建立连接并没有开始传输数据）一切的握手传输数据都是明文的。 而我们在握手成功后最终需要用来对数据进行加解密的是对称加密方法。对称加密方法的好处 是，体积小，效率高。缺点也是显而易见，通信的暗号或密码绝对不能被他人知道。但是ssl 握手时是明文。所以为了不暴露对称加密的暗号或密码我们需要用非对称加密的方式保护起来。
3. 这样的握手机制真的很安全吗？
   1. 刚才的RSA握手机制我们注意到以下几点:
      1. 一共生成了3次随机数
      2. 服务器需要将自己的证书公钥发送给客户端已便客户端生成Premaster secret那么问题来了？如果在你(客户端)与服务器中间有一个每天都没事情做，和你怼上了就一直监控收集你和 服务端之间的通信数据的中间人。然后某一天服务器的私钥不小心泄露了，又刚好秘钥被 这个中间人获得了，因为握手时数据为明文而且为了客户端生成Premaster secret 服 务端发送了公钥，所以中间人很容易得到公钥。最后不幸的事情发生了，中间人有公钥也 有秘钥。虽然服务器马上更换了证书。但是之前的数据，中间人全部都可以解密而获得到。**那有什么办法可以规避？**![image-20220320121243961](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/Image.png)上图是基于DH算法的SSL握手流程

DH和RSA有什么区别吗？

其他的步骤基本一致，只有一点。

RSA采用的是服务器发送证书公钥，客户端公钥加密随机数再传输给服务端解密最后 生成session key

而DH采用的是服务端发送算法的相应算法加解密参数，客户端也发送算法的相应参数。然后两边自己算自己的。最后对比，如果一致则建立session key，这样就算泄露了，由于在建立连接时并没有传输证书公钥所以不会对之前的数据产生被破解的风险。这就是PFS（Perfect Forward Secrecy）完美前向安全性。

## 什么是对称加密？

采用单钥密码系统的加密方法，同一个密钥可以同时用作信息的加密和解密，这种加密方法称为对称加密。

它的优点:

- 安全强度高
- 计算量小
- 加解密速度快效率高

如图：

![image-20220320121059794](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320121059794.png)

从图中我们可以看到,在相同安全强度的情况下,对称加密80位长度 = RSA加密1024位长度 = EC椭圆曲线算法加密160位长度。秘钥长度越长计算量越大体积也越大。由此可见在性能方面对称加密是性能最优的。

所以这也是为什么在建立连接之后我们更倾向于使用对称加密的方式进行数据加密的原因。



## 对称加密模式

对称加密主要是有6种常见的模式：

1. ECB模式 电子密码本模式
2. CBC模式 密文分组链接模式
3. CFB模式 密文反馈模式
4. OFB模式 输出反馈模式
5. CTR模式 计数器模式
6. AEAD模式 关联数据的认证加密模式

这里我们不讨论算法也不讨论实现方式。

拿出3个重点 ECB CBC AEAD

![image-20220320121407193](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320121407193.png)

将一个明文信息 拆分成若干个小块，然后分别进行加密。然后并行传输。对端在分别对块进行解密后拼装成一个完整的信息。

优点显而易见 通俗易懂,简单方便,速度快

缺点也显而易见 可以对密文块替换删除添加从而篡改明文块的相关数据，同样的明文块会被加密成相同的密文块这样不能很好的隐藏数据模式也无法抵御重放攻击。

那怎么办呢？ CBC模式登场

![image-20220320120810138](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320120810138.png)

 		在CBC模式中，每个明文块先与前一个密文块进行异或运算后再进行加密。在这种方法中，每个密文块都依赖于它前面的所有明文块。同时，为了保证每条消息的唯一性，在第一个块中需要使用初始化向量。

​		这样看上去是不是比ECB模式更好呢？但是这样就不能并行计算效率大打折扣。而且也并没有很安全。为什么？

​		在这种模式下需要在加载前确保每个块的长度都是总长度的整数倍。一般情况下，明文的最后一个块很有可能会出现长度不足总长度的现象。这个时候，普遍的做法是在最后一个分组后填充一个固定的值。而攻击者可以根据这个原理进行相应的填充攻击。想了解具体内容可以阅读LittleHann大神的文章

[Padding Oracle攻击的分析和思考](https://link.jianshu.com/?t=http://www.freebuf.com/articles/web/15504.html)

那怎么办？

或许可以尝试下AEAD(认证加密方法) 

它同时对数据提供机密性，完整性和真实性保证。

AEAD 3种方式(来自维基百科):

- Encrypt-then-MAC（EtM）
  - 明文首先被加密，然后基于生成的密文生成MAC。密文和MAC一起发送。
- Encrypt-and-MAC（E＆M）
  - 基于明文生成MAC，明文加密，无MAC。明文的MAC和密文一起发送。
- MAC-then-Encrypt（MtE）
  - 基于明文生成MAC，然后明文和MAC一起被加密以产生基于两者的密文。发送密文（包含加密MAC）

注： MAC(消息摘要算法用于数据完整性校验)

AEAD是目前相对于其他模式而言比较好的选择。

## 对称加密算法

> AES DES

​		目前使用的几乎都是AES-GCM（DES安全性太差基本处于废弃状态）

​		那除了这两种还有其他的吗？

​		答案是有的

​		ChaCha20

那这种算法与AES-GCM相比有什么特别之处呢？

![image-20220320121521620](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320121521620.png)

上图展示的是CHACHA20-Poly1305 与 AES-GCM 两种对称加密算法的加密效率。

​		从这张图中我们可以看到 ChaCha20的加密速度比AES快了差不多3倍左右。那为什么现在的网站打开都基本上AES-GCM呢？

​		仔细看图我们可以看到，这是基于ARM架构的移动端CPU的加密速度。CHACHA20虽然在移动端CPU架构上加解密速度非常快，但是在传统>的x86架构上却比AES慢。再加上支持CHACHA20的证书比较难申请的到。所以导致了现在的网站打开都基本上AES-GCM。

​		不过如果业务主要针对是移动端并且服务端硬件配置比较好且申请到了支持CHACHA20的证书。由于该算法在移动端加解密速度非常快。所以能够得到较好的用户体验。（虽然从运维的角度来说应该尽量降低服务器的负载，将负载转移到用户端比较好。主要还是看如何取舍）

## 关于TLS1.3

​		首先我们了解下正常的https请求的流程:

![image-20220320121622427](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320121622427.png)

​		从图片中我们可以看到http的延迟时间大概在56ms

​		而https因为是在http协议上额外增加了tls协议所以延迟时间大概需要168ms（http+tls）

​		由此可以看出使用https确实会给网站的打开速度产生影响，最直接的就是用户打开网站速度变慢。而且当用户不小心断开之后又要重>新走完这整一套的步骤。那有什么办法可以规避这种办法呢？

session id

![image-20220320121706984](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320121706984.png)

![image-20220320121725432](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320121725432.png)

![image-20220320121743929](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320121743929.png)

​		session id是一种用户断线重连之后能快速建立连接的一种方法。原理非常简单，当用户与服务器第一次成功完成握手之后，服务器>端生成一个session id 自身保存一份，另外一份发送给客户端。当客户端断线重连之后客户端像服务端发送session id 如果服务端发现这个id存在。则免去繁琐的tls握手步骤 直接连接。从图中我们也看到采用这样的方式延迟时间降低了50%左右但是方法不适合分布式架构的业务，原因在于session id 是在每一台服务器中生成的。所以当客户端重连后连接到服务器的另外一台后端业务服务器就会因为没有id 而重新进行一次完整握手。

session ticket

![image-20220320121830718](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320121830718.png)

![image-20220320121846671](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320121846671.png)

​		session ticket的原理与session id 相同只不过session ticket 不是生成id 而是根据服务器上的ticket key 加密一个数据blob发>送给客户端，然后客户端重连时发送blob给服务器，服务器解密这个blob来判断是否是重连用户从而免去繁琐的tls握手步骤。采取这>样的方式的好处就是我们可以将session key 存储在所有的分布式后端业务服务器上。这样不管用户重连到哪一台服务器都可以通过session key来解密判断是否为重连用户。

​		那我们有更简单方便的方法吗？

​		有的，那就是tls1.3！

![image-20220320121931425](https://gitee.com/zhy_bc/my-photo-manage/raw/master/Typora/image-20220320121931425.png)

​		如上图：

​		tls1.3是最新的tls协议版本。

​		废弃了很多年老安全性差的算法并且采用了更优秀的握手机制。

​		tls1.3 移除了RSA握手机制，采用了(EC)HDE 握手机制（RSA与DH的优缺点在上文中有介绍）

​		移除CBC对称加密模式，采用AEAD模式（对称加密模式上文中有介绍）

​		以及其他的特性。[具体详见](https://link.jianshu.com/?t=https://tools.ietf.org/html/draft-ietf-tls-tls13-21)

​		其中最瞩目的莫过于新的握手协商机制

TLS1.3

<img src="C:\Users\The_Monster\AppData\Roaming\Typora\typora-user-images\image-20220320122035136.png" alt="image-20220320122035136" style="zoom:67%;" />

TLS1.2

<img src="C:\Users\The_Monster\AppData\Roaming\Typora\typora-user-images\image-20220320122059361.png" alt="image-20220320122059361" style="zoom: 50%;" />

​		如上图tls1.3协议将之前tls1.2需要的步骤整合在一起，然后集中传输。减少了握手次数，从而大幅减少了延迟时间。加快了网站访问速度，使我们再也不用去纠结不上https不安全，上了https访问速度慢的历史性问题。

## 总结

​		从之前讲的算法到握手机制。使大家对整个tls协议有了一个大概的认识。从生产环境角度来说，采用ECDH+ECDSA+AES-GCM的 ECC证书是目前在安全性和性能上都相对平衡的一种证书种类。本站的证书就是采用了ECC证书。[关于ECC证书可以查看](https://link.jianshu.com/?t=https://imququ.com/post/ecc-certificate.html)

​		在此感谢峰哥的一次知识分享让我了解到了相关的知识（文章中的一些示例图也源于峰哥的ppt）想具体了解大佬可以访问以下博客更深入的了解

[TLS 1.3 如何用性能为 HTTPS 正名](https://link.jianshu.com/?t=https://mp.weixin.qq.com/s?__biz=MjM5NzAwNDI4Mg%3D%3D&mid=2652193066&idx=1&sn=20245f33e41b6dea25d9ad7c5da4d0fa&chksm=bd0177bf8a76fea9f68f0d22e02b331dcadbf34eb1aa680100c76105430e1c6cc6f78f165a72)

[图解SSL/TLS协议](https://link.jianshu.com/?t=http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html)

[分组密码工作模式](https://link.jianshu.com/?t=https://zh.wikipedia.org/zh-cn/分组密码工作模式)

作者：HenryYee

链接：https://www.jianshu.com/p/081bbf9d6b69

来源：简书