---
title: 通过 PowerShell 使用 Mahout HDInsight 生成推荐 | Azure
description: 了解如何使用 Apache Mahout 机器学习库通过客户端上运行的 PowerShell 脚本中的 HDInsight (Hadoop) 生成电影推荐。
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal

ms.assetid: 07b57208-32aa-4e59-900a-6c934fa1b7a7
ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/13/2017
wacn.date: 01/25/2017
ms.author: larryfr
---

# 将 Apache Mahout 与 HDInsight (PowerShell) 中的 Hadoop 配合使用生成电影推荐
[!INCLUDE [mahout-selector](../../includes/hdinsight-selector-mahout.md)]

了解如何使用 [Apache Mahout](http://mahout.apache.org) 机器学习库通过 Azure HDInsight 生成电影推荐。本文档介绍如何使用 Azure PowerShell 远程运行 Mahout。

Mahout 是适用于 Apache Hadoop 的[计算机学习][ml]库。Mahout 包含用于处理数据的算法，例如筛选、分类和群集。在本文中，用户使用推荐引擎根据好友看过的电影生成电影推荐。

## 先决条件

* 基于 Linux 的 HDInsight 群集。有关创建该群集的信息，请参阅[开始在 HDInsight 中使用基于 Linux 的 Hadoop][getstarted]。

    [!INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

    > [!IMPORTANT]
    Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](./hdinsight-component-versioning.md#hdi-version-32-and-33-nearing-deprecation-date)。

* **配备 Azure PowerShell 的工作站**。

    > [!IMPORTANT]
    Azure PowerShell 对于使用 Azure Service Manager 管理 HDInsight 资源的支持已**弃用**，将于 2017 年 1 月 1 日删除。本文档中的步骤使用的是与 Azure Resource Manager 兼容的新 HDInsight cmdlet。
    ><p>
    > 请按照 [Install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)（安装和配置 Azure PowerShell）中的步骤安装最新版本的 Azure PowerShell。如果你的脚本需要修改才能使用与 Azure Resource Manager 兼容的新 cmdlet，请参阅[迁移到适用于 HDInsight 群集的基于 Azure Resource Manager 的开发工具](./hdinsight-hadoop-development-using-azure-resource-manager.md)，了解详细信息。

## <a name="recommendations"></a>使用 Azure PowerShell 生成推荐

> [!NOTE]
尽管在本部分中使用的作业使用 Azure PowerShell 执行，但是，随 Mahout 一起提供的很多类当前不适用于 Azure PowerShell，必须使用 Hadoop 命令行来运行这些类。有关不适用于 Azure PowerShell 的类的列表，请参阅[故障排除](#troubleshooting)部分。
><p>
><p>有关使用 SSH 连接到 HDInsight 和直接在群集上运行 Mahout 示例的示例，请参阅[使用 Mahout 和 HDInsight (SSH) 生成电影推荐](./hdinsight-hadoop-mahout-linux-mac.md)。

Mahout 提供的功能之一是推荐引擎。此引擎接受 `userID`、`itemId` 和 `prefValue` 格式（此项的用户偏好）的数据。然后，Mahout 将执行共现分析，以确定：*偏好某个项的用户也偏好其他类似项*。随后，Mahout 确定拥有类似项偏好的用户，这些偏好可用于推荐。

下面是使用电影的极其简单的示例：

* **共现**：Joe、Alice 和 Bob 都喜欢电影《星球大战》、《帝国反击战》和《绝地归来》。Mahout 可确定喜欢以上电影之一的用户也喜欢其他两部。

* **共现**：Bob 和 Alice 还喜欢电影《幽灵的威胁》、《克隆人的进攻》和《西斯的复仇》。Mahout 可确定喜欢前面三部电影的用户也喜欢这三部电影。

* **类似性推荐**：由于 Joe 喜欢前三部电影，Mahout 会查看具有类似偏好的其他人喜欢、但 Joe 还未观看过（喜欢/评价）的电影。在本例中，Mahout 推荐《幽灵的威胁》、《克隆人的进攻》和《西斯的复仇》。

### 了解数据

为方便起见，[GroupLens 研究][movielens]以兼容 Mahout 的格式提供电影的评价数据。此数据在 `/HdiSamples//HdiSamples/MahoutMovieData` 中你的群集的默认存储中可用。

存在两个文件，`moviedb.txt`（有关影片的信息）和 `user-ratings.txt`。user-ratings.txt 文件用于分析过程中，而 moviedb.txt 用于在显示分析结果时，提供用户友好的文本信息。

user-ratings.txt 中包含的数据具有 `userID`、`movieID`、`userRating` 和 `timestamp` 结构，它将告诉我们每个用户对电影评级的情况。下面是数据的示例：

```
196    242    3    881250949
186    302    3    891717742
22    377    1    878887116
244    51    2    880606923
166    346    1    886397596
```

### 运行作业

使用以下 Windows PowerShell 脚本来运行作业，以将 Mahout 推荐引擎用于电影数据：

> [!NOTE]
此文件将提示你输入用于连接到 HDInsight 群集和运行作业的信息。完成作业和下载 output.txt 文件可能需要几分钟时间。

```
# Script should stop on failures
$ErrorActionPreference = "Stop"

# Login to your Azure subscription
# Is there an active Azure subscription?
$sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Add-AzureRmAccount -EnvironmentName AzureChinaCloud
}

# Get cluster info
$clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
$creds=Get-Credential -Message "Enter the login for the cluster (the default name is usually 'admin')"

#Get the cluster info so we can get the resource group, storage, etc.
$clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
$resourceGroup = $clusterInfo.ResourceGroup
$storageAccountName = $clusterInfo.DefaultStorageAccount.split('.')[0]
$container = $clusterInfo.DefaultStorageContainer
$storageAccountKey = (Get-AzureRmStorageAccountKey `
    -Name $storageAccountName `
-ResourceGroupName $resourceGroup)[0].Value

#Create a storage context and upload the file
$context = New-AzureStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey $storageAccountKey

#Use Hive to figure out the path to the mahout examples
#Because the file name/path has a version number in it that changes
$queryString = "!ls /usr/hdp/current/mahout-client"
$hiveJobDefinition = New-AzureRmHDInsightHiveJobDefinition -Query $queryString
$hiveJob=Start-AzureRmHDInsightJob -ClusterName $clusterName -JobDefinition $hiveJobDefinition -HttpCredential $creds
$dummy = wait-azurermhdinsightjob -ClusterName $clusterName -JobId $hiveJob.JobId -HttpCredential $creds
#Get the files returned from Hive
$files=get-azurermhdinsightjoboutput -clustername $clusterName -JobId $hiveJob.JobId -DefaultContainer $container -DefaultStorageAccountName $storageAccountName -DefaultStorageAccountKey $storageAccountKey -HttpCredential $creds
#Find the file that starts with mahout-examples and ends in job.jar
$jarFile = $files | select-string "mahout-examples.+job\.jar" | % {$_.Matches.Value}
#Add the full path
$jarFile = "file:///usr/hdp/current/mahout-client/$jarFile"

# The arguments for the mahout job
# * input - the path to the data uploaded to HDInsight
# * output - the path to store output data
# * tempDir - the directory for temp files
$jobArguments = "-s", "SIMILARITY_COOCCURRENCE", `
                "--input", "/HdiSamples/HdiSamples/MahoutMovieData/user-ratings.txt",
                "--output", "/example/out",
                "--tempDir", "/example/temp"

# Create the job definition
$jobDefinition = New-AzureRmHDInsightMapReduceJobDefinition `
    -JarFile $jarFile `
    -ClassName "org.apache.mahout.cf.taste.hadoop.item.RecommenderJob" `
    -Arguments $jobArguments

# Start the job
$job = Start-AzureRmHDInsightJob `
    -ClusterName $clusterName `
    -JobDefinition $jobDefinition `
    -HttpCredential $creds

# Wait on the job to complete
Write-Host "Wait for the job to complete ..." -ForegroundColor Green
Wait-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds

# Write out any error information
Write-Host "STDERR"
Get-AzureRmHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -DefaultContainer $container `
        -DefaultStorageAccountName $storageAccountName `
        -DefaultStorageAccountKey $storageAccountKey `
        -HttpCredential $creds `
        -DisplayOutputType StandardError

# Download the output
Get-AzureStorageBlobContent `
        -Blob example/out/part-r-00000 `
        -Container $container `
        -Destination output.txt `
        -Context $context
#Download movie and user files for use in displaying results
Get-AzureStorageBlobContent -blob "HdiSamples/HdiSamples/MahoutMovieData/moviedb.txt" `
        -Container $container `
        -Destination moviedb.txt `
        -Context $context
Get-AzureStorageBlobContent -blob "HdiSamples/HdiSamples/MahoutMovieData/user-ratings.txt" `
        -Container $container `
        -Destination user-ratings.txt `
        -Context $context
```

> [!NOTE]
Mahout 作业不删除在处理作业时创建的临时数据。在示例作业中指定 `--tempDir` 参数，以将临时文件隔离到特定目录中。

Mahout 作业不会将输出返回到 STDOUT。而是会将其作为 **part-r-00000** 存储在指定的输出目录中。该脚本将此文件下载到你工作站上的当前目录中的 **output.txt** 中。

下面是此文件内容的示例：

```
1    [234:5.0,347:5.0,237:5.0,47:5.0,282:5.0,275:5.0,88:5.0,515:5.0,514:5.0,121:5.0]
2    [282:5.0,210:5.0,237:5.0,234:5.0,347:5.0,121:5.0,258:5.0,515:5.0,462:5.0,79:5.0]
3    [284:5.0,285:4.828125,508:4.7543354,845:4.75,319:4.705128,124:4.7045455,150:4.6938777,311:4.6769233,248:4.65625,272:4.649266]
4    [690:5.0,12:5.0,234:5.0,275:5.0,121:5.0,255:5.0,237:5.0,895:5.0,282:5.0,117:5.0]
```

第一列是 `userID`。“[”和“]”中包含的值为 `movieId`:`recommendationScore`。

该脚本还会下载 `moviedb.txt` 和 `user-ratings.txt` 文件，需要这些文件格式化输出使其更具可读性。

### 查看输出

生成的输出也许可用于应用程序中，但其可读性欠佳。可以使用服务器中的 `moviedb.txt` 将 `movieId` 解析为电影名称。使用以下 PowerShell 脚本显示包含影片名称的推荐：

```
<#
.SYNOPSIS
    Displays recommendations for movies.
.DESCRIPTION
    Displays recommendations generated by Mahout
    with HDInsight example in a human readable format.
.EXAMPLE
    .\Show-Recommendation -userId 4
        -userDataFile "user-ratings.txt"
        -movieFile "moviedb.txt"
        -recommendationFile "output.txt"
#>

[CmdletBinding(SupportsShouldProcess = $true)]
param(
    #The user ID
    [Parameter(Mandatory = $true)]
    [String]$userId,

    [Parameter(Mandatory = $true)]
    [String]$userDataFile,

    [Parameter(Mandatory = $true)]
    [String]$movieFile,

    [Parameter(Mandatory = $true)]
    [String]$recommendationFile
)
# Read movie ID & description into hash table
Write-Host "Reading movies descriptions" -ForegroundColor Green
$movieById = @{}
foreach($line in Get-Content $movieFile)
{
    $tokens = $line.Split("|")
    $movieById[$tokens[0]] = $tokens[1]
}
# Load movies user has already seen (rated)
# into a hash table
Write-Host "Reading rated movies" -ForegroundColor Green
$ratedMovieIds = @{}
foreach($line in Get-Content $userDataFile)
{
    $tokens = $line.Split("`t")
    if($tokens[0] -eq $userId)
    {
        # Resolve the ID to the movie name
        $ratedMovieIds[$movieById[$tokens[1]]] = $tokens[2]
    }
}
# Read recommendations generated by Mahout
Write-Host "Reading recommendations" -ForegroundColor Green
$recommendations = @{}
foreach($line in get-content $recommendationFile)
{
    $tokens = $line.Split("`t")
    if($tokens[0] -eq $userId)
    {
        #Trim leading/treailing [] and split at ,
        $movieIdAndScores = $tokens[1].TrimStart("[").TrimEnd("]").Split(",")
        foreach($movieIdAndScore in $movieIdAndScores)
        {
            #Split at : and store title and score in a hash table
            $idAndScore = $movieIdAndScore.Split(":")
            $recommendations[$movieById[$idAndScore[0]]] = $idAndScore[1]
        }
        break
    }
}

Write-Host "Rated movies" -ForegroundColor Green
Write-Host "---------------------------" -ForegroundColor Green
$ratedFormat = @{Expression={$_.Name};Label="Movie";Width=40}, `
                @{Expression={$_.Value};Label="Rating"}
$ratedMovieIds | format-table $ratedFormat
Write-Host "---------------------------" -ForegroundColor Green

write-host "Recommended movies" -ForegroundColor Green
Write-Host "---------------------------" -ForegroundColor Green
$recommendationFormat = @{Expression={$_.Name};Label="Movie";Width=40}, `
                        @{Expression={$_.Value};Label="Score"}
$recommendations | format-table $recommendationFormat
```

下面是运行脚本的示例：

```
PS C:\> show-recommendation.ps1 -userId 4 -userDataFile .\user-ratings.txt -movieFile .\moviedb.txt -recommendationFile .\output.txt
```

输出应如下所示：

```
Reading movies descriptions
Reading rated movies
Reading recommendations
Rated movies
---------------------------
Movie                                    Rating
-----                                    ------
Devil's Own, The (1997)                  1
Alien: Resurrection (1997)               3
187 (1997)                               2
(lines ommitted)

---------------------------
Recommended movies
---------------------------

Movie                                    Score
-----                                    -----
Good Will Hunting (1997)                 4.6504064
Swingers (1996)                          4.6862745
Wings of the Dove, The (1997)            4.6666665
People vs. Larry Flynt, The (1996)       4.834559
Everyone Says I Love You (1996)          4.707071
Secrets & Lies (1996)                    4.818182
That Thing You Do! (1996)                4.75
Grosse Pointe Blank (1997)               4.8235292
Donnie Brasco (1997)                     4.6792455
Lone Star (1996)                         4.7099237
```

## <a name="troubleshooting"></a>故障排除

### 无法覆盖文件

Mahout 作业不清理在处理期间创建的临时文件。此外，作业将不会覆盖现有的输出文件。

若要避免运行 Mahout 作业时出错，请在每次运行作业之前删除临时文件和输出文件，或者使用唯一的临时目录名称和输出目录名称。使用以下 PowerShell 脚本删除本文档前面的脚本创建的文件：

```
# Script should stop on failures
$ErrorActionPreference = "Stop"

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

#Get the cluster info so we can get the resource group, storage, etc.
$clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
$resourceGroup = $clusterInfo.ResourceGroup
$storageAccountName = $clusterInfo.DefaultStorageAccount.split('.')[0]
$container = $clusterInfo.DefaultStorageContainer
$storageAccountKey = (Get-AzureRmStorageAccountKey `
    -Name $storageAccountName `
-ResourceGroupName $resourceGroup)[0].Value

#Create a storage context and upload the file
$context = New-AzureStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey $storageAccountKey

#Azure PowerShell can't delete blobs using wildcard, 
#so have to get a list and delete one at a time
# Start with the output
$blobs = Get-AzureStorageBlob -Container $container -Context $context -Prefix "example/out"
foreach($blob in $blobs)
{
    Remove-AzureStorageBlob -Blob $blob.Name -Container $container -context $context
}
# Next the temp files
$blobs = Get-AzureStorageBlob -Container $container -Context $context -Prefix "example/temp"
foreach($blob in $blobs)
{
    Remove-AzureStorageBlob -Blob $blob.Name -Container $container -context $context
}
```

### <a name="nopowershell"></a>不适用于 Azure PowerShell 的类

Mahout 作业如果使用以下类，则从 Windows PowerShell 中使用这些类时，将返回各种错误消息：

* org.apache.mahout.utils.clustering.ClusterDumper
* org.apache.mahout.utils.SequenceFileDumper
* org.apache.mahout.utils.vectors.lucene.Driver
* org.apache.mahout.utils.vectors.arff.Driver
* org.apache.mahout.text.WikipediaToSequenceFile
* org.apache.mahout.clustering.streaming.tools.ResplitSequenceFiles
* org.apache.mahout.clustering.streaming.tools.ClusterQualitySummarizer
* org.apache.mahout.classifier.sgd.TrainLogistic
* org.apache.mahout.classifier.sgd.RunLogistic
* org.apache.mahout.classifier.sgd.TrainAdaptiveLogistic
* org.apache.mahout.classifier.sgd.ValidateAdaptiveLogistic
* org.apache.mahout.classifier.sgd.RunAdaptiveLogistic
* org.apache.mahout.classifier.sequencelearning.hmm.BaumWelchTrainer
* org.apache.mahout.classifier.sequencelearning.hmm.ViterbiEvaluator
* org.apache.mahout.classifier.sequencelearning.hmm.RandomSequenceGenerator
* org.apache.mahout.classifier.df.tools.Describe

若要运行使用这些类的作业，请使用 SSH 连接到 HDInsight 群集，然后从命令行运行这些作业。有关使用 SSH 运行 Mahout 作业的示例，请参阅[使用 Mahout 和 HDInsight (SSH) 生成电影推荐](./hdinsight-hadoop-mahout-linux-mac.md)。

## 后续步骤

既已学习如何使用 Mahout，可探索在 HDInsight 上处理数据的其他方式：

* [Hive 和 HDInsight](./hdinsight-use-hive.md)
* [Pig 和 HDInsight](./hdinsight-use-pig.md)
* [MapReduce 和 HDInsight](./hdinsight-use-mapreduce.md)

[build]: http://mahout.apache.org/developers/buildingmahout.html
[aps]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs
[movielens]: http://grouplens.org/datasets/movielens/
[100k]: http://files.grouplens.org/datasets/movielens/ml-100k.zip
[getstarted]: ./hdinsight-hadoop-linux-tutorial-get-started.md
[upload]: ./hdinsight-upload-data.md
[ml]: http://en.wikipedia.org/wiki/Machine_learning
[forest]: http://en.wikipedia.org/wiki/Random_forest
[management]: https://manage.windowsazure.cn/
[enableremote]: ./media/hdinsight-mahout/enableremote.png
[connect]: ./media/hdinsight-mahout/connect.png
[hadoopcli]: ./media/hdinsight-mahout/hadoopcli.png
[tools]: https://github.com/Blackmist/hdinsight-tools

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->