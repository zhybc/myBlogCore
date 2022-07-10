---
title: Swagger-IdentityServer4-oauth2
date: 2022-4-10
updated: 2022-4-10
categories: 
  - [笔记,.net]
  - [笔记,Swagger]
  - [笔记,ids4]
  - [笔记,IdentityServer]
  - [笔记,oauth2]
tags: [笔记,.net,Swagger,ids4,IdentityServer,oauth2]
author: 大白菜
cover: false
description: Swagger+IdentityServer4,Swashbuckle,Nswag
group: post_Node
---

#  使用 IdentityServer4 的 ASP.NET Core Swagger UI 授权

## 参考项目Demo

- IdentityService：https://github.com/skoruba/Duende.IdentityServer.Admin
- Sts+ .net Core Api（Jwt）： https://github.com/damienbod/IdentityServer4VueJs
- Sts+ Vue ：https://github.com/damienbod/AspNetCoreMvcVueJs
- net core swagger-ui-authorization：https://github.com/scottbrady91/IdentityServer4-Swagger-Integration
  - 参考项目文档：https://www.scottbrady91.com/identity-server/aspnet-core-swagger-ui-authorization-using-identityserver4

-----------------------------------------------------------------------------------------------------------

##  摘要

上述参考项目文档摘要：

​		Swagger 与 OAuth 授权服务器的集成相对有据可查，因此在本文中，您将了解使用 Swagger 将 IdentityServer 支持添加到 ASP.NET Core API 的基础知识，然后查看这种方法的局限性和一些替代方案可能值得探索。

​		本文将演示Swashbuckle和NSwag。

###  准备你的 API

您可以在您的`ConfigureServices`方法中注册此身份验证库：

```csharp
services.AddAuthentication("Bearer")
    .AddIdentityServerAuthentication("Bearer", options =>
    {
        // required audience of access tokens
        options.ApiName = "api1";

        // auth server base endpoint (this will be used to search for disco doc)
        options.Authority = "https://localhost:5000";
    });
```

`Configure`然后通过将您的方法 更新为如下所示在您的 HTTP 管道中启用它：

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseRouting();

    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints => endpoints.MapDefaultControllerRoute());
}
```

然后，您可以使用`AuthorizeAttribute`on 操作或控制器来触发此操作。您现在应该从这些受保护的端点获得 401 Unauthorized。

### 将 OAuth 支持添加到 Swashbuckle

回到您的 API，让我们引入 Swashbuckle：

```custom
dotnet add package Swashbuckle.AspNetCore
dotnet add package Swashbuckle.AspNetCore.Swagger
```

您可以通过将以下内容添加到您的`ConfigureServices`方法来注册：

```csharp
services.AddSwaggerGen(options =>
    {
        options.SwaggerDoc("v1", new OpenApiInfo {Title = "Protected API", Version = "v1"});
    
        // we're going to be adding more here...
    });
