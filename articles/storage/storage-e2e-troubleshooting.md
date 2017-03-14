---
title: 使用 Azure 存储度量值和日志记录、AzCopy 和 Message Analyzer 进行端到端故障排除 | Azure
description: 本教程演示如何使用 Azure 存储分析、AzCopy 和 Microsoft Message Analyzer 进行端到端故障排除
services: storage
documentationcenter: dotnet
author: robinsh
manager: timlt

ms.assetid: 643372a3-1c07-4e88-b4ef-042512a43086
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 12/08/2016
wacn.date: 01/06/2017
ms.author: robinsh
---

# 使用 Azure 存储指标和日志记录、AzCopy 及 Message Analyzer 进行点对点故障排除

[!INCLUDE [storage-selector-portal-e2e-troubleshooting](../../includes/storage-selector-portal-e2e-troubleshooting.md)]

## 概述

诊断和故障排除是生成和支持使用 Azure 存储的客户端应用程序的关键技能。由于 Azure 应用程序的分布特性，诊断和排查错误与性能问题可能会比在传统环境中更为复杂。

在本教程中，将演示如何识别某些会影响性能的错误，并使用 Microsoft 和 Azure 存储提供的工具以点对点的方式排查这些错误，以优化客户端应用程序。

本教程提供了点对点故障排除方案的实践分析。有关排查 Azure 存储应用程序问题的深入化概念指南，请参阅[监视、诊断和排查 Azure 存储问题](./storage-monitoring-diagnosing-troubleshooting.md)。

## Azure 存储应用程序故障排除工具

若要使用 Azure 存储排查客户端应用程序问题，可组合使用多种工具来确定问题出现的时间以及可能的原因。这些工具包括：

