---
title: 使用门户为应用程序网关创建基于路径的规则 | Azure
description: 了解如何使用门户为应用程序网关创建基于路径的规则
services: application-gateway
documentationCenter: na
authors: georgewallace
manager: carmonm
editor: ''
tags: azure-resource-manager

ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/13/2016
wacn.date: 01/25/2017
ms.author: gwallace
---

# 使用门户为应用程序网关创建基于路径的规则

> [!div class="op_single_selector"]
- [Azure 门户预览](./application-gateway-create-url-route-portal.md)
- [Azure Resource Manager PowerShell](./application-gateway-create-url-route-arm-ps.md)

借助基于 URL 路径的路由，可根据 Http 请求的 URL 路径来关联路由。它将检查是否有路由连接到针对应用程序网关中的 URL 列表配置的后端池，并将网络流量发送到定义的后端池。基于 URL 的路由的常见用法是将不同内容类型的请求负载均衡到不同的后端服务器池。

基于 URL 的路由将新的规则类型引入应用程序网关。应用程序网关有两种规则类型：基本规则和基于路径的规则。基本规则类型针对后端池提供轮循机制服务，而基于路径的规则除了轮循机制分发以外，还在选择后端池时考虑请求 URL 的路径模式。

## 方案

以下方案演示如何在现有应用程序网关中创建基于路径的规则。
该方案假定你已按相关步骤[创建应用程序网关](./application-gateway-create-gateway-portal.md)。

![url 路由][scenario]  

## <a name="createrule"></a>创建基于路径的规则

基于路径的规则需要自己的侦听器，在创建该规则之前，请务必确认有侦听器可供使用。

### 步骤 1

导航到 [Azure 门户预览](http://portal.azure.cn)， 然后选择现有的应用程序网关。单击“规则”

![应用程序网关概述][1]  

### 步骤 2

单击“基于路径”按钮，添加新的基于路径的规则。

### 步骤 3

“添加基于路径的规则”边栏选项卡分为两个部分。第一个部分可供定义侦听器、规则的名称和默认路径设置。默认路径设置适用于不属于基于路径的自定义路由的路由。“添加基于路径的规则”边栏选项卡的第二个部分可供定义基于路径的规则本身。

**基本设置**

- **名称** - 这是可在门户中访问的规则的友好名称。
- **侦听器** - 这是用于规则的侦听器。
- **默认后端池** - 此设置可定义要用于默认规则的后端
- **默认 HTTP 设置** - 此设置可定义要用于默认规则的 HTTP 设置。

**基于路径的规则**

- **名称** - 这是基于路径的规则的友好名称。
- **路径** - 此设置可定义规则在转发流量时将要寻找的路径
- **后端池** - 此设置可定义要用于规则的后端
- **HTTP 设置** - 此设置可定义要用于规则的 HTTP 设置。

>[!IMPORTANT]
> 路径：要匹配的路径模式列表。每个模式必须以 / 开头，“\*”只允许放在末尾处。有效示例包括 /xyz、/xyz* 或 /xyz/*。

![添加填写了信息的“基于路径的规则”边栏选项卡][2]  

将基于路径的规则添加到现有应用程序网关是可以通过门户完成的简单过程。创建基于路径的规则后，即可对其进行编辑，以便轻松地添加其他规则。

![添加其他基于路径的规则][3]  

## 后续步骤

若要了解如何通过 Azure 应用程序网关来配置 SSL 卸载，请参阅[配置 SSL 卸载](./application-gateway-ssl-portal.md)

[1]: ./media/application-gateway-create-url-route-portal/figure1.png
[2]: ./media/application-gateway-create-url-route-portal/figure2.png
[3]: ./media/application-gateway-create-url-route-portal/figure3.png
[scenario]: ./media/application-gateway-create-url-route-portal/scenario.png

<!---HONumber=Mooncake_1010_2016-->