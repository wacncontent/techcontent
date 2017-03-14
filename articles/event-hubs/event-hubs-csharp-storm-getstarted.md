---
title: 使用 Apache Storm 通过 C# 使用事件中心入门 | Azure
description: 遵循本教程开始使用 Azure 事件中心，以通过 C# 发送事件，并在 Apache Storm 群集中接收这些事件。
services: event-hubs
documentationCenter: ''
authors: jtaubensee
manager: timlt
editor: ''

ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/07/2016
wacn.date: 01/23/2017
ms.author: jotaub;sethm
---

# 事件中心入门

[!INCLUDE [service-bus-selector-get-started](../../includes/service-bus-selector-get-started.md)]

## 介绍

事件中心是一个具备高度可伸缩性的引入系统，每秒可收入大量事件，从而使应用程序能够处理和分析你连接的设备和应用程序所产生的海量数据。将数据采集到事件中心后，你可以使用任何实时分析提供程序或存储群集来转换和存储数据。

有关详细信息，请参阅[事件中心概述]。

在本教程中，你将学习如何使用用 C# 编写的控制台应用程序将消息引入事件中心，以及如何使用 Apache Storm 并行检索这些消息。

为了完成本教程，你将需要以下内容：

+ [Microsoft Visual Studio](http://visualstudio.com)

+ 一个 Java 开发环境，配置为运行 [Maven](http://maven.apache.org/)。对于本教程，我们将采用 [Eclipse](https://www.eclipse.org/)。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需几分钟的时间就能创建一个试用帐户。有关详细信息，请参阅 <a href="https://www.azure.cn/pricing/1rmb-trial">Azure 试用</a>。

[!INCLUDE [event-hubs-create-event-hub](../../includes/event-hubs-create-event-hub.md)]

[!INCLUDE [service-bus-event-hubs-get-started-send-csharp](../../includes/service-bus-event-hubs-get-started-send-csharp.md)]

[!INCLUDE [service-bus-event-hubs-get-started-receive-storm](../../includes/service-bus-event-hubs-get-started-receive-storm.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 从 Eclipse 中运行 **LogTopology** 类，然后等待它为所有分区启动接收方。

2. 运行 **Sender** 项目，在控制台窗口中按 **Enter**，然后会看到事件出现在接收方窗口中。

    ![][22]

## 后续步骤

现在，你已生成了一个可以创建事件中心以及发送和接收数据的有效应用程序，接下来请继续学习以下方案：

- [使用事件中心的完整示例应用程序][]
- [使用事件中心扩大事件处理][]示例

<!-- Images. -->

[22]: ./media/event-hubs-csharp-storm-getstarted/receive-storm1.png

<!-- Links -->
[Azure classic portal]: https://manage.windowsazure.cn/
[事件中心概述]: ./event-hubs-overview.md
[使用事件中心的完整示例应用程序]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Event-Hub-286fd097
[使用事件中心扩大事件处理]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Event-Hub-45f43fc3

<!---HONumber=Mooncake_Quality_Review_1230_2016-->
<!--Update_Description:update meta properties-->