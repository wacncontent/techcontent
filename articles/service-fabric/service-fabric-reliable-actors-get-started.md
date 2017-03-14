---
title: Service Fabric Reliable Actors 入门 | Azure
description: 本教程介绍了使用 Service Fabric Reliable Actors 创建、调试和部署简单的基于执行组件的服务的步骤。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''

ms.assetid: d4aebe72-1551-4062-b1eb-54d83297f139
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/04/2017
wacn.date: 02/20/2017
ms.author: vturecek
---

# Reliable Actors 入门
本文介绍了 Azure Service Fabric Reliable Actors 的基础知识，并演示了如何在 Visual Studio 中创建、调试和部署简单的 Reliable Actor 应用程序。

## 安装和设置
在开始之前，确保你的计算机上已设置 Service Fabric 开发环境。如果需要设置此环境，请参阅有关[如何设置开发环境](./service-fabric-get-started.md)的详细说明。

## 基本概念
若要开始使用 Reliable Actors，只需了解几个基本概念：

 * **执行组件服务**。可以在 Service Fabric 基础结构中部署的 Reliable Services 中打包 Reliable Actors。执行组件实例在命名的服务实例中激活。

 * **执行组件注册**。与 Reliable Services 一样，Reliable Actor 服务也需要注册到 Service Fabric 运行时。此外，需要将执行组件类型注册到执行组件运行时。

 * **执行组件接口**。执行组件接口用于定义执行组件的强类型公共接口。在 Reliable Actor 模型术语中，执行组件接口用于定义执行组件可以理解并处理的消息类型。其他执行组件或客户端应用程序使用此执行组件接口将消息“发送”（异步方式）到此执行组件。Reliable Actors 可实现多个接口。

 * **ActorProxy 类**。客户端应用程序使用 ActorProxy 类调用通过执行组件接口公开的方法。ActorProxy 类提供两个重要功能：
    * 名称解析：能够在群集中找到执行组件（查找托管它的群集节点）。
    * 故障处理：例如，在需要将执行组件重新定位到群集中另一个节点的故障之后，它可以重试方法调用和重新解析执行组件位置。

有必要提一下与执行组件接口有关的以下规则：

- 不能重载执行组件接口方法。
- 执行组件接口方法不能有输出参数、引用参数或可选参数。
- 不支持泛型接口。

## 在 Visual Studio 中创建新项目
安装了适用于 Visual Studio 的 Service Fabric 工具之后，你可以创建新的项目类型。新的项目类型位于“新建项目”对话框的“云”类别下面。

![适用于 Visual Studio 的 Service Fabric 工具 - 新建项目][1]  

在下一个对话框中，可以选择你想要创建的项目类型。

![Service Fabric 项目模板][5]  

对于 HelloWorld 项目，我们使用 Service Fabric Reliable Actors 服务。

创建此解决方案之后，可看到以下结构：

![Service Fabric 项目结构][2]  

## Reliable Actors 基本构建基块

典型的 Reliable Actors 解决方案由 3 个项目组成：

* **应用程序项目 (MyActorApplication)**。这是将所有服务打包在一起以进行部署的项目。它包含用于管理应用程序的 *ApplicationManifest.xml* 和 PowerShell 脚本。

* **接口项目 \(MyActor.Interfaces\)**。这是包含执行组件的接口定义的项目。在 MyActor.Interfaces 项目中，你可以定义在解决方案中执行组件所使用的接口。可在任何项目中使用任何名称定义执行组件接口。不过，因为该接口定义了执行组件实现和调用执行组件的客户端所共享的执行组件协定，所以通常合理的做法是在独立于执行组件实现的程序集中定义接口，并且其他多个项目可以共享接口。

    ```
    public interface IMyActor : IActor
    {
        Task<string> HelloWorld();
    }
    ```

* **执行组件服务项目 \(MyActor\)**。这是用于定义要托管执行组件的 Service Fabric 服务的项目。它包含执行组件的实现。执行组件实现是派生自基类型 `Actor` 的一个类，用于实现在 MyActor.Interfaces 项目中定义的接口。执行组件类还必须实现一个构造函数，该函数接受一个 `ActorService` 实例和一个 `ActorId`，并将它们传递给基 `Actor` 类。这样即可注入平台依赖项，完成构造函数依赖项注入。

    ```
    [StatePersistence(StatePersistence.Persisted)]
    class MyActor : Actor, IMyActor
    {
        public MyActor(ActorService actorService, ActorId actorId)
            : base(actorService, actorId)
        {
        }
        public Task<string> HelloWorld()
        {
            return Task.FromResult("Hello world!");
        }
    }
    ```

执行组件服务必须注册到 Service Fabric 运行时中的服务类型。为了使执行组件服务能够运行执行组件实例，还必须向执行组件服务注册你的执行组件类型。`ActorRuntime` 注册方法将为执行组件执行此操作。

```
internal static class Program
{
    private static void Main()
    {
        try
        {
            ActorRuntime.RegisterActorAsync<MyActor>(
                (context, actorType) => new ActorService(context, actorType, () => new MyActor())).GetAwaiter().GetResult();

            Thread.Sleep(Timeout.Infinite);
        }
        catch (Exception e)
        {
            ActorEventSource.Current.ActorHostInitializationFailed(e.ToString());
            throw;
        }
    }
}
```

如果在 Visual Studio 中从新项目开始，并且只有一个执行组件定义，那么默认情况下，在 Visual Studio 生成的代码中包含此注册。如果在服务中定义其他执行组件，则需要使用以下操作添加执行组件注册：

```
ActorRuntime.RegisterActorAsync<MyOtherActor>();
```

> [!TIP]
> Service Fabric 执行组件运行时发出一些[与执行组件方法相关的事件和性能计数器](./service-fabric-reliable-actors-diagnostics.md#actor-method-events-and-performance-counters)。它们可用于进行诊断和性能监视。

## 调试

适用于 Visual Studio 的 Service Fabric 工具支持在本地计算机上进行调试。你可以通过点击 F5 键启动调试会话。Visual Studio 将生成程序包（如果需要）。Visual Studio 还将在本地 Service Fabric 群集中部署应用程序，并附加调试器。

在部署过程中，可以在“输出”窗口中查看进度。

![Service Fabric 调试输出窗口][3]  

## 后续步骤
 - [Reliable Actors 如何使用 Service Fabric 平台](./service-fabric-reliable-actors-platform.md)
 - [执行组件状态管理](./service-fabric-reliable-actors-state-management.md)
 - [执行组件生命周期和垃圾回收](./service-fabric-reliable-actors-lifecycle.md)
 - [执行组件 API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/dn971626.aspx)
 - [代码示例](https://github.com/Azure/servicefabric-samples)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject.PNG
[2]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-projectstructure.PNG
[3]: ./media/service-fabric-reliable-actors-get-started/debugging-output.PNG
[4]: ./media/service-fabric-reliable-actors-get-started/vs-context-menu.png
[5]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject1.PNG

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: update code sample -->