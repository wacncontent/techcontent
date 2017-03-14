---
title: Reliable Actors 框架中的多态性技术 | Azure
description: 在 Reliable Actors 框架中构建 .NET 接口和类型的层次结构，以便重用功能和 API 定义。
services: service-fabric
documentationcenter: .net
author: seanmck
manager: timlt
editor: vturecek

ms.assetid: ef0eeff6-32b7-410d-ac69-87cba8b8fd46
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 12/13/2016
wacn.date: 01/25/2017
ms.author: seanmck
---

# Reliable Actors 框架中的多态性技术
Reliable Actors 框架允许你使用许多在面向对象的设计中使用的相同技术来生成执行组件。其中一种技术是多态性技术，它允许类型和接口从多个通用父类中继承。Reliable Actors 框架中的继承通常遵循 .NET 模型，并会受到一些附加限制。

## 接口
Reliable Actors 框架要求至少定义一个要由执行组件类型实现的接口。此接口用于生成代理类，客户端可以使用此代理类与你的执行组件进行通信。只要由执行组件类型实现的每个接口及其所有父接口最终派生自 IActor，就可以从其他接口继承接口。IActor 是执行组件的平台定义的基类接口。因此，使用形状的经典多态性示例可能如下所示：

![形状执行组件的接口层次结构][shapes-interface-hierarchy]

## 类型
你还可以创建执行组件类型的层次结构，这些类型派生自由平台提供的执行组件基类。如果是形状，你可能具有一个 `Shape` 基类型：

```
public abstract class Shape : Actor, IShape
{
    public abstract Task<int> GetVerticeCount();

    public abstract Task<double> GetAreaAsync();
}
```

`Shape` 的子类型可以重写基类型的方法。

```
[ActorService(Name = "Circle")]
[StatePersistence(StatePersistence.Persisted)]
public class Circle : Shape, ICircle
{
    public override Task<int> GetVerticeCount()
    {
        return Task.FromResult(0);
    }

    public override async Task<double> GetAreaAsync()
    {
        CircleState state = await this.StateManager.GetStateAsync<CircleState>("circle");

        return Math.PI *
            state.Radius *
            state.Radius;
    }
}
```

请注意执行组件类型中的 `ActorService` 属性。此属性告知 Reliable Actor 框架，它应自动创建用于托管此类型的执行组件的服务。在某些情况下，你可能想要创建仅用于与子类型共享功能，并且始终不会用于实例化具体的执行组件的基类型。在这些情况下，应使用 `abstract` 关键字表示你始终不会基于此类型创建执行组件。

## 后续步骤

- 请参阅 [Reliable Actors 框架如何使用 Service Fabric 平台](./service-fabric-reliable-actors-platform.md)，以提供可靠性、可伸缩性和一致状态。
- 了解有关[执行组件生命周期](./service-fabric-reliable-actors-lifecycle.md)的信息。

<!-- Image references -->

[shapes-interface-hierarchy]: ./media/service-fabric-reliable-actors-polymorphism/Shapes-Interface-Hierarchy.png

<!---HONumber=Mooncake_Quality_Review_0125_2017-->
<!--update: -->