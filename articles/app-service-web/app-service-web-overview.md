---
title: Web Apps 概述 | Azure
description: 了解 Azure App Service 如何帮助用户开发和托管 Web 应用程序
services: app-service\web
documentationcenter: ''
author: cephalin
manager: erikre
editor: ''

ms.assetid: 94af2caf-a2ec-4415-a097-f60694b860b3
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 01/04/2017
wacn.date: 02/10/2017
ms.author: cephalin
---

# Web Apps 概述
*应用服务 Web 应用*是一个完全托管的计算平台，非常适合用来托管网站和 Web 应用程序。Azure 的这个[平台即服务](https://zh.wikipedia.org/wiki/平台即服务) (PaaS) 产品可让用户专注于业务逻辑，而让 Azure 负责处理用来运行和缩放应用的基础结构。

## 应用服务中的 Web 应用是什么？
在应用服务中，*Web 应用*是 Azure 提供用来托管网站或 Web 应用程序的计算资源。

该计算资源可能位于共享虚拟机 (VM) 上，也可能位于专用虚拟机上，具体取决于你选择的定价层。你的应用程序代码在独立于其他客户的托管 VM 中运行。

你的代码可以使用 [Azure App Service](../app-service/app-service-value-prop-what-is.md) 支持的任何语言或框架，例如 ASP.NET、Node.js、Java、PHP 或 Python。也可以在 Web 应用中运行使用 [PowerShell 和其他脚本语言](./web-sites-create-web-jobs.md#acceptablefiles)的脚本。

有关可使用 Web 应用的典型应用程序的应用场景示例，请参阅 [Azure App Service、虚拟机、Service Fabric 和云服务的比较](./choose-web-site-cloud-service-vm.md#scenarios)的**应用场景和建议**一节。

## 为何使用 Web Apps？
以下是适用于 Web 应用的一些主要应用服务功能：

* **多种语言和框架** — 应用服务为 ASP.NET、Node.js、Java、PHP 和 Python 提供一流支持。也可以在应用服务 VM 上运行 [PowerShell 和其他脚本或可执行文件](./web-sites-create-web-jobs.md)。
* **开发运营优化** - 使用 GitHub 设置[持续集成和部署](./app-service-continuous-deployment.md)。通过[测试和过渡环境](./web-sites-staged-publishing.md)提升更新。执行 [A/B 测试](./app-service-web-test-in-production-get-start.md)。使用 [Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs) 或[跨平台命令行接口 (CLI)](../xplat-cli-install.md) 在应用服务中管理应用。
* **具有高可用性的全局缩放** - 以手动或自动方式[增加](./web-sites-scale.md)或[扩大](../monitoring-and-diagnostics/insights-how-to-scale.md)。在 Azure.cn 全国数据中心基础结构中的任意位置托管应用，并且应用服务 [SLA](https://www.azure.cn/support/sla/app-service/) 承诺高可用性。
* **连接到 SaaS 平台和本地数据** - 从适用于企业系统（例如 SAP、Siebel 和 Oracle）的 50 多个连接器、SaaS 服务（例如 Salesforce 和 Office 365）以及 Internet 服务中进行选择。使用 [Azure 虚拟网络](./app-service-vnet-integration-powershell.md)访问本地数据。
* **安全性和合规性** - 应用服务符合 [ISO、SOC 和 PCI](https://www.trustcenter.cn/) 的要求。
* **Visual Studio 集成** — Visual Studio 中的专用工具可简化创建、部署和调试工作。

此外，Web 应用可以利用 [API 应用](../app-service-api/app-service-api-apps-why-best-platform.md)提供的功能（例如 CORS 支持）和[移动应用](../app-service-mobile/app-service-mobile-value-prop.md)提供的功能（例如推送通知）。有关应用服务中应用类型的详细信息，请参阅 [Azure App Service 概述](../app-service/app-service-value-prop-what-is.md)。

除了应用服务中的 Web 应用，Azure 还提供可用来托管网站和 Web 应用程序的其他服务。大多数情况下，Web Apps 是最佳选择。对于微服务体系结构，请考虑使用 [Service Fabric](../service-fabric/index.md)；如果需要更好地控制运行代码的 VM，请考虑使用 [Azure 虚拟机](../virtual-machines/index.md)。有关如何在这些 Azure 服务之间做出选择的详细信息，请参阅 [Azure App Service、虚拟机、Service Fabric 和云服务的比较](./choose-web-site-cloud-service-vm.md)。

## 入门
若要首先在应用服务中向新 Web 应用部署示例代码，请遵循以下下拉框中的教程之一。需要一个 Azure 试用帐户。

> [!div class="op_single_selector"]
- [在 5 分钟内将第一个 HTML 站点部署到 Azure](./app-service-web-get-started-html-cli-nodejs.md)
- [在 5 分钟内将第一个 ASP.NET Web 应用部署到 Azure](./app-service-web-get-started-dotnet-cli-nodejs.md)
- [在 5 分钟内将第一个 PHP Web 应用部署到 Azure](./app-service-web-get-started-php-cli-nodejs.md)
- [在 5 分钟内将第一个 Node.js Web 应用部署到 Azure](./app-service-web-get-started-nodejs-cli-nodejs.md)
- [在 5 分钟内将第一个 Python Web 应用部署到 Azure](./app-service-web-get-started-python-cli-nodejs.md)
- [在 5 分钟内将第一个 Java Web 应用部署到 Azure](./app-service-web-get-started-java.md)

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description: add selector for the get-started tutorial-->