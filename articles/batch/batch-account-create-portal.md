---
title: 创建 Azure Batch 帐户 | Azure
description: 了解如何在 Azure 门户预览中创建 Azure Batch 帐户，以便在云中运行大规模并行工作负荷
services: batch
documentationcenter: ''
author: tamram
manager: timlt
editor: ''

ms.assetid: 3fbae545-245f-4c66-aee2-e25d7d5d36db
ms.service: batch
ms.workload: big-compute
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 12/19/2016
wacn.date: 01/24/2017
ms.author: tamram
---

# 使用 Azure 门户预览创建 Azure Batch 帐户

> [!div class="op_single_selector"]
- [Azure 门户预览](./batch-account-create-portal.md)
- [Batch Management .NET](./batch-management-dotnet.md)

了解如何在 [Azure 门户预览][azure_portal]中创建 Azure Batch 帐户，以及在何处查找重要帐户属性，例如访问密钥和帐户 URL。本文还介绍 Batch 定价，以及如何将 Azure 存储帐户链接到 Batch 帐户，以便可以使用[应用程序包](./batch-application-packages.md)以及[保存作业和任务输出](./batch-task-output.md)。

## 创建 Batch 帐户
1. 登录到 [Azure 门户预览][azure_portal]。
2. 单击“新建”>“计算”>“Batch 服务”。

    ![应用商店中的 Batch][marketplace_portal]
