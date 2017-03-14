---
title: Apache Storm on HDInsight 简介 | Azure
description: 获取有关 Apache Storm 的简介，并了解如何使用 Storm on HDInsight 在云中构建实时数据分析解决方案。
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal

ms.assetid: 72d54080-1e48-4a5e-aa50-cce4ffc85077
ms.service: hdinsight
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/11/2017
wacn.date: 01/25/2017
ms.author: larryfr
---

# Apache Storm on HDInsight 简介：面向 Hadoop 的实时分析

Apache Storm on HDInsight 可让你使用 [Apache Hadoop](http://hadoop.apache.org) 在 Azure 环境中创建分布式实时分析解决方案。

## 什么是 Apache Storm？

Apache Storm 是分布式可容错的开源计算系统，可用于配合 Hadoop 实时处理数据。Storm 解决方案还提供有保障的数据处理功能，能够重播第一次未成功处理的数据。

## 为何要使用 Storm on HDInsight？

Apache Storm on HDInsight 是已集成到 Azure 环境中的托管群集。HDInsight 上的 Storm 和其他 Hadoop 组件基于 Hortonworks 数据平台 (HDP)，而群集的操作系统则为 Ubuntu（Linux 分发）。这种情况下提供的平台十分兼容 Hadoop 生态系统中的常用工具和服务。

[!INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [!IMPORTANT]
Linux 是在 HDInsight 3.4 或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](./hdinsight-component-versioning.md#hdi-version-32-and-33-nearing-deprecation-date)。

Apache Storm on HDInsight 具有下述主要优势：

* 以托管服务的形式执行，提供 99.9% 运行时间的 SLA。

* 可以在创建期间或创建后针对群集运行脚本，轻松地进行自定义。有关详细信息，请参阅[使用脚本操作自定义 HDInsight 群集](./hdinsight-hadoop-customize-cluster-linux.md)。

* 使用所选语言：支持以 **Java**、**C#** 和 **Python** 编写的 Storm 组件。

    * 通过 Visual Studio 集成 HDInsight，适用于开发、管理和监视 C# 拓扑。有关详细信息，请参阅[通过用于 Visual Studio 的 HDInsight 工具开发 C# Storm 拓扑](./hdinsight-storm-develop-csharp-visual-studio-topology.md)。

    * 支持 **Trident** Java 接口。可以使用此接口创建支持“一次性”消息处理、“事务性”数据存储持久性和一组常见流分析操作的 Storm 拓扑。

* 轻松地对群集进行上下伸缩：添加或删除辅助角色节点，不影响 Storm 拓扑的运行。

* 与其他 Azure 服务（包括事件中心、Azure 虚拟网络、SQL 数据库、Blob 存储和 DocumentDB）集成。

    * 通过使用 Azure 虚拟网络，安全地组合多个 HDInsight 群集的功能：创建使用 HDInsight、HBase 或 Hadoop 群集的分析管道。

有关在实时分析解决方案中使用 Apache Storm 的公司列表，请参阅[使用 Apache Storm 的公司](https://storm.apache.org/documentation/Powered-By.html)。

若要开始使用 Storm，请参阅 [Storm on HDInsight 入门][gettingstarted]。

### 易于设置

你可以在分钟数设置好新的 Storm on HDInsight 群集。指定群集名称、大小、管理员帐户和存储帐户。Azure 将创建该群集，包括示例拓扑和 Web 管理仪表板。

> [!NOTE]
也可使用 [Azure CLI](../xplat-cli-install.md) 或 [Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs) 预配 Storm 群集。

在提交请求后的 15 分钟内，你就可以运行新的 Storm 群集，并准备好建立第一个实时分析管道。

### 易于使用

* __安全的 Shell 连接__：可以使用 SSH 通过 Internet 访问 HDInsight 群集的头节点。因此，用户可以直接在群集上运行命令。

    有关详细信息，请参阅[将 SSH 与 HDInsight 配合使用](./hdinsight-hadoop-linux-use-ssh-unix.md)。

* __Web 连接__：HDInsight 群集提供 Ambari Web UI。这样即可轻松地监视、配置和管理群集上的服务。Storm on HDInsight 还提供 Storm UI，适用于监视和管理通过浏览器运行 Storm 拓扑的操作。

    有关详细信息，请参阅[使用 Ambari Web UI 管理 HDInsight](./hdinsight-hadoop-manage-ambari.md) 和[使用 Storm UI 进行监视和管理](./hdinsight-storm-deploy-monitor-topology-linux.md#monitor-and-manage-using-the-storm-ui)。

* __Azure PowerShell 和 CLI__：Azure PowerShell 和 Azure CLI 均提供命令行实用工具，可以使用这些工具通过客户端系统使用 HDInsight 和其他 Azure 服务。

* __Visual Studio 集成__：用于 Visual Studio 的 Data Lake 工具（以前称为“用于 Visual Studio 的 HDInsight 工具”）包括适用于创建 C# Storm 拓扑的项目模板，以及适用于 Storm on HDInsight 的工具。可以在 Visual Studio 中创建、部署、监视和管理 C# 拓扑。

    有关详细信息，请参阅[通过用于 Visual Studio 的 HDInsight 工具开发 C# Storm 拓扑](./hdinsight-storm-develop-csharp-visual-studio-topology.md)。

* __与其他 Azure 服务集成__

    * 进行 __Java__ 开发时，Microsoft 会尽可能利用现有的 Storm 组件与其他 Azure 服务集成。在某些情况下，可能需要特定于服务的组件或解决方案。

        * __Azure Blob 存储__（当用作 HDInsight 的存储时）：基于 Java 的拓扑可以访问与群集关联的 Azure Blob 存储，方法是将 Storm-HDFS Bolt 与 URI 方案 `wasb://` 配合使用。

        * __Azure 事件中心__：可以使用 Microsoft 提供的 EventHubSpout 和 EventHubBolt 组件进行访问。这些组件是用 Java 编写的，作为独立的 .jar 文件提供。

        有关如何开发 Java 解决方案的详细信息，请参阅[为 Storm on HDInsight 开发基于 Java 的拓扑](./hdinsight-storm-develop-java-topology.md)。

    * 进行 __C#__ 开发时，通常可以将 .NET SDK 用于 Azure 服务。在某些情况下，SDK 可能依赖于无法在 Linux（适用于 HDInsight 3.4 及更高版本的主机 OS）上使用的框架。 这种情况下，可以使用 C# 解决方案中的 Java 组件。

        * 适用于 __SQL DB__、__DocumentDB__、__EventHub__ 和 __HBase__ 的示例作为模板包括在用于 Visual Studio 的 Azure Data Lake 工具中。有关详细信息，请参阅[为 Storm on HDInsight 开发 C# 拓扑](./hdinsight-storm-develop-csharp-visual-studio-topology.md)。

        * __Azure 事件中心__：有关使用 C# 解决方案中的 Java 组件的示例，请参阅[使用 Storm on HDInsight 从 Azure 事件中心处理事件 (C#)](./hdinsight-storm-develop-csharp-event-hub-topology.md)。

        有关如何开发 C# 解决方案的详细信息，请参阅[为 Storm on HDInsight 开发 C# 拓扑](./hdinsight-storm-develop-csharp-visual-studio-topology.md)。

### 可靠性

Apache Storm 始终保证每个传入消息将完全处理，即使数据分析分散在数百个节点。

**Nimbus 节点**提供的功能与 Hadoop JobTracker 类似，它通过 **Zookeeper** 将任务分配给群集中的其他节点。Zookeeper 节点为群集提供协调功能，并促进 Nimbus 与辅助节点上的 **Supervisor** 进程进行通信。如果处理的一个节点出现故障，Nimbus 节点将得到通知，并分配到另一个节点的任务和关联的数据。

Apache Storm 的默认配置是只能有一个 Nimbus 节点。Storm on HDInsight 运行两个 Nimbus 节点。如果主节点出现故障，HDInsight 群集将切换到辅助节点，同时主节点将会恢复。

![nimbus、zookeeper 和 supervisor 示意图](./media/hdinsight-storm-overview/nimbus.png)

### 缩放

虽然可以在创建过程中指定群集中的节点数，但你可能需要扩大或收缩群集以匹配工作负载。所有 HDInsight 群集允许你更改群集中的节点数，即使在处理数据时。

> [!NOTE]
若要利用通过缩放添加的新节点，你需要重新平衡在增加大小之前启动的拓扑。

### 支持

Storm on HDInsight 附带全天候企业级支持。Storm on HDInsight 也提供 99.9% 的 SLA。这意味着，我们保证至少 99.9% 的时间群集都能建立外部连接。

## 实时分析常见用例

以下是你可能使用 Apache storm on HDInsight 的一些常见方案。有关实际情景的信息，请阅读[各公司如何使用 Storm](https://storm.apache.org/documentation/Powered-By.html)。

* 物联网 (IoT)
* 欺诈检测
* 社交分析
* 提取、转换、加载 (ETL)
* 网络监视
* 搜索
* 移动应用场景

## 如何处理 HDInsight Storm 中的数据？

Apache Storm 运行**拓扑**，而不是 HDInsight 或 Hadoop 中用户熟悉的 MapReduce 作业。Storm on HDInsight 群集包含两种类型的节点：运行 **Nimbus** 的头节点和运行 **Supervisor** 的辅助节点。

* **Nimbus**：类似于 Hadoop 中的 JobTracker，负责在整个群集中分发代码、将任务分配给虚拟机以及监视故障情况。HDInsight 提供两个 Nimbus 节点，因此 Storm on HDInsight 不会出现单点故障
* **Supervisor**：每个辅助节点的 supervisor 负责启动和停止该节点上的**工作进程**。
* **工作进程**：运行**拓扑**的一个子集。正在运行的拓扑分布在整个群集的许多工作进程上。
* **拓扑**：定义处理数据**流**的计算图形。与 MapReduce 作业不同，拓扑运行到你停止它们为止。
* **流**：一个未绑定的**元组**集合。流由 **spout** 和 **bolt** 生成，并由 **bolt** 使用。
* **元组**：动态类型化值的一个命名列表。
* **Spout**：使用数据源中的数据并发出一个或多个**流**。

    > [!NOTE]
    在许多情况下，数据是从队列（例如 Azure 事件中心）读取的。队列确保发生中断时数据持续不断。

* **Bolt**：使用**流**，处理**元组**，并可以发出**流**。Bolt 还负责将数据编写到外部存储，比如队列、HDInsight HBase、blob 或其他数据存储。
* **Apache Thrift**：用于可缩放跨语言服务开发的软件框架。可用于构建在 C++、Java、Python、PHP、Ruby、Erlang、Perl、Haskell、C#、Cocoa、JavaScript、Node.js、Smalltalk 及其他语言间工作的服务。

    * **Nimbus** 是一种 Thrift 服务，**拓扑**是 Thrift 定义，因此可以使用各种编程语言来开发拓扑。

有关 Storm 组件的详细信息，请参阅 apache.org 上的 [Storm 教程][apachetutorial]。

## 我可以使用哪些编程语言？

Storm on HDInsight 群集支持 C#、Java 和 Python。

### C&#35;

用于 Visual Studio 的 Data Lake 工具允许 .NET 开发人员以 C# 语言设计和实现拓扑。你也可以创建使用 Java 和 C# 组件的混合拓扑。

有关详细信息，请参阅[使用 Visual Studio 开发 Apache Storm on HDInsight 的 C# 拓扑](./hdinsight-storm-develop-csharp-visual-studio-topology.md)。

### Java

你遇到的大多数 Java 示例都是无格式 Java 或 Trident。Trident 是一个高级别抽象，可更轻松地执行联接、汇总、分组和筛选等操作。但是，Trident 作用于批量元组，其中原始 Java 解决方案一次将处理一个元组流。

有关 Trident 的详细信息，请参阅 apache.org 上的 [Trident 教程](https://storm.apache.org/documentation/Trident-tutorial.html)。

有关 Java 和 Trident 拓扑的示例，请参阅 [Storm 拓扑示例列表](./hdinsight-storm-example-topology.md)或 HDInsight 群集上的 storm-starter 示例。

storm-starter 示例位于 HDInsight 群集的 ** /usr/hdp/current/storm-client/contrib/storm-starter** 目录中。

## 常见的开发模式有哪些？

### 有保证的消息处理

Storm 可以提供不同级别的有保证的消息处理。例如，基本的 Storm 应用程序至少可以保证一次处理，而 Trident 仅可以保证一次处理。

有关详细信息，请参阅 apache.org 上的[数据处理保证](https://storm.apache.org/about/guarantees-data-processing.html)。

### IBasicBolt

读取输入元组，发出零个或多个元组，然后在执行方法结束时立即询问输入元组，这种模式很常见， Storm 提供用于自动执行此模式的 [IBasicBolt](https://storm.apache.org/apidocs/backtype/storm/topology/IBasicBolt.html) 接口。

### 联接

在应用程序之间联接两个数据流的方式将有所不同。例如，你可以从多个流将每个元组联接到一个新流，也可以仅联接特定窗口的批量元组。两种方式的联接都可以使用 [fieldsGrouping](http://javadox.com/org.apache.storm/storm-core/0.9.1-incubating/backtype/storm/topology/InputDeclarer.html#fieldsGrouping%28java.lang.String,%20backtype.storm.tuple.Fields%29) 来实现，它是定义元组如何路由到 bolt 的方式。

在以下 Java 实例中，fieldsGrouping 用于将来自组件“1”、“2”和“3”的元组路由至 **MyJoiner** bolt。

```
builder.setBolt("join", new MyJoiner(), parallelism) .fieldsGrouping("1", new Fields("joinfield1", "joinfield2")) .fieldsGrouping("2", new Fields("joinfield1", "joinfield2")) .fieldsGrouping("3", new Fields("joinfield1", "joinfield2"));
```

### 批处理

批处理可以通过若干方式来实现。利用 C# 或 Java 拓扑，可以在发出元组前使用简单计数器对 X 个元组进行批处理，或使用称为“计时周期元组”的内部计时机制每 X 秒发出一批元组。

有关如何使用 C# 组件中的计时周期元组的示例，请参阅 [PartialBoltCount.cs](https://github.com/hdinsight/hdinsight-storm-examples/blob/3b2c960549cac122e8874931df4801f0934fffa7/EventCountExample/EventCountTopology/src/main/java/com/microsoft/hdinsight/storm/examples/PartialCountBolt.java)。

如果你使用的是 Trident，则其基于批量处理元组。

### 缓存

内存缓存通常用作加速处理的机制，因为它在内存中存储常用资产。由于拓扑分布于多个节点，并且每个节点中有多个进程，所以应考虑使用 [fieldsGrouping](http://javadox.com/org.apache.storm/storm-core/0.9.1-incubating/backtype/storm/topology/InputDeclarer.html#fieldsGrouping%28java.lang.String,%20backtype.storm.tuple.Fields%29) 来确保包含用于缓存查询的字段的元组始终路由至同一进程。这可以避免在进程间重复缓存条目。

### 流式处理 top N

当拓扑依赖于计算“top”N 值（比如 Twitter 上的前 5 大趋势）时，你应并行计算 top N 值，然后将这些计算的输出合并到全局值中。为此，可以使用 [fieldsGrouping](http://javadox.com/org.apache.storm/storm-core/0.9.1-incubating/backtype/storm/topology/InputDeclarer.html#fieldsGrouping%28java.lang.String,%20backtype.storm.tuple.Fields%29) 按字段路由至并行 bolt（该项按字段值对数据进行划分），然后路由至全局确定前 N 个值的 bolt。

有关此内容的示例，请参阅 [RollingTopWords](https://github.com/nathanmarz/storm-starter/blob/master/src/jvm/storm/starter/RollingTopWords.java) 示例。

## Storm 使用哪种类型的日志记录？

Storm 使用 Apache Log4j 来记录信息。默认情况下，将记录大量的数据，因此很难通过信息排序。可以让日志记录配置文件包括在 Storm 拓扑中，控制日志记录行为。

有关演示如何配置日志记录的示例拓扑，请参阅适用于 Storm on HDInsight 的[基于 Java 的 WordCount](./hdinsight-storm-develop-java-topology.md) 示例。

## 后续步骤

了解有关使用 HDInsight 中的 Apache Storm 构建实时分析解决方案的详细信息：

* [Storm on HDInsight 入门][gettingstarted]
* [Storm on HDInsight 的示例拓扑](./hdinsight-storm-example-topology.md)

[stormtrident]: https://storm.apache.org/documentation/Trident-API-Overview.html
[samoa]: http://yahooeng.tumblr.com/post/65453012905/introducing-samoa-an-open-source-platform-for-mining
[apachetutorial]: https://storm.apache.org/documentation/Tutorial.html
[gettingstarted]: ./hdinsight-apache-storm-tutorial-get-started-linux.md

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->