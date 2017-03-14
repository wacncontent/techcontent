---
title: HDInsight 中 Hadoop 群集的可用性 | Azure
description: HDInsight 可部署包含附加头节点的高度可用且可靠的群集。
services: hdinsight
tags: azure-portal
editor: cgronlun
manager: jhubbard
author: mumian
documentationcenter: ''

ms.assetid: ccab792d-60d6-4287-96c2-479e5d0cf358
ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 02/06/2017
wacn.date: 03/10/2017
ms.author: jgao
---

# HDInsight 中基于 Windows 的 Hadoop 群集的可用性和可靠性
> [!IMPORTANT]
本文档中使用的步骤特定于基于 Windows 的 HDInsight 群集。Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](./hdinsight-component-versioning.md#hdi-version-32-and-33-nearing-deprecation-date)。如果使用基于 Linux 的群集，请参阅 [HDInsight 中基于 Linux 的 Hadoop 群集的可用性和可靠性](./hdinsight-high-availability-linux.md)，以了解有关特定于 Linux 的信息。
>
>

HDInsight 允许客户部署各种群集类型，用于不同的数据分析工作负荷。目前提供的群集类型包括用于查询和分析工作负荷的 Hadoop 群集、用于 NoSQL 工作负荷的 HBase 群集，以及用于实时事件处理工作负荷的 Storm 群集。在给定的群集类型中，不同的节点有不同的角色。例如：

* HDInsight 的 Hadoop 群集部署到了两个角色：

    * 头节点（2 个节点）
    * 数据节点（至少 1 个节点）
* HDInsight 的 HBase 群集部署到了三个角色：

    * 头服务器（2 个节点）
    * 区域服务器（至少 1 个节点）
    * 主控/Zookeeper 节点（3 个节点）
* HDInsight 的 Storm 群集部署到了三个角色：

    * Nimbus 节点（2 个节点）
    * Supervisor 服务器（至少 1 个节点）
    * Zookeeper 节点（3 个节点）

Hadoop 群集的标准实现通常具有单个头节点。HDInsight 通过添加辅助头节点/头服务器/Nimbus 节点来提高管理工作负荷所需的服务可用性和可靠性，从而消除了此单点故障。这些头节点/头服务器/Nimbus 节点设计为顺畅管理辅助节点故障，不过在头节点上运行的主服务的任何停机都会导致群集停止工作。

[ZooKeeper](http://zookeeper.apache.org/) 节点 (ZK) 已添加，用于头节点的主导选择，并确保辅助节点和网关 (GW) 知道在活动头节点 (Head Node0) 处于非活动状态的情况下何时故障转移到辅助头节点 (Head Node1)。

![HDInsight Hadoop 实现中高度可靠的头节点示意图。](./media/hdinsight-high-availability/hadoop.high.availability.architecture.diagram.png)

## 检查活动头节点服务的状态
若要确定头节点处于活动状态，并检查该头节点上运行的服务状态，必需通过使用远程桌面协议 (RDP) 连接到 Hadoop 群集。有关 RDP 的说明，请参阅[使用 Azure 门户预览管理 HDInsight 中的 Hadoop 群集](./hdinsight-administer-use-management-portal.md#connect-to-clusters-using-rdp)。连接到该群集后，请双击位于桌面上的“Hadoop 服务可用状态”图标，以便通过相关状态来了解 Namenode、Jobtracker、Templeton、Oozieservice、Metastore 和 Hiveserver2 服务正在哪个头节点上运行，如果是 HDI 3.0，则可通过相关状态来了解 Namenode、Resource Manager、History Server、Templeton、Oozieservice、Metastore 和 Hiveserver2 服务正在哪个头节点上运行。

![](./media/hdinsight-high-availability/Hadoop.Service.Availability.Status.png)  

在屏幕截图中，活动头节点是 *headnode0*。

## 访问辅助头节点上的日志文件
当辅助头节点变为活动头节点时，可以通过浏览 JobTracker UI 来访问辅助头节点上的作业日志，就像是在主活动节点上操作一样。若要访问 JobTracker，必需通过使用 RDP 连接到 Hadoop 群集，如以上部分中所述。通过 RDP 连接到该群集后，双击位于桌面上的“Hadoop 名称节点状态”图标，然后单击“NameNode 日志”，即可转到辅助头节点上的日志目录。

![](./media/hdinsight-high-availability/Hadoop.Head.Node.Log.Files.png)  

## 配置头节点大小
默认情况下，头节点分配作为大型虚拟机 (VM)。管理在群集上运行的大多数 Hadoop 作业只需要这种大小。但是有时，头节点可能需要超大 VM。例如，当群集需要管理大量的小型 Oozie 作业时。

超大 VM 可通过使用 Azure PowerShell cmdlet 或 HDInsight SDK 来配置。

通过使用 Azure PowerShell 创建和预配群集的过程记录在[使用 PowerShell 管理 HDInsight](./hdinsight-administer-use-powershell.md) 中。配置超大头节点需要将 `-HeadNodeVMSize ExtraLarge` 参数添加到此代码中使用的 `New-AzureRmHDInsightcluster` cmdlet。

```
# Create a new HDInsight cluster in Azure PowerShell
# Configured with an ExtraLarge head-node VM
New-AzureRmHDInsightCluster `
            -ResourceGroupName $resourceGroupName `
            -ClusterName $clusterName `
            -Location $location `
            -HeadNodeVMSize ExtraLarge `
            -DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" `
            -DefaultStorageAccountKey $storageAccountKey `
            -DefaultStorageContainerName $containerName  `
            -ClusterSizeInNodes $clusterNodes
```

对于 SDK，情况也是如此。通过使用 SDK 创建和预配群集的过程记录在[使用 HDInsight .NET SDK](./hdinsight-provision-clusters.md) 中。配置超大头节点需要将 `HeadNodeSize = NodeVMSize.ExtraLarge` 参数添加到此代码中使用的 `ClusterCreateParameters()` 方法。

```
# Create a new HDInsight cluster with the HDInsight SDK
# Configured with an ExtraLarge head-node VM
ClusterCreateParameters clusterInfo = new ClusterCreateParameters()
{
    Name = clustername,
    Location = location,
    HeadNodeSize = NodeVMSize.ExtraLarge,
    DefaultStorageAccountName = storageaccountname,
    DefaultStorageAccountKey = storageaccountkey,
    DefaultStorageContainer = containername,
    UserName = username,
    Password = password,
    ClusterSizeInNodes = clustersize
};
```

## 后续步骤
* [Apache ZooKeeper](http://zookeeper.apache.org/)
* [使用 RDP 连接到 HDInsight 群集](./hdinsight-administer-use-management-portal.md#connect-to-clusters-using-rdp)
* [使用 HDInsight .NET SDK](./hdinsight-provision-clusters.md)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned-->