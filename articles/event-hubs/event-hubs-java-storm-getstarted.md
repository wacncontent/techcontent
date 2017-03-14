---
title: 使用 Apache Storm 通过 Java 使用事件中心入门 | Azure
description: 按照本教程开始使用 Azure 事件中心，通过 Java 发送事件，并在 Apache Storm 群集中接收这些事件。
services: event-hubs
documentationCenter: ''
authors: fsautomata
manager: timlt
editor: ''

ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/07/2016
wacn.date: 01/23/2017
ms.author: sethm
---

# 事件中心入门

[!INCLUDE [service-bus-selector-get-started](../../includes/service-bus-selector-get-started.md)]

## 介绍

事件中心是一个具备高度可伸缩性的引入系统，每秒可收入大量事件，从而使应用程序能够处理和分析连接的设备和应用程序所产生的海量数据。数据采集到事件中心后，可以使用任何实时分析提供程序或存储群集来转换和存储数据。

有关详细信息，请参阅[事件中心概述][]。

本教程介绍如何使用以 Java 编写的控制台应用程序将消息收集到事件中心，以及如何使用 Apache Storm 并行检索这些消息。

若要完成本教程，需要满足以下条件：

+ 配置为运行 [Maven](http://maven.apache.org/) 的 Java 开发环境。对于本教程，我们将采用 [Eclipse](https://www.eclipse.org/)。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 <a href="https://www.azure.cn/pricing/1rmb-trial/" target="_blank">Azure 试用</a>。

[!INCLUDE [event-hubs-create-event-hub](../../includes/event-hubs-create-event-hub.md)]

[!INCLUDE [service-bus-event-hubs-get-started-send-java](../../includes/service-bus-event-hubs-get-started-send-java.md)]

[!INCLUDE [service-bus-event-hubs-get-started-receive-storm](../../includes/service-bus-event-hubs-get-started-receive-storm.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 从 Eclipse 中运行 **LogTopology** 类，然后等待它为所有分区启动接收方。

2. 运行 **Sender** 项目，在控制台窗口中按 **Enter**，然后会看到事件出现在接收方窗口中。

       ![][22]

> [!NOTE]
>  在本教程中，只出于开发目的以本地模式使用 Storm。请参阅 [HDInsight Storm 概述][]和官方 [Apache Storm][] 文档，以了解 Storm 部署和模式的详细信息。

## 后续步骤

以下资源可用于开发集成事件中心和 Storm 的应用程序：

- [用 Storm 和 HDInsight 分析传感器数据]是一个完整方案教程，它使用了事件中心、Storm 和 HBase，可在 Hadoop 群集中引入传感器数据。
<!-- - [使用 SCP.NET 和 C# 在 Storm 和 HDInsight 上开发流式数据处理应用程序][]是有关使用 C# 编写 Storm 管道的教程。 -->

<!-- Images. -->
[22]: ./media/event-hubs-java-storm-getstarted/receive-storm2.png

<!-- Links -->
[Azure Management Portal]: https://manage.windowsazure.cn/
[Event Processor Host]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[事件中心概述]: ./event-hubs-overview.md

[Apache Storm]: https://storm.incubator.apache.org
[HDInsight Storm 概述]: ../hdinsight/hdinsight-storm-overview.md
[用 Storm 和 HDInsight 分析传感器数据]: ../hdinsight/hdinsight-storm-sensor-data-analysis.md
[使用 SCP.NET 和 C# 在 Storm 和 HDInsight 上开发流式数据处理应用程序]: /documentation/articles/hdinsight-hadoop-storm-scpdotnet-csharp-develop-streaming-data-processing-application/

<!---HONumber=Mooncake_0116_2017-->
<!--Update_Description:update wording-->