```

这将配置一个带有一些描述性信息的基本 Swagger 文档。

接下来，您想将一些有关您的授权服务器的信息添加到 swagger 文档中。由于您的 UI 将在最终用户的浏览器中运行，并且在该浏览器中运行的 JavaScript 将需要访问令牌，因此您将使用授权代码流（以及稍后用于代码交换的 Proof-Key （PKCE））。

因此，您需要告诉 Swashbuckle 您的授权和令牌端点的位置（检查您的 IdentityServer 迪斯科文档），以及它将使用的范围（其中键是范围本身，值是显示名称）。

```csharp
options.AddSecurityDefinition("oauth2", new OpenApiSecurityScheme
{
    Type = SecuritySchemeType.OAuth2,
    Flows = new OpenApiOAuthFlows
    {
        AuthorizationCode = new OpenApiOAuthFlow
        {
            AuthorizationUrl = new Uri("https://localhost:5000/connect/authorize"),
            TokenUrl = new Uri("https://localhost:5000/connect/token"),
            Scopes = new Dictionary<string, string>
            {
                {"api1", "Demo API - full access"}
            }
        }
    }
});
```

您现在需要告诉您的 swagger 文档哪些端点需要访问令牌才能工作，并且它们可以返回 401 和 403 响应。您可以使用 来执行此操作`IOperationFilter`，您可以在下面看到它（这已改编自[eShopOnContainers](https://github.com/dotnet-architecture/eShopOnContainers)示例存储库中的过滤器）。

```csharp
public class AuthorizeCheckOperationFilter : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        var hasAuthorize = 
          context.MethodInfo.DeclaringType.GetCustomAttributes(true).OfType<AuthorizeAttribute>().Any() 
          || context.MethodInfo.GetCustomAttributes(true).OfType<AuthorizeAttribute>().Any();

        if (hasAuthorize)
        {
            operation.Responses.Add("401", new OpenApiResponse { Description = "Unauthorized" });
            operation.Responses.Add("403", new OpenApiResponse { Description = "Forbidden" });

            operation.Security = new List<OpenApiSecurityRequirement>
            {
                new OpenApiSecurityRequirement
                {
                    [
                        new OpenApiSecurityScheme {Reference = new OpenApiReference 
                        {
                            Type = ReferenceType.SecurityScheme, 
                            Id = "oauth2"}
                        }
                    ] = new[] {"api1"}
                }
            };

        }
    }
}
```

在这里，您正在寻找所有带有 的控制器和操作`AuthorizeAttribute`，并告诉您的 Swagger 文档包括额外的可能响应，并且它需要一个具有特定范围授权的访问令牌，如安全定义中所定义。

然后，您可以像这样注册：

```csharp
options.OperationFilter<AuthorizeCheckOperationFilter>();
```

现在，您可以通过将以下内容添加到您的`Configure`方法中来配置管道中的 Swagger 文档端点和 Swagger UI：

```csharp
app.UseSwagger();
app.UseSwaggerUI(options =>
{
    options.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");

    options.OAuthClientId("demo_api_swagger");
    options.OAuthAppName("Demo API - Swagger");
    options.OAuthUsePkce();
});
```

在这里，您还要说明您希望 Swagger UI 用于授权请求的客户端 ID、显示名称以及它应该使用 PKCE。

[您可以在ASP.NET Core 文档](https://docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle) 的 ASP.NET Core API 中找到更全面的配置 Swashbuckle 的演练。

### 将 OAuth 支持添加到 NSwag

让我们看看如何使用 NSwag 实现同样的效果。第一步是通过 NuGet 引入 NSwag 库：

```custom
dotnet add package NSwag.AspNetCore
```

`ConfigureServices`现在，您可以通过将注册添加到您的方法 来将 Swagger 文档生成添加到您的项目中。

```csharp
services.AddOpenApiDocument(options =>
{
    options.DocumentName = "v1";
    options.Title = "Protected API";
    options.Version = "v1";

    // we're going to be adding more here...
});
```

接下来，您想将一些有关您的授权服务器的信息添加到 swagger 文档中。由于您的 UI 将在最终用户的浏览器中运行，并且在该浏览器中运行的 JavaScript 将需要访问令牌，因此您将使用授权代码流（以及稍后用于代码交换的 Proof-Key （PKCE））。

因此，您需要告诉 Swashbuckle 您的授权和令牌端点的位置（检查您的 IdentityServer 迪斯科文档），以及它将使用的范围（其中键是范围本身，值是显示名称）。

```csharp
options.AddSecurity("oauth2", new OpenApiSecurityScheme
{
    Type = OpenApiSecuritySchemeType.OAuth2,
    Flows = new OpenApiOAuthFlows
    {
        AuthorizationCode = new OpenApiOAuthFlow
        {
            AuthorizationUrl = "https://localhost:5000/connect/authorize",
            TokenUrl = "https://localhost:5000/connect/token",
            Scopes = new Dictionary<string, string> { { "api1", "Demo API - full access" } }
        }
    }
});
```

为了让 NSwag 了解哪些端点需要访问令牌并向 Swagger 文档添加安全范围，您可以使用`OperationSecurityScopeProcessor`该类自动扫描所有控制器和操作以获取`AuthorizationAttributes`.

```csharp
options.OperationProcessors.Add(new OperationSecurityScopeProcessor("oauth2"));
```

然后，您可以通过将以下内容添加到您的`Configure`方法中来启用管道中的 Swagger 文档和 UI：

```csharp
app.UseOpenApi();
app.UseSwaggerUi3(options =>
{
    options.OAuth2Client.ClientId = "demo_api_swagger";
    options.OAuth2Client.AppName = "Demo API - Swagger";
    options.OAuth2Client.UsePkceWithAuthorizationCodeGrant = true;
});
```

在这里，您还要说明您希望 Swagger UI 用于授权请求的客户端 ID、显示名称以及它应该使用 PKCE。

[您可以在ASP.NET Core 文档](https://docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-nswag) 中找到更全面的在 ASP.NET Core API 中配置 NSwag 的演练。

##  部分代码

### 安装Nuget

- IdentityServer4.AccessTokenValidation

- Swashbuckle.AspNetCore

- Swashbuckle.AspNetCore.Swagger

### startup.cs

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.OpenApi.Models;
using Swashbuckle.AspNetCore.SwaggerGen;

namespace Api.Swashbuckle
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();

            // 设置跨域
            services.AddCors(options =>
            {
                options.AddPolicy("AllowAllOrigins",
                    builder =>
                    {
                        builder
                            .AllowCredentials()
                            .WithOrigins("http://localhost:44357", "https://localhost:44357")  // change me！！
                            .SetIsOriginAllowedToAllowWildcardSubdomains()
                            .AllowAnyHeader()
                            .AllowAnyMethod();
                    });
            });

            services.AddAuthentication("Bearer")
                .AddIdentityServerAuthentication("Bearer", options =>
                {
                    options.ApiName = "Demo_Api_Source"; // change me！！
                    options.Authority = "https://localhost:44310";  // 认证地址 change me！！
                });

            services.AddSwaggerGen(options =>
            {
                options.SwaggerDoc("v1", new OpenApiInfo {Title = "Demo.SwaggerUi", Version = "v1"});

                options.AddSecurityDefinition("oauth2", new OpenApiSecurityScheme
                {
                    Type = SecuritySchemeType.OAuth2,
                    Flows = new OpenApiOAuthFlows
                    {
                        AuthorizationCode = new OpenApiOAuthFlow
                        {
                            AuthorizationUrl = new Uri("https://localhost:44310/connect/authorize"),
                            TokenUrl = new Uri("https://localhost:44310/connect/token"),
                            Scopes = new Dictionary<string, string>
                            {
                                //认证资源,一个或者更多..
                                { "Demo_Api_Scopes", "Demo Simple Scopes"}  // change me！！
                                // more...
                            },
                        },
                    }
                });

                options.OperationFilter<AuthorizeCheckOperationFilter>();
            });
        }

        public void Configure(IApplicationBuilder app)
        {
            app.UseDeveloperExceptionPage();
            app.UseHttpsRedirection();

            app.UseRouting();

            app.UseAuthentication();
            app.UseAuthorization();

            app.UseSwagger();
            app.UseSwaggerUI(options =>
            {
                options.SwaggerEndpoint("/swagger/v1/swagger.json", "My API");

                options.OAuthClientId("Demo_Api_SwaggerUi"); // Ids4 认证客户端名称  // change me！！
                options.OAuthAppName("Demo API - Swagger"); // 应用名称   // change me！！
                options.OAuthUsePkce();
            });
            // 设置跨域
            app.UseCors("AllowAllOrigins"); // change me！！
            app.UseEndpoints(endpoints => endpoints.MapDefaultControllerRoute());
        }
    }

    public class AuthorizeCheckOperationFilter : IOperationFilter
    {
        public void Apply(OpenApiOperation operation, OperationFilterContext context)
        {
            var hasAuthorize = context.MethodInfo.DeclaringType.GetCustomAttributes(true).OfType<AuthorizeAttribute>().Any() ||
                               context.MethodInfo.GetCustomAttributes(true).OfType<AuthorizeAttribute>().Any();

            if (hasAuthorize)
            {
                operation.Responses.Add("401", new OpenApiResponse { Description = "Unauthorized" });
                operation.Responses.Add("403", new OpenApiResponse { Description = "Forbidden" });

                // If the security scheme is of type "oauth2" or "openIdConnect", then the value is a list of scope
                // names required for the execution.
                operation.Security = new List<OpenApiSecurityRequirement> 
                {
                    new OpenApiSecurityRequirement
                    {
                        [
                          new OpenApiSecurityScheme {
                               Reference = new OpenApiReference {
                                   Type = ReferenceType.SecurityScheme, 
                                   Id = "oauth2",
                               }
                          }
                        ] = new[] { "Demo_Api_Scopes" } // change me！！
                    }
                };
            }
        }
    }
}

```

### WeatherForecastController

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace Api.Swashbuckle.Controllers
{
    [Authorize] // 可以设置角色，规则
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
        private static readonly string[] Summaries = {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };
        
        [HttpGet]
        public IEnumerable<WeatherForecast> Get()
        {
            var rng = new Random();
            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateTime.Now.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = Summaries[rng.Next(Summaries.Length)]
            })
            .ToArray();
        }
    }
}
```

##  附

Token在线解析工具：https://tooltt.com/jwt-decode/

Tips token中的aud：Ids4 toke 中设置aud:需要在api Scopes 中设置对应资源，并且在客户端中引用



## The End.