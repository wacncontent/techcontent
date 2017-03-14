---
title: Azure Batch 服务基础知识 | Azure
description: 了解如何使用 Azure Batch 服务执行大规模并发工作负荷与 HPC 工作负荷
services: batch
documentationcenter: ''
author: tamram
manager: timlt
editor: ''

ms.assetid: 93e37d44-7585-495e-8491-312ed584ab79
ms.service: batch
ms.workload: big-compute
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 12/19/2016
wacn.date: 01/24/2017
ms.author: tamram
---

# Azure Batch 基础知识
Azure Batch 是一种平台服务，可在云中有效运行大规模并行的高性能计算 (HPC) 应用程序。Azure Batch 可计划要在托管的虚拟机集合上运行的计算密集型工作，并且可以缩放计算资源以符合作业的需求。

使用 Azure Batch 时，可轻松定义用于大规模并行执行应用程序的 Azure 计算资源。无需手动创建、配置和管理 HPC 群集、各个虚拟机、虚拟网络或复杂的作业和任务计划基础结构。Azure Batch 将自动执行或简化这些任务。

## Batch 的用例
Batch 是一种托管的 Azure 服务，可用于实现批处理或批量计算 -- 运行大量类似任务以获取所需的结果。定期处理、转换和分析大量数据的组织最常使用批量计算。

Batch 很适合处理本质并行（也称为“超简单并行”）的应用程序和工作负荷。内在并行的工作负荷可轻松拆分成多个任务，然后在多台计算机上同时执行。

![并行任务][1]

通常使用此技术处理的工作负荷有以下示例：

- 金融风险建模
- 气候和水文数据分析
- 图像渲染、分析和处理
- 媒体编码和转码
- 基因序列分析
- 工程压力分析
- 软件测试

用户还可以使用 Batch 执行并行计算（最后加上归纳步骤），以及其他更复杂的 HPC 工作负荷，例如[消息传递接口 (MPI)](./batch-mpi.md) 应用程序。

有关 Batch 与 Azure 中其他 HPC 解决方案选项的比较，请参阅 [Batch 和 HPC 解决方案](./batch-hpc-solutions.md)。

[!INCLUDE [batch-pricing-include](../../includes/batch-pricing-include.md)]