3. “新建 Batch 帐户”边栏选项卡随即显示。请参阅以下 *a* 到 *e* 项，获取每个边栏选项卡元素的说明。

    ![创建 Batch 帐户][account_portal]  

    a.**帐户名**：批处理帐户的名称。选择的名称必须是将创建新帐户的 Azure 区域内的唯一名称（请参阅下面的“位置”）。帐户名只能包含小写字符或数字，且长度必须为 3-24 个字符。

    b.**订阅**：要在其中创建批处理帐户的订阅。如果只有一个订阅，则默认选择此项。

    c.**资源组**：为新的批处理帐户选择现有的资源组，或选择创建一个新组。

    d.**位置**：要在其中创建批处理帐户的 Azure 区域。只有订阅和资源组支持的区域显示为选项。

    e.**存储帐户**（可选）：要与新的批处理帐户关联的常规用途 Azure 存储帐户。有关详细信息，请参阅下面的[链接的 Azure 存储帐户](#linked-azure-storage-account)。

4. 单击“创建”创建帐户。

   门户将指示**正在部署**帐户，完成后，“通知”中会显示“部署成功”通知。

## 查看 Batch 帐户属性
创建帐户后，可以打开“Batch 帐户”边栏选项卡，访问其该帐户的设置和属性。可以使用“Batch 帐户”边栏选项卡中左侧的菜单访问所有帐户设置和属性。

![Azure 门户预览中的 Batch 帐户边栏选项卡][account_blade]  

- **批处理帐户 URL**：使用[批处理 API](./batch-technical-overview.md#batch-development-apis) 开发应用程序时，需要帐户 URL 才能访问批处理资源。Batch 帐户 URL 采用以下格式：

    `https://<account_name>.<region>.batch.azure.com`  

    ![门户中的 Batch 帐户 URL][account_url]  

- **访问密钥**：若要通过应用程序对批处理帐户的访问进行身份验证，需要帐户访问密钥。若要查看或重新生成 Batch 帐户的访问密钥，请在“Batch 帐户”边栏选项卡左侧菜单的“搜索”框中输入 `keys`，然后选择“密钥”。

    ![Azure 门户预览中的 Batch 帐户密钥][account_keys]  

[!INCLUDE [batch-pricing-include](../../includes/batch-pricing-include.md)]

## 链接的 Azure 存储帐户

如前所述，可选择性地将常规用途的 Azure 存储帐户链接到批处理帐户。与[批处理文件约定 .NET](./batch-task-output.md) 库一样，Batch 的[应用程序包](./batch-application-packages.md)功能将使用 Azure Blob 存储。这些可选功能有助于部署批处理任务运行的应用程序，并保存它们生成的数据。

建议创建批处理帐户专用的新存储帐户。

![创建“常规用途”存储帐户][storage_account]  

> [!NOTE] 
Azure 批处理目前仅支持常规用途的存储帐户类型。有关此帐户类型，请参见[关于 Azure 存储帐户](../storage/storage-create-storage-account.md)的步骤 5：[创建存储帐户](../storage/storage-create-storage-account.md#create-a-storage-account)。
>
>

>[!WARNING]
> 重新生成链接存储帐户的访问密钥时，请多加小心。只能重新生成一个存储帐户密钥，然后单击链接存储帐户边栏选项卡中的“同步密钥”。等待五分钟，让密钥传播到池中的计算节点，然后重新生成并同步其他密钥（如果需要）。如果同时重新生成这两个密钥，计算节点将无法同步任何一个密钥，并且无法访问存储帐户。

![重新生成存储帐户密钥][4]  

## Batch 服务的配额和限制
请注意，与 Azure 订阅和其他 Azure 服务一样，Batch 帐户也有特定的[配额和限制](./batch-quota-limit.md)。Batch 帐户的当前配额显示在门户上的帐户“属性”中。

![Azure 门户预览中的 Batch 帐户配额][quotas]  

设计和扩展 Batch 工作负荷时，请记住这些配额。例如，如果池未达到指定的计算节点目标数目，可能已达到 Batch 帐户的核心配额限制。

批处理帐户的配额为每个区域一个订阅，因此默认情况下，用户可具有多个批处理帐户，只要它们处于不同区域。可在单个批处理帐户中运行多个批处理工作负荷，或者在同一订阅、不同 Azure 区域的批处理帐户之间分散工作负荷。

此外，在 Azure 门户预览中提交免费产品支持请求，即可提高其中的多项配额。有关请求提高配额的详细信息，请参阅 [Quotas and limits for the Azure Batch service](./batch-quota-limit.md)（Azure Batch 服务的配额和限制）。

## 其他 Batch 帐户管理选项
除了使用 Azure 门户预览以外，还可以使用以下工具创建和管理 Batch 帐户：

- [Batch PowerShell cmdlet](./batch-powershell-cmdlets-get-started.md)
- [Azure CLI](../xplat-cli-install.md)
- [Batch Management .NET](./batch-management-dotnet.md)

## 后续步骤

- 若要深入了解 Batch 服务的概念和功能，请参阅 [Azure Batch feature overview](./batch-api-basics.md)（Azure Batch 功能概述）。本文讨论主要 Batch 资源（例如池、计算节点、作业和任务），并提供能够进行大规模计算工作负荷执行的服务功能概述。

- 了解使用 [Batch .NET 客户端库](./batch-dotnet-get-started.md)开发支持 Batch 的应用程序的基本概念。[简介文章](./batch-dotnet-get-started.md)介绍了使用 Batch 服务在多个计算节点上执行工作负荷的可行应用程序，并说明如何使用 Azure 存储进行工作负荷文件暂存和检索。

[api_net]: https://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[api_rest]: https://msdn.microsoft.com/zh-cn/library/azure/Dn820158.aspx

[azure_portal]: https://portal.azure.cn
[batch_pricing]: https://www.azure.cn/pricing/details/batch/

[4]: ./media/batch-account-create-portal/batch_acct_04.png "重新生成存储帐户密钥"
[marketplace_portal]: ./media/batch-account-create-portal/marketplace_batch.PNG
[account_blade]: ./media/batch-account-create-portal/batch_blade.png
[account_portal]: ./media/batch-account-create-portal/batch_acct_portal.png
[account_keys]: ./media/batch-account-create-portal/account_keys.PNG
[account_url]: ./media/batch-account-create-portal/account_url.png
[storage_account]: ./media/batch-account-create-portal/storage_account.png
[quotas]: ./media/batch-account-create-portal/quotas.png

<!---HONumber=Mooncake_0116_2017-->
<!---Update_Description: wording update -->