---
title: 使用 C# 在 Linux 上创建第一个 Service Fabric 应用程序 | Azure
description: 使用 C# 创建并部署 Service Fabric 应用程序
services: service-fabric
documentationCenter: csharp
authors: mani-ramaswamy
manager: timlt
editor: ''

ms.service: service-fabric
ms.devlang: csharp
ms.topic: hero-article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/04/2016
wacn.date: 11/28/2016
ms.author: subramar
---

# 创建第一个 Azure Service Fabric 应用程序

> [!div class="op_single_selector"]
- [C# - Windows](./service-fabric-create-your-first-application-in-visual-studio.md)
- [Java - Linux](./service-fabric-create-your-first-linux-application-with-java.md)
- [C# - Linux](./service-fabric-create-your-first-linux-application-with-csharp.md)

Service Fabric 提供用于在 Linux 上构建服务的 .NET Core 和 Java SDK。本教程探讨如何创建适用于 Linux 的应用程序以及使用 C# (.NET Core) 构建服务。

## 先决条件

开始之前，请确保已[设置 Linux 开发环境](./service-fabric-get-started-linux.md)。如果使用的是 Mac OS X，可以[使用 Vagrant 在虚拟机中设置 Linux 一体式环境](./service-fabric-get-started-mac.md)。

## 创建应用程序

Service Fabric 应用程序可以包含一个或多个服务，每个服务都在提供应用程序功能时具有特定角色。适用于 Linux 的 Service Fabric SDK 包含 [Yeoman](http://yeoman.io/) 生成器，使用它可以轻松创建第一个服务并在以后添加更多服务。让我们使用 Yeoman 创建包含单个服务的应用程序。

1. 在终端中，键入以下命令开始构建基架：`yo azuresfcsharp`

2. 为应用程序命名。

3. 选择第一个服务的类型并为其命名。本教程选择了 Reliable Actor 服务。

  ![适用于 C# 的 Service Fabric Yeoman 生成器][sf-yeoman]  

>[!NOTE]
> 有关选项的详细信息，请参阅 [Service Fabric 编程模型概述](./service-fabric-choose-framework.md)。

## 构建应用程序

Service Fabric Yeoman 模板包含构建脚本，可用于从终端构建应用程序（在导航到应用程序文件夹后）。

  ```bash
 cd myapp 
 ./build.sh 
  ```

## 部署应用程序

构建应用程序后，可以使用 Azure CLI 将它部署到本地群集。

1. 连接到本地 Service Fabric 群集。

    ```bash
    azure servicefabric cluster connect
    ```

2. 使用模板中提供的安装脚本，将应用程序包复制到群集的映像存储、注册应用程序类型，然后创建应用程序的实例。

    ```bash
    ./install.sh
    ```

3. 打开浏览器并导航到 Service Fabric Explorer (http://localhost:19080/Explorer)（如果在 Mac OS X 上使用 Vagrant，请将 localhost 替换为 VM 的专用 IP）。

4. 展开“应用程序”节点。可以看到，应用程序类型现在有一个条目，另一个条目对应于该类型的第一个实例。

## 启动测试客户端并执行故障转移

执行组件项目没有任何属于自己的项。它们需要其他服务或客户端发送消息给它们。执行组件模板包含简单的测试脚本，可用于与执行组件服务交互。

1. 使用监视实用工具运行脚本，查看执行组件服务的输出。

    ```bash
    cd myactorsvcTestClient
    watch -n 1 ./testclient.sh
    ```

2. 在 Service Fabric Explorer 中，找到托管执行组件服务主副本的节点。在以下屏幕截图中，该节点是节点 3。

    ![在 Service Fabric Explorer 中查找主副本][sfx-primary]  

3. 单击在上一步骤中找到的节点，然后从“操作”菜单中选择“禁用(重新启动)”。此操作将重新启动本地群集中五个节点中的一个，强制故障转移到另一个节点上运行的辅助副本。执行此操作时，请注意测试客户端的输出，可以看到，尽管是故障转移，计数器仍继续递增。

## 后续步骤

- [详细了解 Reliable Actors](./service-fabric-reliable-actors-introduction.md)
- [使用 Azure CLI 来与 Service Fabric 群集交互](./service-fabric-azure-cli.md)

<!-- Images -->

[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-csharp/yeoman-csharp.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-csharp/sfx-primary.png

<!---HONumber=Mooncake_1121_2016-->