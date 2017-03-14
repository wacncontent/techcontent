---
title: 为 Azure 服务创建警报 - 跨平台 CLI | Azure
description: 满足指定的条件时，触发电子邮件、通知、调用网站 URL (webhook) 或自动执行。
authors: rboucher
manager: carmonm
editor: ''
services: monitoring-and-diagnostics
documentationCenter: monitoring-and-diagnostics

ms.service: monitoring-and-diagnostics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/24/2016
wacn.date: 03/03/2017
ms.author: robb
---

# 在 Azure Monitor 中为 Azure 服务创建警报 - 跨平台 CLI

> [!div class="op_single_selector"]
- [门户预览](./insights-alerts-portal.md)
- [PowerShell](./insights-alerts-powershell.md)
- [CLI](./insights-alerts-command-line-interface.md)

## 概述
本文说明如何使用跨平台命令行接口 (CLI) 设置 Azure 警报。

> [!NOTE]
> “Azure Insights”在 2016 年 9 月 25 日后称为 Azure Monitor。但是，命名空间和以下命令中仍然包含“insights”。

可以根据监视指标或事件接收 Azure 服务的警报。

- **指标值** - 当指定指标的值在任一方向越过了指定的阈值时警报将触发。也就是说，当条件先是满足以及之后不再满足该条件时，警报都会触发。
- **活动日志事件** - 警报可以在发生 *每个* 事件时都触发，也可以仅在发生特定数量的事件时触发。

可以配置警报以在其触发时执行以下操作：

- 向服务管理员和共同管理员发送电子邮件通知
- 向指定的其他电子邮件地址发送电子邮件。
- 调用 Webhook
- 开始执行 Azure Runbook（目前仅在 Azure 门户预览中可行）

可以使用以下工具配置和获取关于警报的信息：

- [Azure 门户预览](./insights-alerts-portal.md)
- [PowerShell](./insights-alerts-powershell.md)
- [命令行界面 (CLI)](./insights-alerts-command-line-interface.md)
- [Azure Insights REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931945.aspx)

始终可以通过键入一个命令并将 -help 置于末尾来获取有关该命令的帮助信息。例如：

```
azure insights alerts -help
azure insights alerts actions email create -help
```

## 使用 CLI 创建警报规则
1. 执行先决条件并登录到 Azure。请参阅 [Azure Monitor CLI 示例](./insights-cli-samples.md)。简而言之，就是安装 CLI 并运行以下命令。这些命令可帮助登录、显示当前使用的订阅，以及为运行 Azure Monitor 命令做好准备。

    ```
    azure login -e AzureChinaCloud
    azure account show
    azure config mode arm 
    ```

2.  若要列出资源组上的现有规则，请使用以下形式：**azure insights alerts rule list** *[options] &lt;resourceGroup&gt;*

    ```
    azure insights alerts rule list myresourcegroupname
    ```

3. 若要创建规则，需要首先获得以下几条重要信息。
    - 要为其设置警报的资源的**资源 ID**
    - 可用于该资源的**指标定义**

    获取资源 ID 的方法是使用 Azure 门户预览。假定该资源已创建，则在门户预览中将其选中。然后，在下一个边栏选项卡中，在“设置”部分下选择“属性”。“资源 ID”是下一个边栏选项卡中的一个字段。
    下面是 Web 应用的一个示例资源 ID：

    ```
    /subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename
    ```

    对于前面的资源示例，若要获取可用指标的列表和这些指标的单位，请使用以下 CLI 命令：

    ```
    azure insights metrics list /subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename PT1M 
    ```

    _PT1M_ 是可用度量的粒度（1 分钟时间间隔）。使用不同的粒度可以提供不同的指标选项。

4. 若要创建基于指标的警报规则，请使用以下形式的命令：

    ```
    **azure insights alerts rule metric set** *[options] &lt;ruleName&gt; &lt;location&gt; &lt;resourceGroup&gt; &lt;windowSize&gt; &lt;operator&gt; &lt;threshold&gt; &lt;targetResourceId&gt; &lt;metricName&gt; &lt;timeAggregationOperator&gt;*
    ```

    以下示例在一个网站资源上设置警报。当在 5 分钟内持续收到任何流量以及再次在 5 分钟内未收到任何流量时，警报将触发。

    ```
    azure insights alerts rule metric set myrule eastus myreasourcegroup PT5M GreaterThan 2 /subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename BytesReceived Total
    ```

5. 若要在警报触发时创建 Webhook 或发送电子邮件，请首先创建电子邮件和/或 Webhook。然后紧随其后创建规则。无法使用 CLI 将 Webhook 或电子邮件与已创建的规则相关联。

    ```
    azure insights alerts actions email create --customEmails myemail@contoso.com

    azure insights alerts actions webhook create https://www.contoso.com

    azure insights alerts rule metric set myrulewithwebhookandemail chinaeast myreasourcegroup PT5M GreaterThan 2 /subscriptions/dededede-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/myresourcegroupname/providers/Microsoft.Web/sites/mywebsitename BytesReceived Total
    ```

6. 若要创建在活动日志中出现特定条件时触发的警报，请使用以下形式：

    ```
    **insights alerts rule log set** *[options] &lt;ruleName&gt; &lt;location&gt; &lt;resourceGroup&gt; &lt;operationName&gt;*
    ```

    例如

    ```
    azure insights alerts rule log set myActivityLogRule eastus myresourceGroupName Microsoft.Storage/storageAccounts/listKeys/action
    ```

    operationName 对应于活动日志中某个条目的事件类型。示例有 *Microsoft.Compute/virtualMachines/delete* 和 *microsoft.insights/diagnosticSettings/write*。

    可以使用 PowerShell 命令 [Get-AzureRmProviderOperation](https://msdn.microsoft.com/zh-cn/library/mt603720.aspx) 获取可能的 operationName 的列表。另外，还可以使用 Azure 门户预览来查询活动日志并查找要为其创建警报的过去的特定操作。各项操作以友好名称显示在图形化日志视图中。请查看条目的 JSON 并找出 OperationName 名称。

7. 可以通过查看各个规则来验证是否已正确创建了警报。

    ```
    azure insights alerts rule list myresourcegroup --ruleName myrule
    ```

8. 若要删除规则，请使用以下形式的命令：

    ```
    **insights alerts rule delete** [options] &lt;resourceGroup&gt; &lt;ruleName&gt;
    ```

    这些命令将删除本文中前面创建的规则。

    ```
    azure insights alerts rule delete myresourcegroup myrule
    azure insights alerts rule delete myresourcegroup myrulewithwebhookandemail
    azure insights alerts rule delete myresourcegroup myActivityLogRule
    ```

## 后续步骤

* 详细了解[在警报中配置 Webhook](./insights-webhooks-alerts.md)。
* 详细了解 [Azure 自动化 Runbook](../automation/automation-starting-a-runbook.md)。
* [大致了解指标收集](./insights-how-to-customize-monitoring.md)以确保你的服务可用且响应迅速。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description:update wording -->