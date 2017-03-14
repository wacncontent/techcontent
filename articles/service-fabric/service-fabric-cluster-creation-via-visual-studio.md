---
title: 使用 Visual Studio 设置 Service Fabric 群集 | Azure
description: 介绍如何在 Visual Studio 中使用 Azure 资源组项目创建的 Azure Resource Manager 模板来设置 Azure Service Fabric 群集
services: service-fabric
documentationCenter: .net
authors: karolz-ms
manager: adegeo
editor: ''

ms.service: service-fabric
ms.devlang: dotNet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/06/2016
wacn.date: 02/21/2017
ms.author: karolz@microsoft.com
---

# 使用 Visual Studio 设置 Service Fabric 群集

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本文介绍如何使用 Visual Studio 和 Azure Resource Manager 模板设置 Azure Service Fabric 群集。我们将使用 Visual Studio Azure 资源组项目来创建模板。创建模板后，可以从 Visual Studio 直接将它部署到 Azure。也可以通过脚本使用它，或者将它用作连续集成 (CI) 工具的一部分。

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-visual-studio-login-guide.md)]

## 使用 Azure 资源组项目创建 Service Fabric 群集模板
若要开始，请打开 Visual Studio 并创建 Azure 资源组项目（位于“云”文件夹中）：

![已选中 Azure 资源组项目的“新建项目”对话框][1]

可以为此项目创建新的 Visual Studio 解决方案，或将它添加到现有解决方案。

