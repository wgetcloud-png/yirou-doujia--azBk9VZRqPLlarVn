
[![](https://img2024.cnblogs.com/blog/47875/202412/47875-20241205121340970-720741815.svg)](https://github.com)


.NET Aspire 是一组功能强大的工具、模板和包，用于构建可观察的生产就绪应用程序。.NET Aspire 通过处理特定云原生问题的 NuGet 包集合提供。云原生应用程序通常由小型互连部分或微服务组成，而不是单个整体式代码库。云原生应用程序通常会消耗大量的服务，例如数据库、消息收发和缓存。


.NET Aspire 旨在改善构建 .NET 云原生应用程序的体验。它提供了一组一致的、有主见的工具和模式，可帮助您构建和运行分布式应用程序。NET Aspire 旨在帮助您：


* 编排：.NET Aspire 为本地开发环境提供了运行和连接多项目应用程序及其依赖项的功能。
* 集成：.NET Aspire 集成是适用于常用服务（如 Redis 或 Postgres）的 NuGet 包，具有标准化接口，可确保它们与您的应用程序一致且无缝地连接。
* 工具：.NET Aspire 附带适用于 Visual Studio、Visual Studio Code 和 .NET CLI 的项目模板和工具体验，可帮助你创建 .NET Aspire 项目并与之交互。


# 前提条件


* .NET 8\.0 或 .NET 9\.0
* 符合 OCI 标准的容器运行时，例如：
	+ [Docker Desktop](https://github.com) 或[Podman](https://github.com)。
* 集成开发人员环境 （IDE） 或代码编辑器，例如：
	+ [Visual Studio 2022](https://github.com) 版本 17\.9 或更高版本（可选）
	+ [Visual Studio Code](https://github.com)（可选）
		- [C\# 开发工具包：扩展](https://github.com)（可选）
	+ [带有 .NET Aspire 插件的 JetBrains Rider](https://github.com)（（可选）


# 安装.NET Aspire 模板


如果尚未安装 [.NET Aspire 模板](https://github.com)，请运行以下命令：



```
dotnet new install Aspire.ProjectTemplates

```

完成安装后，执行一下命令可看到aspire项目模板：



```
dotnet new list aspire

模板名                        短名称                  语言  标记
----------------------------  ----------------------  ----  -------------------------------------------------------
.NET Aspire 入门应用          aspire-starter          [C#]  Common/.NET Aspire/Blazor/Web/Web API/API/Service/Cloud
.NET Aspire 应用主机          aspire-apphost          [C#]  Common/.NET Aspire/Cloud
.NET Aspire 服务默认值        aspire-servicedefaults  [C#]  Common/.NET Aspire/Cloud/Web/Web API/API/Service
.NET Aspire 测试项目(MSTest)  aspire-mstest           [C#]  Common/.NET Aspire/Cloud/Web/Web API/API/Service/Test
.NET Aspire 测试项目(NUnit)   aspire-nunit            [C#]  Common/.NET Aspire/Cloud/Web/Web API/API/Service/Test
.NET Aspire 测试项目(xUnit)   aspire-xunit            [C#]  Common/.NET Aspire/Cloud/Web/Web API/API/Service/Test
.NET Aspire 空应用            aspire                  [C#]  Common/.NET Aspire/Cloud/Web/Web API/API/Service

```

从模板创建 .NET Aspire 空应用，请运行以下命令:



```
dotnet new aspire -o Stargazer

```

创建的应用是一个最小的 .NET Aspire 项目，包括以下内容：


* [**Stargazer.AppHost**](https://github.com)：一个业务流程协调程序项目，旨在连接和配置应用程序的不同项目和服务。
* [**Stargazer.ServiceDefaults**](https://github.com):[westworld加速器](https://xbsj9.com)：一个 .NET Aspire 共享项目，用于管理在解决方案中与[弹性](https://github.com)、[服务发现](https://github.com)和[遥测](https://github.com)相关的项目中重复使用的配置。


# 集成服务


加入适用于常用服务（如 Redis 或 Postgres）的 NuGet 包`Aspire.Hosting.PostgreSQL`、`Aspire.Hosting.Redis`、`Aspire.Hosting.MongoDB`，然后在代码中创建docker容器：



```
using System.Runtime.InteropServices;

var builder = DistributedApplication.CreateBuilder(args);

string redisImage = "hub.atomgit.com/amd64/redis";
string postgresqlImage = "hub.atomgit.com/amd64/postgres";
string mongodbImage = "hub.atomgit.com/amd64/mongo";
Architecture architecture = RuntimeInformation.ProcessArchitecture;
if(architecture == Architecture.Arm
   || architecture == Architecture.Arm64)
{
    redisImage = "hub.atomgit.com/arm64v8/redis";
    postgresqlImage = "hub.atomgit.com/arm64v8/postgres";
    mongodbImage = "hub.atomgit.com/arm64v8/mongo";
}
    
var redis = builder.AddRedis("redis", 6379)
    .WithContainerName("redis")
    .WithImage(redisImage, "7-alpine")
    .WithDataVolume("redis")
    .WithRedisCommander(null, "redis-commander");

var username = builder.AddParameter("postgres-uid", "postgres");
var password = builder.AddParameter("postgres-pwd", "123456");
var postgres = builder.AddPostgres("postgres", username, password, 5432)
    .WithContainerName("postgres")
    .WithImage(postgresqlImage, "15-alpine")
    .WithDataVolume("postgres");
var postgresql = postgres.AddDatabase("postgresql");

var mongoUser = builder.AddParameter("mongo-user", "root");
var mongoPwd = builder.AddParameter("mongo-pwd", "123456");
var mongo = builder.AddMongoDB("mongo", 27017, mongoUser, mongoPwd)
    .WithContainerName("mongo")
    .WithImage(mongodbImage, "7-jammy")
    .WithDataVolume("mongo");
var mongodb = mongo.AddDatabase("mongodb");

IResourceBuilder<ProjectResource> apiService = builder.AddProject<Projects.Stargazer_Abp_Template_Host>("api-service");

builder.AddProject<Projects.Stargazer_Abp_Template_Web>("frontend")
    .WithExternalHttpEndpoints()
    .WithReference(redis)
    .WithReference(postgresql)
    .WithReference(mongodb)
    .WaitFor(redis)
    .WaitFor(postgres)
    .WaitFor(mongodb)
    .WithReference(apiService);

builder.Build().Run();

```

# 启动应用程序


运行以下命令启动应用程序：



```
dotnet run --project Stargazer.AppHost

```

[![](https://img2024.cnblogs.com/blog/47875/202412/47875-20241205121400690-1873407022.png)](https://github.com)
访问`https://localhost:17125/login?t=337c3ec0bfdadd302fcdb467d76453ad`，就可以使用.NET Aspire 仪表板。
[![](https://img2024.cnblogs.com/blog/47875/202412/47875-20241205121412783-202653399.png)](https://github.com)
访问仪表板上的链接`http://localhost:5136/`，就可以访问应用程序。
[![](https://img2024.cnblogs.com/blog/47875/202412/47875-20241205121423452-1866623736.png)](https://github.com)
首发网站：[https://stargazer.tech/2024/12/05/build\-your\-dotnet\-aspire\-solution/](https://github.com)
相关链接


* [.NET Aspire 官方文档 https://learn.microsoft.com/zh\-cn/dotnet/aspire/](https://github.com)
* [本文代码 https://github.com/huangmingji/Stargazer.Abp.Template/tree/main/modules/Aspire](https://github.com)


