---
title: 使用 Hadoop Tools for Visual Studio 执行 Hive 查询 | Azure
description: 了解如何通过 Visual Studio Hadoop 工具将 Hive 与 HDInsight 中的 Hadoop 配合使用。
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal

ms.assetid: 2b3e672a-1195-4fa5-afb7-b7b73937bfbe
ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 11/28/2016
wacn.date: 03/10/2017
ms.author: larryfr
---

# 使用适用于 Visual Studio 的 HDInsight 工具运行 Hive 查询

[!INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本文介绍如何使用适用于 Visual Studio 的 HDInsight 工具将 Hive 查询提交到 HDInsight 群集。

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-visual-studio-login-guide.md)]

> [!NOTE]
本文档未详细描述示例中使用的 HiveQL 语句的作用。有关此示例中使用的 HiveQL 的详细信息，请参阅[将 Hive 与 HDInsight 上的 Hadoop 配合使用](./hdinsight-use-hive.md)。

## <a id="prereq"></a>先决条件

要完成本文中的步骤，需要：

* Azure HDInsight（HDInsight 上的 Hadoop）群集

    > [!IMPORTANT]
    Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](./hdinsight-component-versioning.md#hdi-version-32-and-33-nearing-deprecation-date)。

* Visual Studio（以下版本之一）：

    包含 [Update 4](https://www.microsoft.com/download/details.aspx?id=44921) 的 Visual Studio 2013 Community/Professional/Premium/Ultimate

    Visual Studio 2015 (Community/Enterprise)

* 用于 Visual Studio 的 HDInsight 工具或用于 Visual Studio 的 Azure Data Lake 工具。有关安装和配置这些工具的信息，请参阅[开始使用适用于 HDInsight 的 Visual Studio Hadoop 工具](./hdinsight-hadoop-visual-studio-tools-get-started.md)。

## <a id="run"></a> 使用 Visual Studio 运行 Hive 查询

1. 打开 Visual Studio，然后选择“新建”>“项目”>“Azure Data Lake”>“HIVE”>“Hive 应用程序”。为此项目提供一个名称。

2. 打开在创建此项目时产生的 **Script.hql** 文件，并在其中粘贴以下 HiveQL 语句：

    ```
    set hive.execution.engine=tez;
    DROP TABLE log4jLogs;
    CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
    STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';
    SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;
    ```

    这些语句可执行以下操作：

    * **DROP TABLE**：删除表和数据文件（如果该表已存在）。

    * **CREATE EXTERNAL TABLE**：在 Hive 中创建新“外部”表。外部表仅在 Hive 中存储表定义；数据会保留在原始位置。

        > [!NOTE]
        如果希望以外部源更新基础数据（例如自动化数据上载过程），或以其他 MapReduce 操作更新基础数据，但希望 Hive 查询始终使用最新数据，则必须使用外部表。
        > <p> 
        > 删除外部表**不会**删除数据，只会删除表定义。

    * **ROW FORMAT**：告知 Hive 如何设置数据的格式。在此情况下，每个日志中的字段以空格分隔。

    * **STORED AS TEXTFILE LOCATION**：让 Hive 知道数据的存储位置（example/data 目录），并且数据已存储为文本。

    * **SELECT**：选择列 **t4** 包含值 **[ERROR]** 的所有行计数。这应会返回值 **3**，因为有三个行包含此值。

    * **INPUT\_\_FILE\_\_NAME LIKE '%.log'** - 告诉 Hive，我们只应返回以 .log 结尾的文件中的数据。此项将搜索限定为包含数据的 sample.log 文件，而不返回与所定义架构不符的其他示例数据文件中的数据。

3. 从工具栏中，选择要用于此查询的“HDInsight 群集”，然后选择“提交到 WebHCat”，以便使用 WebHCat 以 Hive 作业形式运行语句。如果 HiveServer2 在群集版本上可用，则也可以使用“通过 HiveServer2 执行”按钮提交作业。“Hive 作业摘要”会出现并显示有关正在运行的作业的信息。在“作业状态”更改为“已完成”之前，使用“刷新”链接刷新作业信息。

4. 使用“作业输出”链接查看此作业的输出。它应该会显示 `[ERROR] 3`，这是 SELECT 语句返回的值。

5. 也可以运行 Hive 查询，无需创建项目。使用“服务器资源管理器”，展开“Azure”>“HDInsight”，右键单击 HDInsight 服务器，然后选择“编写 Hive 查询”。

6. 在出现的 **temp.hql** 文档中，添加以下 HiveQL 语句：

    ```
    set hive.execution.engine=tez;
    CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
    INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log';
    ```

    这些语句可执行以下操作：

    * **CREATE TABLE IF NOT EXISTS**：创建表（如果该表不存在）。由于未使用 **EXTERNAL** 关键字，因此这是一个内部表，它存储在 Hive 数据仓库中并完全受 Hive 的管理。

        > [!NOTE]
        与**外部**表不同，删除内部表会同时删除基础数据。

    * **STORED AS ORC**：以优化行纵栏表 (ORC) 格式存储数据。这是高度优化且有效的 Hive 数据存储格式。

    * **INSERT OVERWRITE ...SELECT**：从包含 **[ERROR]** 的 **log4jLogs** 表中选择行，然后将数据插入 **errorLogs** 表中。

7. 从工具栏中，选择“提交”以运行该作业。使用“作业状态”确定作业是否已成功完成。

8. 要确认作业已完成且已创建新表，请使用“服务器资源管理器”，展开“Azure”>“HDInsight”> HDInsight 群集 >“Hive 数据库”>“默认值”。你应该会看到 **errorLogs** 表和 **log4jLogs** 表。

## <a id="summary"></a>摘要

适用于 Visual Studio 的 HDInsight 工具提供了一种简单的方法，可在 HDInsight 群集上运行 Hive 查询，监视作业状态，以及检索输出。

## <a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 Hive 的一般信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](./hdinsight-use-hive.md)

有关 HDInsight 上 Hadoop 的其他使用方法的信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](./hdinsight-use-pig.md)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](./hdinsight-use-mapreduce.md)