>[!NOTE]
> 如果在“云”节点下面看不到 Azure 资源组项目，表示尚未安装 Azure SDK。启动 Web 平台安装程序（如果尚未安装，[现在请安装](http://www.microsoft.com/web/downloads/platform.aspx)），然后搜索“用于 .NET 的 Azure SDK”并安装与 Visual Studio 版本兼容的版本。

按“确定”按钮后，Visual Studio 将要求你选择想要创建的 Resource Manager 模板：

![已选中 Service Fabric 群集模板的“选择 Azure 模板”对话框][2]

选择“Service Fabric 群集”模板，然后再次点击“确定”。现在已创建项目与 Resource Manager 模板。

## 准备用于部署的模板
在部署模板以创建群集之前，必须提供所需的模板参数值。这些参数值从 `ServiceFabricCluster.parameters.json` 文件读取，该文件位于资源组项目的 `Templates` 文件夹中。打开该文件并提供以下值：

|参数名称 |说明|
|-----------------------  |--------------------------|
|adminUserName |Service Fabric 计算机（节点）的管理员帐户的名称。|
|certificateThumbprint |用于保护群集的证书的指纹。|
|sourceVaultResourceId |存储用于保护群集的证书的密钥保管库的*资源 ID*。|
|certificateUrlValue |群集安全证书的 URL。|

Visual Studio Service Fabric Resource Manager 模板将创建一个受证书保护的安全群集。此证书以最后三个模板参数标识（`certificateThumbprint`、`sourceVaultValue` 和 `certificateUrlValue`），并且必须在 **Azure 密钥保管库**中存在。有关如何创建群集安全证书的详细信息，请参阅 [Service Fabric 群集安全方案](./service-fabric-cluster-security.md#x509-certificates-and-service-fabric)一文。

## 可选：更改群集名称
每个 Service Fabric 群集都有一个名称。在 Azure 中创建结构群集时，群集名称（连同 Azure 区域）确定了群集的域名系统 (DNS) 名称。例如，如果将群集命名为 `myBigCluster`，需托管新群集的资源组的位置（Azure 区域）为“中国东部”，则群集的 DNS 名称为 `myBigCluster.chinaeast.cloudapp.chinacloudapi.cn`。

默认情况下，系统会自动生成群集名称，并在“群集”前缀后面附加一个随机后缀，使该名称唯一。这样便可以轻松使用模板作为**持续集成** (CI) 系统的一部分。如果想要为群集使用特定的名称，有效的方法之一是将 Resource Manager 模板文件 (`ServiceFabricCluster.json`) 中的 `clusterName` 变量的值设置为所选的名称。该名称是该文件中定义的第一个变量。

## 可选：添加公共应用程序端口
在部署模板之前，你还可能想要更改群集的公共应用程序端口。默认情况下，模板只打开两个公共 TCP 端口（80 和 8081）。如果应用程序需要更多端口，请修改模板中的 Azure 负载均衡器定义。此定义存储在主模板文件 (`ServiceFabricCluster.json`) 中。打开该文件并搜索 `loadBalancedAppPort`。每个端口与三个项目关联：

1. 一个模板变量，用于定义端口的 TCP 端口值：

    ```
    "loadBalancedAppPort1": "80"
    ```

2. 一个*探测*，用于定义 Azure 负载均衡器在故障转移到另一个节点之前，尝试使用特定 Service Fabric 节点的频率和时间长短。探测是负载均衡器资源的一部分。下面是第一个默认应用程序端口的探测定义：

    ```
    {
        "name": "AppPortProbe1",
        "properties": {
            "intervalInSeconds": 5,
            "numberOfProbes": 2,
            "port": "[variables('loadBalancedAppPort1')]",
            "protocol": "Tcp"
        }
    }
    ```

3. 一个*负载均衡规则*，用于将端口和探测绑定在一起，并在一组 Service Fabric 群集节点之间实现负载均衡：

    ```
    {
        "name": "AppPortLBRule1",
        "properties": {
            "backendAddressPool": {
                "id": "[variables('lbPoolID0')]"
            },
            "backendPort": "[variables('loadBalancedAppPort1')]",
            "enableFloatingIP": false,
            "frontendIPConfiguration": {
                "id": "[variables('lbIPConfig0')]"
            },
            "frontendPort": "[variables('loadBalancedAppPort1')]",
            "idleTimeoutInMinutes": 5,
            "probe": {
                "id": "[concat(variables('lbID0'),'/probes/AppPortProbe1')]"
            },
            "protocol": "Tcp"
        }
    }
    ```

如果你要部署到群集的应用程序需要更多端口，可以创建额外的探测和负载均衡规则定义来添加端口。有关如何通过 Resource Manager 模板来使用 Azure 负载均衡器的详细信息，请参阅[开始使用模板创建内部负载均衡器](../load-balancer/load-balancer-get-started-ilb-arm-template.md)。

## 使用 Visual Studio 部署模板
在 `ServiceFabricCluster.param.dev.json` 文件中保存所有必需的参数值后，可以部署模板并创建 Service Fabric 群集。在 Visual Studio 解决方案资源管理器中右键单击资源组项目，然后选择“部署 | 新建部署...”。在必要的情况下，Visual Studio 会显示“部署到资源组”对话框，要求你在 Azure 中进行身份验证：

![“部署到资源组”对话框][3]  

此对话框可让你选择群集的现有 Resource Manager 资源组，并提供用于创建新资源组的选项。通常，有利的做法是对 Service Fabric 群集使用不同的资源组。

按“部署”按钮后，Visual Studio 将提示你确认模板参数值。点击“保存”按钮。有一个参数没有持久值：群集的管理帐户密码。请根据 Visual Studio 的提示提供密码值。

>[!NOTE]
> 从 Azure SDK 2.9 开始，Visual Studio 支持在部署期间从 **Azure 密钥保管库**读取密码。在模板参数对话框中，可以看到 `adminPassword` 参数文本框右侧有一个小小的“钥匙”图标。使用此图标可以选择现有的密钥保管库密钥作为群集管理密码。首先，请务必在密钥保管库的高级访问策略中，为模板部署启用 Azure Resource Manager 访问权限。

可以在 Visual Studio 的“输出”窗口中监视部署过程的进度。一旦完成模板部署，新群集即可供使用！

>[!NOTE]
> 如果以前从未使用 PowerShell 从当前正在使用的计算机管理 Azure，则需要完成几个步骤。
>1. 运行 [`Set-ExecutionPolicy`](https://technet.microsoft.com/zh-cn/library/hh849812.aspx) 命令启用 PowerShell 脚本。开发计算机通常可接受“不受限制的”策略。
>2. 确定是否允许从 Azure PowerShell 命令收集诊断数据，并根据需要运行 [`Enable-AzureRmDataCollection`](https://msdn.microsoft.com/zh-cn/library/mt619303.aspx) 或 [`Disable-AzureRmDataCollection`](https://msdn.microsoft.com/zh-cn/library/mt619236.aspx)。这可以避免在模板部署期间不必要地出现提示。

如果出现任何错误，请转到 [Azure 门户预览](https://portal.azure.cn/)并打开已部署到的资源组。单击“所有设置”，然后在设置边栏选项卡上单击“部署”。失败的资源组部署会在“通知”中留下详细的诊断信息。

>[!NOTE]
> Service Fabric 群集需要有一定数量的节点可以启动，以保持可用性和状态 - 称为“维持仲裁”。 因此，除非已事先执行[状态的完整备份](./service-fabric-reliable-services-backup-restore.md)，否则关闭群集中的所有计算机是不安全的做法。

## 后续步骤
- [了解如何使用 Azure 门户预览设置 Service Fabric 群集](./service-fabric-cluster-creation-via-portal.md)
- [了解如何使用 Visual Studio 管理和部署 Service Fabric 应用程序](./service-fabric-manage-application-in-visual-studio.md)

<!--Image references-->

[1]: ./media/service-fabric-cluster-creation-via-visual-studio/azure-resource-group-project-creation.png
[2]: ./media/service-fabric-cluster-creation-via-visual-studio/selecting-azure-template.png
[3]: ./media/service-fabric-cluster-creation-via-visual-studio/deploy-to-azure.png

<!---HONumber=Mooncake_Quality_Review_0104_2017-->