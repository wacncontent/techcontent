---
title: 通过 C 使用事件中心入门 | Azure
description: 遵循本教程开始使用 Azure 事件中心，以便通过事件处理程序主机，使用 C 程序发送事件和使用 Java 程序接收事件。
services: event-hubs
documentationcenter: ''
author: jtaubensee
manager: timlt
editor: ''

ms.assetid: aa6553f9-e12e-4568-9bf3-667f1c47a6cf
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: c
ms.devlang: csharp
ms.topic: article
ms.date: 01/04/2017
wacn.date: 02/24/2017
ms.author: jotaub;sethm
---

# 事件中心入门

[!INCLUDE [service-bus-selector-get-started](../../includes/service-bus-selector-get-started.md)]

## 介绍

事件中心是一个具备高度可伸缩性的引入系统，每秒可收入大量事件，从而使应用程序能够处理和分析连接的设备和应用程序所产生的海量数据。数据采集到事件中心后，可以使用任何实时分析提供程序或存储群集来转换和存储数据。

有关详细信息，请参阅[事件中心概述][Event Hubs overview]。

在本教程中，你将学习如何使用用 C 编写的控制台应用程序将消息引入事件中心，以及如何使用 C\# [事件处理程序主机][Event Processor Host]库并行检索这些消息。

若要完成本教程，你需要以下各项：

* C 语言开发环境。对于本教程，我们假设 gcc 堆栈位于使用 Ubuntu 14.04 的 [Azure Linux VM](../virtual-machines/virtual-machines-linux-quick-create-cli.md) 上。其他环境的说明将在外部链接中提供。

* Microsoft Visual Studio Express for Windows

* 有效的 Azure 帐户。<br/>如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="https://www.azure.cn/pricing/free-trial/" target="_blank">Azure 免费试用</a>。

[!INCLUDE [event-hubs-create-event-hub](../../includes/event-hubs-create-event-hub.md)]

[!INCLUDE [service-bus-event-hubs-get-started-send-c](../../includes/service-bus-event-hubs-get-started-send-c.md)]

[!INCLUDE [service-bus-event-hubs-get-started-receive-ephjava](../../includes/service-bus-event-hubs-get-started-receive-ephjava.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 运行 **Receiver** 项目。

    ![][21]

2. 运行 **Sender** 程序，然后就会看到事件出现在接收方窗口中。

    ![][24]

## 后续步骤

现在，你已生成了一个可以创建事件中心以及发送和接收数据的有效应用程序，接下来请继续学习以下方案：

* [使用事件中心的完整示例应用程序][sample application that uses Event Hubs]。
* [使用事件中心扩大事件处理][Scale out Event Processing with Event Hubs]示例。
* [事件中心概述][Event Hubs overview]

<!-- Images. -->
[21]: ./media/event-hubs-c-ephjava-getstarted/ephjava.png
[24]: ./media/event-hubs-c-ephjava-getstarted/receive-eph-c.png

<!-- Links -->

[Event Processor Host]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[Event Hubs overview]: ./event-hubs-overview.md
[sample application that uses Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[Scale out Event Processing with Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3

<!---HONumber=Mooncake_0220_2017-->
<!-- Update_Description: update meta properties; wording update; update link reference -->