## 使用 Batch 进行开发
使用 Azure Batch 处理并行工作负荷通常是使用 [Batch API](#batch-development-apis) 之一以编程方式实现的。客户端应用程序或服务可使用 Batch API 与 Batch 服务通信。Batch API 允许用户创建和管理计算节点（虚拟机或云服务）池。随后可将作业和任务计划为在这些节点上运行。

可以为组织高效处理大量工作负荷，或提供服务前端给客户，让他们可以在一个、数百个甚至数千个节点上，按需要或按计划运行作业和任务。

> [!TIP]
> 若要深入了解 Batch API 所提供的功能，请参阅 [Batch feature overview for developers](./batch-api-basics.md)（面向开发人员的 Batch 功能概述）。
> 
> 

### 需要的 Azure 帐户
开发 Batch 解决方案时，用户在 Azure 中使用以下帐户。

- **Azure 帐户和订阅** - 如果还没有 Azure 订阅，可注册 [Azure 帐户][free_account]。创建帐户时，系统为用户创建默认订阅。
- **批处理帐户** - Azure Batch 资源（包括池、计算节点、作业和任务）都与 Azure 批处理帐户关联。如果应用程序对 Batch 服务发出请求，它将使用 Azure 批处理帐户名称、帐户 URL 和访问密钥对请求进行身份验证。可以在 Azure 门户预览中[创建 Batch 帐户](./batch-account-create-portal.md)。
- **存储帐户** - Batch 提供的内置支持允许处理 [Azure 存储][azure_storage]中的文件。几乎每个 Batch 方案都使用 Azure Blob 存储暂存任务所运行的程序及其处理的数据，以及存储任务生成的输出数据。若要创建存储帐户，请参阅[关于 Azure 存储帐户](../storage/storage-create-storage-account.md)。

### Batch 开发 API  <a name="batch-development-apis"></a>
应用程序和服务可发出直接的 REST API 调用，或使用以下一个或多个客户端库，运行和管理你的 Azure Batch 工作负荷。

| API | API 参考 | 下载 | 教程 | 代码示例 |
| --- | --- | --- | --- | --- |
| **Batch REST** |[MSDN][batch_rest] |N/A |- |- |
| **Batch .NET** |[docs.microsoft.com][api_net] |[NuGet ][api_net_nuget] |[Tutorial](./batch-dotnet-get-started.md) |[GitHub][api_sample_net] |
| **Batch Python** |[readthedocs.io][api_python] |[PyPI][api_python_pypi] |[Tutorial](./batch-python-tutorial.md)|[GitHub][api_sample_python] |
| **Batch Node.js** |[github.io][api_nodejs] |[npm][api_nodejs_npm] |- |- |
| **Batch Java** (preview) |[github.io][api_java] |[Maven][api_java_jar] |- |[GitHub][api_sample_java] |

### Batch 命令行工具

也可通过命令行工具使用开发 API 提供的功能：

- [Batch PowerShell cmdlet][batch_ps]：Azure PowerShell 模块中的 Azure Batch cmdlet 允许用户使用 PowerShell 管理 Batch 资源。
- [Azure CLI](../xplat-cli-install.md)：Azure 命令行接口 (Azure CLI) 是一个跨平台工具集，提供用来与许多 Azure 服务（包括 Batch）交互的 shell 命令。

### Batch 资源管理

通过用于 Batch 的 Azure资源管理器API，可以编程方式访问批处理帐户。使用这些 API，可以编程方式管理批处理帐户、配额和应用程序包。

| API | API 参考 | 下载 | 教程 | 代码示例 |
| --- | --- | --- | --- | --- |
| **Batch 资源管理器 REST** | [docs.microsoft.com][api_rest_mgmt] | N/A | - | [GitHub](https://github.com/Azure-Samples/batch-dotnet-manage-batch-accounts) | 
| **Batch 资源管理器 .NET** | [docs.microsoft.com][api_net_mgmt] | [NuGet ][api_net_mgmt_nuget] | [教程](./batch-management-dotnet.md) | [GitHub][api_sample_net] |

### Batch 工具
以下是在生成和调试 Batch 应用程序和服务时可以使用的一些有用工具，虽然在使用 Batch 生成解决方案时并不需要这些工具。

- [Azure 门户预览][portal]：可以在 Azure 门户预览的 Batch 边栏选项卡中创建、监视和删除 Batch 池、作业和任务。用户运行作业时，可以查看这些资源和其他资源的状态信息，甚至从池中的计算节点下载文件（例如，在进行故障排除时下载失败任务的 `stderr.txt`）。用户还可以下载可用于登录到计算节点的远程桌面 (RDP) 文件。
- [Azure Batch Explorer][batch_explorer]：Batch Explorer 提供与 Azure 门户预览类似的 Batch 资源管理功能，但位于独立的 Windows Presentation Foundation (WPF) 客户端应用程序中。[GitHub][github_samples] 上提供的一个 Batch .NET 示例应用程序，用户可以使用 Visual Studio 2015 或更高版本生成它，然后在开发和调试 Batch 解决方案时，使用它浏览和管理 Batch 帐户中的资源。可以查看作业、池和任务详细信息，从计算节点下载文件，以及使用可通过 Batch Explorer 下载的远程桌面 (RDP) 文件以远程方式连接到节点。
- [Azure 存储资源管理器][storage_explorer]：虽然严格地说，存储资源管理器不算是 Azure Batch 工具，但却是开发和调试 Batch 解决方案时的另一个很有用的工具。

## 方案：扩大并行工作负荷
使用 Batch API 来与 Batch 服务交互的一个常见方案涉及在计算节点池上放大本质并行任务，例如渲染 3D 场景的图像。例如，此计算节点池可能是“渲染场”，为渲染作业提供数十、数百甚至数千个核心。

下图显示一个常见的 Batch 工作流，其中有一个客户端应用程序或托管服务使用 Batch 运行并行工作负荷。

![Batch 解决方案工作流][2]

在此常见方案中，应用程序或服务执行以下步骤，在 Azure Batch 中处理计算工作负荷：

1. 将**输入文件**和处理这些文件的**应用程序**上载到 Azure 存储帐户。输入文件可以是应用程序要处理的任何数据，例如金融建模数据或要转码的视频文件。应用程序文件可以是任何用于处理数据的应用程序，例如 3D 渲染应用程序或媒体转码器。
2. 在 Batch 帐户中创建计算节点的 Batch **池** - 这些节点是将执行任务的虚拟机。需要指定属性，例如[节点大小](../cloud-services/cloud-services-sizes-specs.md)、其操作系统，以及节点加入池时要安装的应用程序在 Azure 存储中的位置（在步骤 1 中上载的应用程序）。还可将池配置为随着任务所生成的工作负荷而[自动缩放](./batch-automatic-scaling.md)。自动缩放功能可动态调整池中计算节点的数目。
3. 创建 Batch **作业**，在计算节点池上运行工作负荷。创建作业时，需要将它与 Batch 池关联。
4. 将**任务**添加到作业。当你将任务添加到作业时，Batch 服务将自动计划任务在池中的计算节点上执行。每项任务使用上载的应用程序来处理输入文件。

   - 4a.任务执行之前，它可以将它要处理的数据（输入文件）下载到它被分配到的计算节点。如果应用程序未安装在节点上（请参阅步骤 2#），可以从此处下载。下载完成后，任务将在它被分配到的节点上执行。
5. 任务执行时，你可以查询 Batch 来监视作业及其任务的进度。客户端应用程序或服务通过 HTTPS 与 Batch 服务通信。由于可能要监视数千个计算节点上运行的数千个任务，因此请务必[有效地查询 Batch 服务](./batch-efficient-list-queries.md)。
6. 当任务完成时，它们可以将其输出数据上载到 Azure 存储空间。也可直接从计算节点上的文件系统检索文件。
7. 当监视检测到作业中的任务已完成时，客户端应用程序或服务可以下载输出数据来进一步处理或评估。

请记住，这只是使用 Batch 的一种方式，此方案只描述它的几项可用功能。例如，可以在每个计算节点上[以并行方式执行多项任务](./batch-parallel-node-tasks.md)，也可以使用[作业准备和完成任务](./batch-job-prep-release.md)来准备作业的节点，然后再清除。

## 后续步骤
在大致了解 Batch 服务后，接下来可以更深入探索该服务，以了解如何使用它来处理计算密集型并行工作负荷。

- 想要使用 Batch 的任何人都必须阅读[面向开发人员的 Batch 功能概述](./batch-api-basics.md)中的信息。该文章包含有关 Batch 服务资源池、节点、作业和任务，以及构建 Batch 应用程序时可以使用的许多 API 功能的详细信息。
- 阅读 [Get started with the Azure Batch library for .NET](./batch-dotnet-get-started.md)（适用于 .NET 的 Azure Batch 库入门），了解如何使用 C# 和 Batch .NET 库在常见的 Batch 工作流中执行简单的工作负荷。若要了解如何使用 Batch 服务，应先学习此文。此外还提供了该教程的 [Python 版本](./batch-python-tutorial.md)。
- 下载 [GitHub 上的代码示例][github_samples]，了解如何通过综合使用 C# 和 Python 与 Batch 来计划和处理示例工作负荷。

[azure_storage]: https://www.azure.cn/home/features/storage/
[api_java]: http://azure.github.io/azure-sdk-for-java/
[api_java_jar]: http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-batch%22
[api_net]: https://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[api_net_nuget]: https://www.nuget.org/packages/Azure.Batch/
[api_rest_mgmt]: https://docs.microsoft.com/zh-cn/rest/api/batchmanagement/
[api_net_mgmt]: https://msdn.microsoft.com/zh-cn/library/azure/mt463120.aspx
[api_net_mgmt_nuget]: https://www.nuget.org/packages/Microsoft.Azure.Management.Batch/
[api_nodejs]: http://azure.github.io/azure-sdk-for-node/azure-batch/latest/
[api_nodejs_npm]: https://www.npmjs.com/package/azure-batch
[api_python]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.html
[api_python_pypi]: https://pypi.python.org/pypi/azure-batch
[api_sample_net]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp
[api_sample_python]: https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch
[api_sample_java]: https://github.com/Azure/azure-batch-samples/tree/master/Java/
[batch_ps]: https://msdn.microsoft.com/zh-cn/library/azure/mt125957.aspx
[batch_rest]: https://msdn.microsoft.com/zh-cn/library/azure/Dn820158.aspx
[free_account]: https://www.azure.cn/pricing/1rmb-trial/
[github_samples]: https://github.com/Azure/azure-batch-samples
[batch_explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[storage_explorer]: http://storageexplorer.com/
[portal]: https://portal.azure.cn

[1]: ./media/batch-technical-overview/tech_overview_01.png
[2]: ./media/batch-technical-overview/tech_overview_02.png

<!---HONumber=Mooncake_0116_2017-->
<!---Update_Description: wording update -->