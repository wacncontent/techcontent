---
title: 使用 .NET SDK 管理 HDInsight 中的 Hadoop 群集 | Azure
description: 了解如何使用 HDInsight .NET SDK 针对 HDInsight 中的 Hadoop 群集执行管理任务。
services: hdinsight
editor: cgronlun
manager: paulettm
tags: azure-portal
authors: mumian
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

# 使用 .NET SDK 管理 HDInsight 中的 Hadoop 群集

[!INCLUDE [选择器](../../includes/hdinsight-portal-management-selector.md)]

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

了解如何使用 [HDInsight.NET SDK](https://msdn.microsoft.com/zh-cn/library/mt271028.aspx) 管理 HDInsight 群集。

**先决条件**

在开始本文前，你必须具有以下项：

- **一个 Azure 订阅**。请参阅[获取 Azure 试用版](https://www.azure.cn/pricing/1rmb-trial/)。

##连接到 Azure HDInsight

需要以下 Nuget 包：

	Install-Package Microsoft.WindowsAzure.Management.HDInsight

以下代码示例演示如何先连接到 Azure，然后管理 Azure 订阅下面的 HDInsight 群集。

	using System;
	using Microsoft.WindowsAzure.Management.HDInsight;
	using System.Security.Cryptography.X509Certificates;

	namespace HDInsightManagement
	{
		class Program
		{
			private static IHDInsightClient _hdinsightClient;
			private static String SubscriptionId = "<Your Azure Subscription ID>";
			private static Uri baseUri = new Uri("https://management.core.chinacloudapi.cn");
			private static X509Certificate2 cert = new X509Certificate2("c:/path/to/cert.cer");

			static void Main(string[] args)
			{
				_hdinsightClient = HDInsightClient.Connect(new HDInsightCertificateCredential(new Guid(SubscriptionId), cert, baseUri));

				// insert code here

				System.Console.WriteLine("Press ENTER to continue");
				System.Console.ReadLine();
			}
		}
	}

若要使用此代码段，需要创建证书文件并将其上传到 Azure。有关详细信息，请参阅[上传 Azure Management API 管理证书](../azure-api-management-certs.md)。

##列出群集

以下代码片段列出了群集和一些属性：

    var results = _hdinsightClient.ListClusters();
    foreach (var name in results) {
        Console.WriteLine("Cluster Name: " + name.Name);
        Console.WriteLine("\t Cluster type: " + name.ClusterType);
        Console.WriteLine("\t Cluster location: " + name.Location);
        Console.WriteLine("\t Cluster version: " + name.Version);
    }

##删除群集

使用以下代码片段以同步或异步方式删除群集：

    _hdinsightClient.DeleteCluster("<Cluster Name>");
    _hdinsightClient.DeleteClusterAsync("<Cluster Name>");

## <a name="grant/revoke-access"></a>授予/撤消访问权限

HDInsight 群集提供以下 HTTP Web 服务（所有这些服务都有 REST 样式的终结点）：

- ODBC
- JDBC
- Oozie
- Templeton

默认情况下，将授权这些服务进行访问。你可以撤消/授予访问权限。若要撤消：

	var cluster = _hdinsightClient.GetCluster("<Cluster Name>");
    _hdinsightClient.DisableHttp(cluster.Name, cluster.Location);

若要授予：

	_hdinsightClient.EnableHttp(cluster.Name, cluster.Location, "admin","<password>");

>[!NOTE] 通过授予/撤消访问权限，你将重置群集用户名和密码。

也可以通过经典管理门户完成此操作。请参阅[使用 Azure 经典管理门户管理 HDInsight][hdinsight-admin-portal]。

##更新 HTTP 用户凭据

这与[授予/撤消 HTTP 访问权限](#grant/revoke-access)是同一过程。如果已授予群集 HTTP 访问权限，必须先撤消该访问权限。然后再使用新的 HTTP 用户凭据授予访问权限。

##查找默认存储帐户

以下代码片段演示如何获取群集的默认存储帐户名称和默认存储帐户密钥。

	var cluster = _hdinsightClient.GetCluster("<Cluster Name>");
	var storageKey = cluster.DefaultStorageAccount.Key;

##提交作业

**提交 Hive 作业**

请参阅[使用 .NET SDK 运行 Hive 查询）](./hdinsight-hadoop-use-hive-dotnet-sdk.md)。

**提交 Pig 作业**

请参阅[使用 .NET SDK 运行 Pig 作业](./hdinsight-hadoop-use-pig-dotnet-sdk-v1.md)。

**提交 Sqoop 作业**

请参阅[将 Sqoop 与 HDInsight 配合使用](./hdinsight-hadoop-use-sqoop-dotnet-sdk.md)。

**提交 Oozie 作业**

请参阅[在 HDInsight 中将 Oozie 与 Hadoop 配合使用以定义和运行工作流](./hdinsight-use-oozie.md)。

##将数据上传到 Azure Blob 存储
请参阅[将数据上传到 HDInsight][hdinsight-upload-data]。

## 另请参阅
* [HDInsight .NET SDK 参考文档](https://msdn.microsoft.com/zh-cn/library/mt271028.aspx)
* [使用 Azure 经典管理门户管理 HDInsight][hdinsight-admin-portal]
* [使用命令行接口管理 HDInsight][hdinsight-admin-cli]
* [创建 HDInsight 群集][hdinsight-provision]
* [将数据上传到 HDInsight][hdinsight-upload-data]
* [Azure HDInsight 入门][hdinsight-get-started]

[azure-purchase-options]: https://www.azure.cn/pricing/overview/
[azure-member-offers]: https://www.azure.cn/pricing/member-offers/
[azure-trial]: https://www.azure.cn/pricing/1rmb-trial/

[hdinsight-get-started]: ./hdinsight-hadoop-tutorial-get-started-windows-v1.md
[hdinsight-provision]: ./hdinsight-provision-clusters-v1.md
[hdinsight-provision-custom-options]: ./hdinsight-provision-clusters-v1.md#configuration
[hdinsight-submit-jobs]: ./hdinsight-submit-hadoop-jobs-programmatically.md

[hdinsight-admin-cli]: ./hdinsight-administer-use-command-line.md
[hdinsight-admin-portal]: ./hdinsight-administer-use-management-portal-v1.md
[hdinsight-storage]: ./hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-hive]: ./hdinsight-use-hive.md
[hdinsight-use-mapreduce]: ./hdinsight-use-mapreduce.md
[hdinsight-upload-data]: ./hdinsight-upload-data.md
[hdinsight-flight]: ./hdinsight-analyze-flight-delay-data.md

<!---HONumber=Mooncake_Quality_Review_1215_2016-->