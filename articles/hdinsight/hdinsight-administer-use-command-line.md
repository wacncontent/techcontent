---
title: 使用 Azure CLI 管理 Hadoop 群集 | Azure
description: 如何使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集
services: hdinsight
editor: cgronlun
manager: paulettm
authors: mumian
tags: azure-portal
documentationCenter: 

ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/15/2016
wacn.date: 12/30/2016
ms.author: jgao
---

# 使用 Azure CLI 管理 HDInsight 中的 Hadoop 群集

[!INCLUDE [选择器](../../includes/hdinsight-portal-management-selector.md)]

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

了解如何使用 [Azure 命令行接口](../xplat-cli-install.md)管理 Azure HDInsight 中的 Hadoop 群集。Azure CLI 在 Node.js 中实现。可以在支持 Node.js 的任意平台上使用。

本文仅介绍如何将 Azure CLI 与 HDInsight 配合使用。有关如何使用 Azure CLI 的常规指南，请参阅[安装和配置 Azure CLI][azure-command-line-tools]。

[!INCLUDE [use-latest-version](../../includes/hdinsight-use-latest-cli.md)]

## 先决条件

开始阅读本文之前，必须具备以下条件：

- **Azure 订阅**。请参阅[获取 Azure 试用版](https://www.azure.cn/pricing/1rmb-trial/)。
- **Azure CLI** - 有关安装和配置信息，请参阅[安装和配置 Azure CLI](../xplat-cli-install.md)。
- 使用以下命令**连接到 Azure**：

        azure config mode asm
        azure login -e AzureChinaCloud

    > [!NOTE]
    > 如果想用 Azure CLI 管理 Azure 中国的 HDInsight 群集，请安装 Azure CLI 0.9.x，而不是最新的 0.10.x.

要获取帮助，请使用 **-h** 开关。例如：

    azure hdinsight cluster create -h

## 创建群集

请参阅[使用 Azure CLI 在 HDInsight 中创建基于 Windows 的 Hadoop 群集](./hdinsight-hadoop-create-windows-clusters-cli.md)。

## 列出并显示群集详细信息
使用以下命令列出并显示群集详细信息：

    azure hdinsight cluster list
    azure hdinsight cluster show <Cluster Name>

![HDI.CLIListCluster][image-cli-clusterlisting]

## 删除群集
使用以下命令删除群集：

    azure hdinsight cluster delete <Cluster Name>

## 后续步骤
在本文中，已了解如何执行不同的 HDInsight 群集管理任务。要了解更多信息，请参阅下列文章：

* [使用 Azure 经典管理门户管理 HDInsight][hdinsight-admin-portal]
* [使用 Azure PowerShell 管理 HDInsight][hdinsight-admin-powershell]
* [Azure HDInsight 入门][hdinsight-get-started]
* [如何使用 Azure CLI][azure-command-line-tools]

[azure-command-line-tools]: ../xplat-cli-install.md
[azure-create-storageaccount]: ../storage/storage-create-storage-account.md
[azure-purchase-options]: https://www.azure.cn/pricing/overview/
[azure-member-offers]: https://www.azure.cn/pricing/member-offers/
[azure-trial]: https://www.azure.cn/pricing/1rmb-trial/

[hdinsight-admin-portal]: ./hdinsight-administer-use-management-portal-v1.md
[hdinsight-admin-powershell]: ./hdinsight-administer-use-powershell.md
[hdinsight-get-started]: ./hdinsight-hadoop-tutorial-get-started-windows-v1.md

[image-cli-account-download-import]: ./media/hdinsight-administer-use-command-line/HDI.CLIAccountDownloadImport.png
[image-cli-clustercreation]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreation.png
[image-cli-clustercreation-config]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreationConfig.png
[image-cli-clusterlisting]: ./media/hdinsight-administer-use-command-line/HDI.CLIListClusters.png "列出并显示群集"

<!---HONumber=Mooncake_Quality_Review_1202_2016-->