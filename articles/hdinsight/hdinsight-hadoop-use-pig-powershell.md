---
title: 在 HDInsight 中将 Hadoop Pig 与 PowerShell 配合使用 | Azure
description: 了解如何使用 Azure PowerShell 将 Pig 作业提交到 HDInsight 上的 Hadoop 群集。
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal

ms.assetid: 737089c1-b494-4387-9def-7b4dac3be532
ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/19/2017
wacn.date: 03/10/2017
ms.author: larryfr
---

# 使用 PowerShell 运行 Pig 作业
[!INCLUDE [pig-selector](../../includes/hdinsight-selector-use-pig.md)]

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本文档提供了一个示例，演示了使用 Azure PowerShell 向 HDInsight 群集上的 Hadoop 提交 Pig 作业。Pig 允许你使用可为数据转换建模的语言 (Pig Latin) 编写 MapReduce 作业，无需使用映射和化简函数。

> [!NOTE]
本文档未详细描述示例中使用的 Pig Latin 语句的作用。有关此示例中使用的 Pig Latin 的信息，请参阅[将 Pig 与 HDInsight 上的 Hadoop 配合使用](./hdinsight-use-pig.md)。
> 
> 

## <a id="prereq"></a>先决条件
要完成本文中的步骤，需要：

* **一个 Azure HDInsight 群集**

    > [!IMPORTANT]
    Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](./hdinsight-component-versioning.md#hdi-version-32-and-33-nearing-deprecation-date)。

* **配备 Azure PowerShell 的工作站**。

[!INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

## <a id="powershell"></a>使用 PowerShell 运行 Pig 作业
Azure PowerShell 提供 *cmdlet*，可让你在 HDInsight 上远程运行 Pig 作业。从内部来讲，这是通过使用 REST 调用 HDInsight 群集上运行的 [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat)（以前称为 Templeton）实现的。

在远程 HDInsight 群集上运行 Pig 作业时，使用以下 Cmdlet：

* **Login-AzureRmAccount**：对 Azure 订阅进行 Azure PowerShell 身份验证
* **New-AzureRmHDInsightPigJobDefinition**：使用指定的 Pig Latin 语句创建新的*作业定义*
* **Start-AzureRmHDInsightJob**：将作业定义发送到 HDInsight，启动作业，然后返回可用来检查作业状态的*作业*对象
* **Wait-AzureRmHDInsightJob**：使用作业对象来检查作业的状态。它会等待作业完成或超时。
* **Get-AzureRmHDInsightJobOutput**：用于检索作业的输出

以下步骤演示了如何使用这些 Cmdlet 在 HDInsight 群集上运行作业。

1. 使用编辑器将以下代码保存为 **pigjob.ps1**。

    ```
    # Login to your Azure subscription
    # Is there an active Azure subscription?
    $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
    if(-not($sub))
    {
        Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    }

    # Get cluster info
    $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
    $creds=Get-Credential -Message "Enter the login for the cluster"

    #Store the Pig Latin into $QueryString
    $QueryString =  "LOGS = LOAD 'wasb:///example/data/sample.log';" +
    "LEVELS = foreach LOGS generate REGEX_EXTRACT(`$0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;" +
    "FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;" +
    "GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;" +
    "FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;" +
    "RESULT = order FREQUENCIES by COUNT desc;" +
    "DUMP RESULT;"

    #Create a new HDInsight Pig Job definition
    $pigJobDefinition = New-AzureRmHDInsightPigJobDefinition `
        -Query $QueryString `
        -Arguments "-w"

    # Start the Pig job on the HDInsight cluster
    Write-Host "Start the Pig job ..." -ForegroundColor Green
    $pigJob = Start-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $pigJobDefinition `
        -HttpCredential $creds

    # Wait for the Pig job to complete
    Write-Host "Wait for the Pig job to complete ..." -ForegroundColor Green
    Wait-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobId $pigJob.JobId `
        -HttpCredential $creds

    # Display the output of the Pig job.
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzureRmHDInsightJobOutput `
        -ClusterName $clusterName `
        -JobId $pigJob.JobId `
        -HttpCredential $creds
    ```

1. 打开新的 Windows PowerShell 命令提示符。将目录更改为 **pigjob.ps1** 文件的所在位置，然后使用以下命令来运行脚本：

    ```
    .\pigjob.ps1
    ```

    首先会提示登录 Azure 订阅。然后，将要求输入 HDInsight 群集的 HTTPs/Admin 帐户名称和密码。

2. 作业完成时，应返回如下信息：

    ```
    Start the Pig job ...
    Wait for the Pig job to complete ...
    Display the standard output ...
    (TRACE,816)
    (DEBUG,434)
    (INFO,96)
    (WARN,11)
    (ERROR,6)
    (FATAL,2)
    ```

## <a id="troubleshooting"></a>故障排除

如果作业完成时未返回任何信息，可能表示处理期间发生错误。若要查看此作业的错误信息，请将以下命令添加到 **pigjob.ps1** 文件的末尾，保存，然后重新运行该文件。

```
# Print the output of the Pig job.
Write-Host "Display the standard error output ..." -ForegroundColor Green
Get-AzureRmHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $pigJob.JobId `
        -HttpCredential $creds `
        -DisplayOutputType StandardError
```

这样就会返回运行作业时写入服务器的 STDERR 的信息，可用于确定该作业失败的原因。

## <a id="summary"></a>摘要
Azure PowerShell 提供了一种简单方法，可在 HDInsight 群集上运行 Pig 作业、监视作业状态，以及检索输出。

## <a id="nextsteps"></a>后续步骤
有关 HDInsight 中的 Pig 的一般信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](./hdinsight-use-pig.md)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](./hdinsight-use-hive.md)
* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](./hdinsight-use-mapreduce.md)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned and update some code-->