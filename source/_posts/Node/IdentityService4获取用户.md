---
title: Identity Service 4 APi 获取用户信息
date: 2022-4-11
updated: 2022-4-11
categories: 
  - [笔记,IdentityServer]
  - [笔记,.net]
tags: [笔记,IdentityServer,.net]
author: 大白菜
cover: false
description: Identity Service 4 APi 获取用户信息
group: post_Node
---

# Identity Service 4 APi 获取用户信息

在构造Asp.Net等基于.Net Framework等应用时，Web Api或者其它后台往往与前台处于同一进程，一般通过Session或者其它方式获取用户信息。采用bearer JWT认证的Web Api与被调用的前端处于不同的进程甚至不同的服务器，因此不能使用Session等获取用户信息。用户信息通过JWT传递到API，被平台填充到User.Claims中，可以直接获取，代码如下：

`from c in User.Claims select new { c.Type, c.Value }`

```c#
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace Api.Controllers
{
    [Route("identity")]
    [Authorize]
    public class IdentityController : ControllerBase
    {
        [HttpGet]
        public IActionResult Get()
        {
            return new JsonResult(from c in User.Claims select new { c.Type, c.Value });
        }
    }

```

 作者：寻找无名的特质
 链接：https://www.jianshu.com/p/36c2ea3160a0
 来源：简书
 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



## The End.