有关适用于 Visual Studio 的 HDInsight 工具的详细信息：

* [适用于 Visual Studio 的 HDInsight 工具入门](./hdinsight-hadoop-visual-studio-tools-get-started.md)

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx

[azure-purchase-options]: https://www.azure.cn/pricing/overview/
[azure-member-offers]: https://www.azure.cn/pricing/member-offers/
[azure-trial]: https://www.azure.cn/pricing/1rmb-trial/

[apache-tez]: http://tez.apache.org
[apache-hive]: http://hive.apache.org/
[apache-log4j]: http://zh.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: ./hdinsight-connect-excel-power-query.md

[hdinsight-use-oozie]: ./hdinsight-use-oozie.md
[hdinsight-analyze-flight-data]: ./hdinsight-analyze-flight-delay-data.md

[hdinsight-storage]: ./hdinsight-hadoop-use-blob-storage.md

[hdinsight-provision]: ./hdinsight-provision-clusters.md
[hdinsight-submit-jobs]: ./hdinsight-submit-hadoop-jobs-programmatically.md
[hdinsight-upload-data]: ./hdinsight-upload-data.md
[hdinsight-get-started]: ./hdinsight-hadoop-linux-tutorial-get-started.md

[powershell-here-strings]: http://technet.microsoft.com/zh-cn/library/ee692792.aspx

[image-hdi-hive-powershell]: ./media/hdinsight-use-hive/HDI.HIVE.PowerShell.png
[img-hdi-hive-powershell-output]: ./media/hdinsight-use-hive/HDI.Hive.PowerShell.Output.png
[image-hdi-hive-architecture]: ./media/hdinsight-use-hive/HDI.Hive.Architecture.png

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned-->