- **Azure 存储分析**。[Azure 存储分析](http://msdn.microsoft.com/zh-cn/library/azure/hh343270.aspx)提供 Azure 存储的指标和日志记录。
    - **存储指标**用于跟踪存储帐户的事务指标和容量指标。使用指标，可根据各项指标确定应用程序的性能。有关存储分析跟踪的指标类型的详细信息，请参阅[存储分析指标表架构](http://msdn.microsoft.com/zh-cn/library/azure/hh343264.aspx)。

    - **存储日志记录**可以在服务器端日志中记录发送到 Azure 存储服务的每个请求。日志用于跟踪每个请求的详细数据，包括执行的操作、操作的状态和延迟信息。有关存储分析写入日志的请求和响应数据的详细信息，请参阅[存储分析日志格式](http://msdn.microsoft.com/zh-cn/library/azure/hh343259.aspx)。

- **Azure 门户预览**。可在 [Azure 门户预览](https://portal.azure.cn)中配置存储帐户的指标和日志记录。还可以查看显示应用程序在各时间段执行情况的图表和图形，以及配置警报，以便在应用程序的特定指标不同于预期时接收通知。

    请参阅[在 Azure 门户预览中监视存储帐户](./storage-monitor-storage-account.md)，了解如何在 Azure 门户预览中的配置监视功能。

- **AzCopy**。Azure 存储的服务器日志存储为 Blob，因此可以使用 AzCopy 将日志 Blob 复制到本地目录，以使用 Microsoft Message Analyzer 进行分析。有关 AzCopy 的详细信息，请参阅[使用 AzCopy 命令行实用程序传输数据](./storage-use-azcopy.md)。

- **Microsoft Message Analyzer**。Message Analyzer 是一个工具，它使用日志文件并以可视格式显示日志数据，方便筛选、搜索日志数据，以及将日志数据组合成有用的集，用于分析错误和性能问题。有关 Message Analyzer 的详细信息，请参阅 [Microsoft Message Analyzer 操作指南](http://technet.microsoft.com/zh-cn/library/jj649776.aspx)。

## 关于示例情景

在本教程中，将介绍 Azure 存储指标指示调用 Azure 存储的应用程序成功率较低的情景。低成功率指标（在 [Azure 门户预览](https://portal.azure.cn)和指标表中显示为 **PercentSuccess**）用于跟踪已经成功，但返回的 HTTP 状态代码大于 299 的操作。在服务器端存储日志文件中，这些操作将使用事务状态 **ClientOtherErrors** 进行记录。有关低成功率指标的详细信息，请参阅[指标显示低 PercentSuccess，或者分析日志项包含事务状态为 ClientOtherErrors 的操作](./storage-monitoring-diagnosing-troubleshooting.md#metrics-show-low-percent-success)。

Azure 存储操作可能返回 HTTP 状态代码大于 299 作为其正常功能的一部分。但在某些情况下，这些错误指示可能能够优化客户端应用程序以提高性能。

在此情况下，将低成功率视为低于 100% 的任何状态。但也可以根据自己的需求选择不同的指标级别。建议在测试期间，为关键性能指标建立基线容差。例如，可以根据测试，确定应用程序应该持续保持 90% 或 85% 的成功率。如果指标数据显示应用程序与该数字有偏差，则可以调查导致成功率上升的原因。

对于示例方案，一旦建立了成功率指标低于 100%，将检查日志以查找与指标相关的错误，并使用它们来找出导致较低成功率的原因。将专门介绍 400 范围内的错误。然后，将更细致地调查 404（未找到）错误。

### 400 范围错误的某些原因

以下示例中显示了对 Azure Blob 存储发出请求后返回的某些 400 范围错误示例和可能的原因。其中的任何错误，以及 300 范围和 500 范围内的错误，可能会导致低成功率。

请注意，以下列表远远算不上完整。有关一般性 Azure 存储错误以及特定于每个存储服务的错误的详细信息，请参阅 MSDN 上的[状态和错误代码](http://msdn.microsoft.com/zh-cn/library/azure/dd179382.aspx)。

**状态代码 404（找不到）示例**

当针对容器或 Blob 的读取操作失败，或者由于找不到 Blob 或容器而失败时发生。

- 当容器或 Blob 被此请求之前的另一个客户端删除时发生。
- 当使用的 API 调用在检查容器或 Blob 是否存在后创建容器或 Blob 时发生。CreateIfNotExists API 先执行 HEAD 调用以检查容器或 Blob 是否存在；如果不存在，则将返回 404 错误，然后再次执行 PUT 调用以写入容器或 Blob。

**状态代码 409（冲突）示例**

- 使用 Create API 创建新容器或 Blob，但未事先检查存在性，而已经存在同名的容器或 Blob 时，将出现此错误。
- 如果要删除某个容器，而在删除操作完成之前尝试创建同名的新容器，将出现此错误。
- 如果在容器或 Blob 中指定了租约，但已存在租约，则会发生此错误。

**状态代码 412（前置条件失败）示例**

- 当未满足条件标头指定的条件时发生。
- 当指定的租约 ID 与容器或 Blob 上的租约 ID 不匹配时发生。

## 生成日志文件用于分析

在本教程中，将使用 Message Analyzer 来处理三种不同类型的日志文件，但可以从以下类型中选择要处理的类型：

- **服务器日志**，这是在启用 Azure 存储日志记录时创建的。服务器日志包含有关针对 Azure 存储服务（Blob、队列、表和文件）之一调用的每个操作的数据。服务器日志将指示调用的操作、返回的状态代码，以及有关请求和响应的其他详细信息。
- **.NET 客户端日志**，这是在从 .NET 应用程序内部启用客户端日志记录时创建的。客户端日志包括有关客户端准备请求以及接收和处理响应的详细信息。
- **HTTP 网络跟踪日志**，它收集有关 HTTP/HTTPS 请求的数据和响应数据，包括针对 Azure 存储的操作的数据。在本教程中，将通过 Message Analyzer 生成网络跟踪。

### 配置服务器端日志记录和指标

首先，需要配置 Azure 存储日志记录和指标，以便可以从客户端应用程序获取要分析的数据。可以通过不同的方式配置日志记录和指标 - 通过 [Azure 门户预览](https://portal.azure.cn)、使用 PowerShell 或以编程方式。有关配置日志记录和度量的详细信息，请参阅 MSDN 上的[启用存储指标和查看指标数据](http://msdn.microsoft.com/zh-cn/library/azure/dn782843.aspx)及[启用存储日志记录和访问日志数据](http://msdn.microsoft.com/zh-cn/library/azure/dn782840.aspx)。

**通过 Azure 门户预览**

若要使用 [Azure 门户预览](https://portal.azure.cn)配置存储帐户的日志记录和指标，请遵循[监视 Azure 门户预览中的存储帐户](./storage-monitor-storage-account.md)中的说明。

> [!NOTE]
> 无法使用 Azure 门户预览设置分钟指标。但是，对于本教程，建议设置分钟指标，它还可以调查应用程序的性能问题。可以使用 PowerShell（如下所示）设置分钟指标，也可以使用存储客户端库通过编程方式来进行。
>
> 请注意，Azure 门户预览无法显示分钟指标，而只显示小时指标。

**通过 PowerShell**

若要开始使用用于 Azure 的 PowerShell，请参阅 [如何安装和配置 Azure PowerShell](../powershell-install-configure.md)。

1. 使用 [Add-AzureAccount](http://msdn.microsoft.com/zh-cn/library/azure/dn722528.aspx) cmdlet 将 Azure 用户帐户添加到 PowerShell 窗口中：

    ```
    Add-AzureAccount -Environment AzureChinaCloud
    ```

2. 在“登录 Azure”窗口中，键入与帐户关联的电子邮件地址和密码。Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。
3. 通过在 PowerShell 窗口中执行以下命令，将默认存储帐户设置为用于本教程的存储帐户：

    ```
    $SubscriptionName = 'Your subscription name'
    $StorageAccountName = 'yourstorageaccount' 
    Set-AzureSubscription -Environment AzureChinaCloud -CurrentStorageAccountName $StorageAccountName -SubscriptionName $SubscriptionName 
    ```

4. 为 Blob 服务启用存储日志记录：

    ```
    Set-AzureStorageServiceLoggingProperty -ServiceType Blob -LoggingOperations Read,Write,Delete -PassThru -RetentionDays 7 -Version 1.0 
    ```

5. 为 Blob 服务启用存储指标，确保将 **-MetricsType** 设置为 `Minute`：

    ```
    Set-AzureStorageServiceMetricsProperty -ServiceType Blob -MetricsType Minute -MetricsLevel ServiceAndApi -PassThru -RetentionDays 7 -Version 1.0 
    ```

### 配置 .NET 客户端日志记录

若要为 .NET 应用程序配置客户端日志记录，请在应用程序的配置文件（web.config 或 app.config）中启用 .NET 诊断。有关详细信息，请参阅 MSDN 上的[使用 .NET 存储客户端库进行的客户端日志记录](http://msdn.microsoft.com/zh-cn/library/azure/dn782839.aspx)和[通过 Azure Storage SDK for Java 进行的客户端日志记录](http://msdn.microsoft.com/zh-cn/library/azure/dn782844.aspx)。

客户端日志包括有关客户端准备请求以及接收和处理响应的详细信息。

存储客户端库将客户端的日志数据存储在应用程序的配置文件（web.config 或 app.config）中的指定位置。

### 收集网络跟踪

当客户端应用程序正在运行时，可以使用 Message Analyzer 来收集 HTTP/HTTPS 网络跟踪。Message Analyzer 在后端使用 [Fiddler](http://www.telerik.com/fiddler)。在收集网络跟踪之前，建议配置 Fiddler 来记录未加密的 HTTPS 通信：

1. 安装 [Fiddler](http://www.telerik.com/download/fiddler)。
2. 启动 Fiddler。
2. 选择**“工具”| Fiddler Options**.
3. 在“选项”对话框中，确保“捕获 HTTPS 连接”和“解密 HTTPS 通信”都已选中，如下所示。

![配置 Fiddler 选项](./media/storage-e2e-troubleshooting/fiddler-options-1.png)

对于本教程，将先在 Message Analyzer 中收集并保存网络跟踪，然后创建分析会话以分析跟踪和日志。在 Message Analyzer 中收集网络跟踪：

1. 在 Message Analyzer 中，选择**“文件”| Quick Trace | Unencrypted HTTPS**.
2. 随后将立即开始跟踪。选择“停止”可以停止跟踪，以便将它配置为仅跟踪存储通信。
3. 选择“编辑”可以编辑跟踪会话。
4. 选择 **Microsoft-Pef-WebProxy** ETW 提供程序右侧的“配置”链接。
5. 在“高级设置”对话框中，单击“提供程序”选项卡。
6. 在“主机名筛选器”字段中，指定存储终结点，以空格分隔。例如，可按如下所示指定终结点；将 `storagesample` 更改为存储帐户名称：

    ```
    storagesample.blob.core.chinacloudapi.cn storagesample.queue.core.chinacloudapi.cn storagesample.table.core.chinacloudapi.cn 
    ```

7. 退出对话框，然后单击“重新启动”，在启用主机名筛选器的情况下开始收集跟踪，以便仅在跟踪中包含 Azure 存储网络通信。

>[!NOTE]
> 在完成收集网络跟踪后，强烈建议还原可能在 Fiddler 中为了解密 HTTPS 通信而更改的设置。在“Fiddler 选项”对话框中，取消选中“捕获 HTTPS 连接”和“解密 HTTPS 通信”复选框。

有关详细信息，请参阅 Technet 上的[使用网络跟踪功能](http://technet.microsoft.com/zh-cn/library/jj674819.aspx)。

## 在 Azure 门户预览中查看指标数据

在应用程序运行一段时间后，可以查看 [Azure 门户预览](https://portal.azure.cn)中显示的指标图表，以观察服务的性能。首先，导航到在 Azure 门户预览中的存储帐户，然后添加“成功百分比”指标图表。

现在，Azure 门户预览中的监视图上将会显示“成功百分比”，以及可能添加的任何其他指标。在此情况下，接下来要在 Message Analyzer 中通过分析日志进行调查，成功率百分比略低于 100%。

有关将指标添加到监视页的详细信息，请参阅[如何：向指标表中添加指标](./storage-monitor-storage-account.md#how-to-add-metrics-to-the-metrics-table)。

> [!NOTE]
> 在启用存储指标后，可能需要经过一段时间，指标数据才会显示在 Azure 门户预览中。这是因为，只有在当前小时已过后，前一个小时的小时指标才会显示在 Azure 门户预览中。此外，分钟指标当前不会显示在 Azure 门户预览中。因此，根据启用指标的时间，最多可能需要两个小时才能看到指标数据。

## 使用 AzCopy 将服务器日志复制到本地目录

Azure 存储将服务器日志数据写入 Blob，将指标写入表。存储帐户的已知的 `$logs` 容器中提供了日志 Blob。日志 Blob 按年、月、日和小时的分层形式命名，因此，可以轻松地找到想要调查的时间范围。例如，在 `storagesample` 帐户中，对应于 2015 年 1 月 2 日上午 8-9 点的日志 Blob 容器为 `https://storagesample.blob.core.chinacloudapi.cn/$logs/blob/2015/01/08/0800`。此容器中的各个 Blob 均按顺序命名，并以 `000000.log` 开头。

可以使用 AzCopy 命令行工具将这些服务器端日志文件下载到本地计算机上的所选位置。例如，可以使用以下命令将发生于 2015 年 1 月 2 日 Blob 操作的日志文件下载到文件夹 `C:\Temp\Logs\Server`；将 `<storageaccountname>` 替换为存储帐户名称，将 `<storageaccountkey>` 替换为帐户访问密钥：

```
AzCopy.exe /Source:http://<storageaccountname>.blob.core.chinacloudapi.cn/$logs /Dest:C:\Temp\Logs\Server /Pattern:"blob/2015/01/02" /SourceKey:<storageaccountkey> /S /V
```

可以从 [Azure 下载](/downloads/)页下载 AzCopy。有关使用 AzCopy 的详细信息，请参阅[使用 AzCopy 命令行实用程序传输数据](./storage-use-azcopy.md)。

有关下载服务器端日志的其他信息，请参阅[下载存储日志记录日志数据](http://msdn.microsoft.com/zh-cn/library/azure/dn782840.aspx#DownloadingStorageLogginglogdata)。

## 使用 Microsoft Message Analyzer 分析日志数据

Microsoft Message Analyzer 是在故障排除和诊断方案中用于捕获、显示和分析协议消息传递通信、事件和其他系统或应用程序消息的工具。Message Analyzer 还允许加载、聚合和分析日志和保存的跟踪文件中的数据。有关 Message Analyzer 的详细信息，请参阅 [Microsoft Message Analyzer 操作指南](http://technet.microsoft.com/zh-cn/library/jj649776.aspx)。

Message Analyzer 包括 Azure 存储的资产，可帮助分析服务器、客户端和网络日志。在本部分中，将讨论如何使用这些工具来解决存储日志中成功百分比较低的问题。

### 下载并安装 Message Analyzer 和 Azure 存储资产

1. 从 Microsoft 下载中心下载 [Message Analyzer](http://www.microsoft.com/download/details.aspx?id=44226)，并运行安装程序。
2. 启动 Message Analyzer。
3. 从“工具”菜单中选择“资产管理器”。在“资产管理器”对话框中，选择“下载”，然后筛选“Azure 存储”。将会看到 Azure 存储资产，如下图中所示。
4. 单击“同步所有显示的项”以安装 Azure 存储资产。可用的资产包括：
    - **Azure 存储颜色规则**：Azure 存储颜色规则可让可以定义特殊筛选器，以使用颜色、文本和字体样式来突出显示跟踪中包含特定信息的消息。
    - **Azure 存储图表**：Azure 存储图表是根据服务器日志数据绘制的预定义图表。请注意，若在此时使用 Azure 存储图表，可以只将服务器日志加载到分析网格中。
    - **Azure 存储分析程序**：Azure 存储分析程序可以分析 Azure 存储客户端、服务器和 HTTP 日志，以便在分析网格中显示。
    - **Azure 存储筛选器**：Azure 存储筛选器是可用于查询分析网格中的数据的预定义条件。
    - **Azure 存储视图布局**：Azure 存储视图布局是分析网格中的预定义列布局和分组。
4. 安装资产后，请重新启动 Message Analyzer。

![Message Analyzer 资产管理器](./media/storage-e2e-troubleshooting/mma-start-page-1.png)

> [!NOTE]
> 对于本教程，请安装显示的所有 Azure 存储资产。

### 将日志文件导入 Message Analyzer

可以将所有已保存的日志文件（服务器端、客户端和网络）导入到 Microsoft Message Analyzer 的单个会话中以进行分析。

1. 在 Microsoft Message Analyzer 中的“文件”菜单上单击“新建会话”，然后单击“空白会话”。在“新建会话”对话框中，输入分析会话的名称。在“会话详细信息”面板中，单击“文件”按钮。
1. 若要加载 Message Analyzer 生成的网络跟踪数据，请单击“添加文件”，浏览到通过 Web 跟踪会话将 .matp 文件保存到的位置，选择该 .matp 文件，然后单击“打开”。
1. 若要加载服务器端日志数据，请单击“添加文件”，浏览到将服务器端日志下载到的位置，选择要分析的时间范围内的日志文件，然后单击“打开”。在“会话详细信息”面板中，将每个服务器端日志文件的“文本日志配置”下拉列表设置为 AzureStorageLog，以确保 Microsoft Message Analyzer 可以正确分析日志文件。
1. 若要加载客户端日志数据，请单击“添加文件”，浏览到客户端日志保存到的位置，选择想要分析的日志文件，然后单击“打开”。在“会话详细信息”面板中，将每个客户端日志文件的“文本日志配置”下拉列表设置为 AzureStorageClientDotNetV4，以确保 Microsoft Message Analyzer 可以正确分析日志文件。
1. 在“新建会话”对话框中单击“开始”以加载并分析日志数据。日志数据将显示在 Message Analyzer 分析网格中。

下图显示了使用服务器、客户端和网络跟踪日志文件配置的示例会话。

![配置 Message Analyzer 会话](./media/storage-e2e-troubleshooting/configure-mma-session-1.png)

请注意，Message Analyzer 会将日志文件载入内存中。如果有大量的日志数据，则需要进行筛选，以便从 Message Analyzer 获得最佳性能。

首先，确定想要审查的时间范围，并使用尽可能小的时间范围。在许多情况下，需要审查最多几分钟或几小时的时间段。导入可以满足需求的最小日志集。

如果仍有大量的日志数据，则可能需要在加载日志数据之前指定会话筛选器以筛选数据。在“会话筛选器”框中，选择“库”按钮可以选择预定义的筛选器；例如，从 Azure 存储筛选器中选择“全局时间筛选器 I”可根据某个时间间隔进行筛选。然后，可以编辑筛选条件，以指定想要查看的间隔的起始和结束时间戳。还可以根据特定的状态代码筛选；例如，可以选择仅加载状态代码为 404 的日志条目。

有关如何将日志数据导入 Microsoft Message Analyzer 的详细信息，请参阅 TechNet 上的[检索消息数据](http://technet.microsoft.com/zh-cn/library/dn772437.aspx)。

### 使用客户端请求 ID 关联日志文件数据

Azure 存储客户端库会自动为每个请求生成唯一的客户端请求 ID。此值将写入客户端日志、服务器日志和网络跟踪，因此可以在 Message Analyzer 中使用它在所有三个日志之间关联数据。请参阅[客户端请求 ID](./storage-monitoring-diagnosing-troubleshooting.md#client-request-id)，了解有关客户端请求 ID 的更多信息。

以下部分介绍如何使用预配置的和自定义的布局视图，来根据客户端请求 ID 关联和分组数据。

### 选择要在分析网格中显示的视图布局

Message Analyzer 的存储空间资产包括 Azure 存储视图布局，这是一些预配置的视图，可用于显示不同情况下的数据，以及有用的分组和列。还可以创建自定义视图布局，并保存它们以供重复使用。

下图显示了可以通过工具栏功能区选择“视图布局”访问的“视图布局”菜单。Azure 存储的视图布局分组在菜单中的“Azure 存储”节点下。可以在搜索框中搜索 `Azure Storage`，以仅筛选 Azure 存储视图布局。还可以选择某个视图布局旁边的星形，使之成为收藏的布局并显示在菜单顶部。

![“视图布局”菜单](./media/storage-e2e-troubleshooting/view-layout-menu.png)

若要开始，请选择“按客户端请求 ID 和模块分组”。此视图布局会先根据客户端请求 ID，然后根据源日志文件（或 Message Analyzer 中的“模块”）对所有三个日志中的日志数据分组。使用此视图可以深入到特定的客户端请求 ID，并查看该客户端请求 ID 的所有三个日志文件中的数据。

下图显示了已应用到示例日志数据的此布局视图，并显示了一部分列。可以看到，对于特定的客户端请求 ID，分析网格显示了客户端日志、服务器日志和网络跟踪中的数据。

![Azure 存储视图布局](./media/storage-e2e-troubleshooting/view-layout-client-request-id-module.png)

>[!NOTE]
> 不同的日志文件具有不同的列，因此，当分析网格中显示了多个日志文件中的数据时，某些列可能不包含某一给定行的任何数据。例如，在上图中，客户端日志行未显示“时间戳”，“已用时间”、“源”和“目标”列的任何数据，因为这些列不在客户端日志中，而在网络跟踪中。同样，“时间戳”列显示了服务器日志中的时间戳数据，但未显示“已用时间”、“源”和“目标”列的任何数据，因为这些数据不在服务器日志中。

除了使用 Azure 存储视图布局以外，还可以定义并保存自己的视图布局。可以选择其他所需字段来分组数据，并将分组保存为自定义布局的一部分。

### 将颜色规则应用到分析网格

存储空间资产还包括颜色规则，用于以直观方式在分析网格中标识不同类型的错误。预定义的颜色规则将应用到 HTTP 错误，因此它们仅对服务器日志和网络跟踪显示。

若要应用颜色规则，请从工具栏功能区中选择“颜色规则”。将在菜单中看到 Azure 存储颜色规则。对于本教程，请选择“客户端错误(介于 400 和 499 之间的状态代码)”，如下图中所示。

![Azure 存储视图布局](./media/storage-e2e-troubleshooting/color-rules-menu.png)

除了使用 Azure 存储颜色规则以外，还可以定义并保存自己的颜色规则。

### 分组并筛选日志数据以查找 400 范围错误

接下来，将分组并筛选日志数据，以查找 400 范围内的所有错误。

1. 在分析网格中找到 **StatusCode** 列，右键单击列标题，然后选择“分组”。
2. 接下来，根据 **ClientRequestId** 列分组。将看到，分析网格中的数据现已根据状态代码和客户端请求 ID 进行组织。
1. 显示“视图筛选器”工具窗口（如果尚未显示）。在工具栏功能区中，选择“工具窗口”，然后选择“视图筛选器”。
2. 若要将日志数据筛选为仅显示 400 范围错误，请将以下筛选条件添加到“视图筛选器”窗口，然后单击“应用”：

    ```
    (AzureStorageLog.StatusCode >= 400 && AzureStorageLog.StatusCode <=499) || (HTTP.StatusCode >= 400 && HTTP.StatusCode <= 499)
    ```

下图显示了此分组和筛选器的结果。例如，展开状态代码 409 所对应的分组下面的 **ClientRequestID** 字段会显示导致该状态代码的操作。

![Azure 存储视图布局](./media/storage-e2e-troubleshooting/400-range-errors1.png)

应用此筛选器后，将看到已从客户端日志中排除的行，因为客户端日志不包含 **StatusCode** 列。首先，将检查服务器和网络跟踪日志，以找到 404 错误，然后会返回到客户端日志以检查导致它们的客户端操作。

>[!NOTE]
> 可以根据 **StatusCode** 列筛选，并仍显示所有三个日志（包括客户端日志）中的数据，如果将表达式添加到包括日志项的状态代码为 null 的情况的筛选器。若要构造此筛选器表达式，请使用：
>
> <code>&#42;StatusCode >= 400 或 !&#42;StatusCode</code>
>
> 此筛选器返回所有行从客户端日志和仅从服务器日志和 HTTP 日志的行的状态代码大于 400。如果将其应用到客户端请求 ID 和模块按分组视图布局，可以搜索或向下滚动到日志条目，以查找与其中表示所有三个日志。

### 筛选日志数据以查找 404 错误

存储空间资产包括可用于缩小日志数据，以找出错误或正在寻找的趋势的预定义筛选器。接下来，将要应用两个预定义的筛选器：一个用于筛选服务器和网络跟踪日志中的 404 错误，另一个用于筛选指定时间范围内的数据。

1. 显示“视图筛选器”工具窗口（如果尚未显示）。在工具栏功能区中，选择“工具窗口”，然后选择“视图筛选器”。
2. 在“视图筛选器”窗口中，选择“库”，然后搜索 `Azure Storage` 以查找 Azure 存储筛选器。选择用于筛选“所有日志中的 404（找不到）消息”的筛选器。
3. 再次显示“库”菜单，找到并选择“全局时间筛选器”。
4. 将筛选器中显示的时间戳编辑为想要查看的范围。这有助于缩小分析数据的范围。
5. 筛选器应类似于以下示例。单击“应用”将筛选器应用到分析网格。

    ```
    ((AzureStorageLog.StatusCode == 404 || HTTP.StatusCode == 404)) And
    (#Timestamp >= 2014-10-20T16:36:38 and #Timestamp <= 2014-10-20T16:36:39)
    ```

![Azure 存储视图布局](./media/storage-e2e-troubleshooting/404-filtered-errors1.png)

### 分析日志数据

现在，已分组并筛选数据，接下来可以检查生成 404 错误的各个请求的详细信息。在当前视图布局中，分别按客户端请求 ID 和日志源分组数据。由于是根据“StatusCode”字段包含 404 的请求进行筛选的，因此只能看到服务器和网络跟踪数据，而看不到客户端日志数据。

下图显示了由于 Get Blob 不存在，而导致 Get Blob 操作生成 404 的特定请求。请注意，为了显示相关数据，某些列已从标准视图中删除。

![筛选的服务器和网络跟踪日志](./media/storage-e2e-troubleshooting/server-filtered-404-error.png)

接下来，与此客户端请求 ID 与客户端日志数据相关联，以查看发生错误时客户端正在执行哪些操作。可以为此会话显示一个新的分析网格视图，以在另一个选项卡中查看客户端日志数据：

1. 首先，将 **ClientRequestId** 字段的值复制到剪贴板。为此，可以选择任一行，找到 **ClientRequestId** 字段，右键单击数据值，然后选择“复制 'ClientRequestId'”。
1. 在工具栏功能区中，选择“新建查看器”，然后选择“分析网格”打开一个新选项卡。新选项卡将显示日志文件中的所有数据，且未应用分组、筛选或颜色规则。
2. 在工具栏功能区中，选择“视图布局”，然后选择“Azure 存储”部分下的“所有 .NET 客户端列”。此视图布局显示客户端日志以及服务器和网络跟踪日志中的数据。默认情况下，这些数据已按 **MessageNumber** 列排序。
3. 接下来，搜索客户端请求 ID 的客户端日志。在工具栏功能区中，选择“查找消息”，然后在“查找”字段中指定客户端请求 ID 上的自定义筛选器。对于筛选器，请使用以下语法指定自己的客户端请求 ID：

    ```
    *ClientRequestId == "398bac41-7725-484b-8a69-2a9e48fc669a"
    ```

Message Analyzer 将查找并选择搜索条件匹配客户端请求 ID 的第一个日志条目在客户端日志中，每个客户端请求 ID 有多个条目，因此，可能需要根据 **ClientRequestId** 字段对其分组，以更轻松地查看其聚合视图。下图显示了指定的客户端请求 ID 的客户端日志中的所有消息。

![显示 404 错误的客户端日志](./media/storage-e2e-troubleshooting/client-log-analysis-grid1.png)

使用这两个选项卡中的视图布局内显示的数据，可以分析请求数据，以确定导致错误的可能原因。还可以查看此项前面的请求，以确定前一个事件是否是导致 404 错误的可能原因。例如，可以查看此客户端请求 ID 前面的客户端日志条目，以确定 Blob 是否已删除，或者错误的原因是否为客户端应用程序对容器或 Blob 调用了 CreateIfNotExists API。在客户端日志中，可以在“说明”字段中查找 Blob 的地址；在服务器和网络跟踪日志中，此信息显示在“摘要”字段中。

在知道产生 404 错误 Blob 地址后，可以进一步展开调查。通过搜索与同一 Blob 中的操作关联的其他消息的日志条目，可以检查客户端以前是否删除了实体。

## 分析其他类型的存储错误

在熟悉如何使用 Message Analyzer 分析日志数据后，可以使用视图布局、颜色规则和搜索/筛选分析其他类型的错误。下表列出了可能会遇到一些问题，以及可用于查找这些问题的筛选条件。有关构造筛选器和 Message Analyzer 筛选语言的详细信息，请参阅[筛选消息数据](http://technet.microsoft.com/zh-cn/library/jj819365.aspx)。

| 若要调查... | 使用筛选器表达式… | 将表达式应用到日志（客户端、服务器、网络、所有） |
|------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------|
| 队列上的消息传递出现意外的延迟 | AzureStorageClientDotNetV4.Description 包含“Retrying failed operation.” | 客户端 |
| PercentThrottlingError 的 HTTP 提升 | <p>HTTP.Response.StatusCode == 500</p><p> HTTP.Response.StatusCode == 503</p> | 网络 |
| PercentTimeoutError 提升 | HTTP.Response.StatusCode == 500 | 网络 |
| PercentTimeoutError 提升（全部） | *StatusCode == 500 | 全部 |
| PercentNetworkError 提升 | AzureStorageClientDotNetV4.EventLogEntry.Level < 2 | 客户端 |
| HTTP 403（禁止）消息 | HTTP.Response.StatusCode == 403 | 网络 |
| HTTP 404（未找到）消息 | HTTP.Response.StatusCode == 404 | 网络 |
| 404（全部） | *StatusCode == 404 | 全部 |
| 共享访问签名 (SAS) 授权问题 | AzureStorageLog.RequestStatus == "SASAuthorizationError" | 网络 |
| HTTP 409（冲突）消息 | HTTP.Response.StatusCode == 409 | 网络 |
| 409（全部） | *StatusCode == 409 | 全部 |
| 低 PercentSuccess，或者分析日志项包含事务状态为 ClientOtherErrors 的操作 | AzureStorageLog.RequestStatus == "ClientOtherError" | 服务器 |
| Nagle 警告 | ((AzureStorageLog.EndToEndLatencyMS - AzureStorageLog.ServerLatencyMS) > (AzureStorageLog.ServerLatencyMS * 1.5)) and (AzureStorageLog.RequestPacketSize <1460) and (AzureStorageLog.EndToEndLatencyMS - AzureStorageLog.ServerLatencyMS >= 200) | 服务器 |
| 服务器和网络日志中的时间范围 | #Timestamp >= 2014-10-20T16:36:38 and #Timestamp <= 2014-10-20T16:36:39 | 服务器、网络 |
| 服务器日志中的时间范围 | AzureStorageLog.Timestamp >= 2014-10-20T16:36:38 and AzureStorageLog.Timestamp <= 2014-10-20T16:36:39 | 服务器 |

## 后续步骤

有关在 Azure 存储中执行点对点故障排除方案的详细信息，请参阅以下资源：

- [对 Azure 存储进行监视、诊断和故障排除](./storage-monitoring-diagnosing-troubleshooting.md)
- [存储分析](http://msdn.microsoft.com/zh-cn/library/azure/hh343270.aspx)
- [在 Azure 门户预览中监视存储帐户](./storage-monitor-storage-account.md)
- [使用 AzCopy 命令行实用程序传输数据](./storage-use-azcopy.md)
- [Microsoft Message Analyzer 操作指南](http://technet.microsoft.com/zh-cn/library/jj649776.aspx)

<!---HONumber=Mooncake_0103_2017-->