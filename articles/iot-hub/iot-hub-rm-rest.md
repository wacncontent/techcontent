---
title: 使用资源提供程序 REST API 创建 Azure IoT 中心 | Azure
description: 如何使用资源提供程序 REST API 创建 IoT 中心。
services: iot-hub
documentationcenter: .net
author: dominicbetts
manager: timlt
editor: ''

ms.assetid: 52814ee5-bc10-4abe-9eb2-f8973096c2d8
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/16/2016
wacn.date: 01/13/2017
ms.author: dobett
---

# 使用资源提供程序 REST API 创建 IoT 中心 \(.NET\)
[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## 介绍
可使用 IoT 中心资源提供程序 REST API 按编程方式创建和管理 Azure IoT 中心。本教程介绍如何使用 IoT 中心资源提供程序 REST API 通过 C\# 程序创建 IoT 中心。

> [!NOTE]
Azure 具有用于创建和处理资源的两个不同的部署模型：[Azure Resource Manager 模型和经典模型](../azure-resource-manager/resource-manager-deployment-model.md)。本文介绍如何使用 Azure Resource Manager 部署模型。
> 
> 

若要完成本教程，您需要以下各项：

- Microsoft Visual Studio 2015。
- 有效的 Azure 帐户。<br/>如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。
- [Azure PowerShell 1.0][lnk-powershell-install] 或更高版本。

[!INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## 准备 Visual Studio 项目
1. 在 Visual Studio 中，使用**控制台应用程序**项目模板创建一个新的 Visual C\# Windows 项目。将该项目命名为 **CreateIoTHubREST**。
2. 在解决方案资源管理器中右键单击你的项目，然后单击“管理 NuGet 包”。

3. 在 NuGet 包管理器中，选中“包括预发行版”，然后搜索 **Microsoft.Azure.Management.ResourceManager**。单击“安装”，在“审阅更改”中单击“确定”，然后单击“我接受”以接受许可证。

4. 在 NuGet 包管理器中，搜索 **Microsoft.IdentityModel.Clients.ActiveDirectory**。单击“安装”，在“审阅更改”中单击“确定”，然后单击“我接受”以接受许可证。

6. 在 Program.cs 中，将现有 **using** 语句替换为以下内容：

    ```
    using System;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.ResourceManager.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json;
    using Microsoft.Rest;
    using System.Linq;
    using System.Threading;
    ```

6. 在 Program.cs 中，将占位符值替换为以下静态变量。在本教程前面的介绍中，已记下 **ApplicationId**、**SubscriptionId**、**TenantId** 和 **Password**。**Resource group name** 是创建 IoT 中心时使用的资源组名称，它可以是现有的资源组，也可以是新资源组。**IoT Hub name** 是创建的 IoT 中心的名称，例如 **MyIoTHub**（请注意，此名称必须全局唯一，因此，应包含姓名或姓名首字母缩写）。**Deployment name** 是部署的名称，例如 **Deployment\_01**。

    ```
    static string applicationId = "{Your ApplicationId}";
    static string subscriptionId = "{Your SubscriptionId}";
    static string tenantId = "{Your TenantId}";
    static string password = "{Your application Password}";

    static string rgName = "{Resource group name}";
    static string iotHubName = "{IoT Hub name including your initials}";
    ```

[!INCLUDE [iot-hub-get-access-token](../../includes/iot-hub-get-access-token.md)]

## 使用资源提供程序 REST API 创建 IoT 中心
在资源组中使用 [IoT 中心资源提供程序 REST API][lnk-rest-api] 创建 IoT 中心。还可使用资源提供程序 REST API 更改现有的 IoT 中心。

1. 将以下方法添加到 Program.cs：

    ```
    static void CreateIoTHub(string token)
    {

    }
    ```

2. 在 **CreateIoTHub** 方法中添加以下代码，创建 **HttpClient** 对象并在标头中指定身份验证令牌：

    ```
    HttpClient client = new HttpClient();
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    ```

3. 在 **CreateIoTHub** 方法中添加以下代码，描述如何使用 IoT 中心创建并生成 JSON 表示法。

    ```
    var description = new
    {
      name = iotHubName,
      location = "China East",
      sku = new
      {
        name = "S1",
        tier = "Standard",
        capacity = 1
      }
    };

    var json = JsonConvert.SerializeObject(description, Formatting.Indented);
    ```

4. 在 **CreateIoTHub** 方法中添加以下代码，向 Azure 提交 REST 请求、检查响应，并检索可用于监视部署任务状态的 URL：

    ```
    var content = new StringContent(JsonConvert.SerializeObject(description), Encoding.UTF8, "application/json");
    var requestUri = string.Format("https://management.chinacloudapi.cn/subscriptions/{0}/resourcegroups/{1}/providers/Microsoft.devices/IotHubs/{2}?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
    var result = client.PutAsync(requestUri, content).Result;

    if (!result.IsSuccessStatusCode)
    {
      Console.WriteLine("Failed {0}", result.Content.ReadAsStringAsync().Result);
      return;
    }

    var asyncStatusUri = result.Headers.GetValues("Azure-AsyncOperation").First();
    ```

5. 在 **CreateIoTHub** 方法末尾添加以下代码，使用上一步中检索的 **asyncStatusUri** 地址等待部署完成：

    ```
    string body;
    do
    {
      Thread.Sleep(10000);
      HttpResponseMessage deploymentstatus = client.GetAsync(asyncStatusUri).Result;
      body = deploymentstatus.Content.ReadAsStringAsync().Result;
        } while (body == "{\"status\":\"Running\"}");
    ```

6. 在 **CreateIoTHub** 方法末尾添加以下代码，检索创建的 IoT 中心的密钥并将其输出到控制台：

    ```
    var listKeysUri = string.Format("https://management.chinacloudapi.cn/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Devices/IotHubs/{2}/IoTHubKeys/listkeys?api-version=2015-08-15-preview", subscriptionId, rgName, iotHubName);
    var keysresults = client.PostAsync(listKeysUri, null).Result;

    Console.WriteLine("Keys: {0}", keysresults.Content.ReadAsStringAsync().Result);
    ```

## 完成并运行应用程序

现在，可以调用 **CreateIoTHub** 方法来完成应用程序，然后生成并运行该应用程序。

1. 在 **Main** 方法末尾添加以下代码：

    ```
    CreateIoTHub(token.AccessToken);
    Console.ReadLine();
    ```

2. 依次单击“生成”和“生成解决方案”。更正所有错误。

3. 单击“调试”，然后单击“开始调试”，运行应用程序。运行部署可能需要几分钟时间。
4. 若要验证应用程序中是否添加了新 IoT 中心，可以访问 [Azure 门户预览][lnk-azure-portal]并查看资源列表，或使用 **Get-AzureRmResource** PowerShell cmdlet。

> [!NOTE]
> 此示例应用程序添加用于计费的 S1 标准 IoT 中心。完成操作后，可以通过[门户预览][lnk-azure-portal]删除该 IoT 中心，或者在完成后使用 **Remove-AzureRmResource** PowerShell cmdlet。

## 后续步骤
现已使用资源提供程序 REST API 部署了 IoT 中心，接下来可更进一步探索：

- 有关 Azure Resource Manager 功能的详细信息，请参阅 [Azure Resource Manager 概述][lnk-azure-rm-overview]。

若要深入了解如何开发 IoT 中心，请参阅以下内容：

- [C SDK 简介][lnk-c-sdk]
- [Azure IoT SDK][lnk-sdks]

若要进一步探索 IoT 中心的功能，请参阅：

- [使用 IOT 网关 SDK 模拟设备][lnk-gateway]

<!-- Links -->
[lnk-free-trial]: https://www.azure.cn/pricing/1rmb-trial/
[lnk-azure-portal]: https://portal.azure.cn/
[lnk-powershell-install]: ../powershell-install-configure.md

[lnk-azure-rm-overview]: ../azure-resource-manager/resource-group-overview.md

[lnk-c-sdk]: ./iot-hub-device-sdk-c-intro.md
[lnk-sdks]: ./iot-hub-devguide-sdks.md

[lnk-gateway]: ./iot-hub-linux-gateway-sdk-simulated-device.md

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording-->