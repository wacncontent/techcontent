---
title: 使用 Node.js 连接到 SQL 数据库 | Azure
description: 演示了一个可以用来连接到 Azure SQL 数据库的 Node.js 代码示例。
services: sql-database
documentationcenter: ''
author: LuisBosquez
manager: jhubbard
editor: ''

ms.assetid: 53f70e37-5eb4-400d-972e-dd7ea0caacd4
ms.service: sql-database
ms.custom: development
ms.workload: drivers
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 12/24/2016
wacn.date: 01/25/2017
ms.author: lbosq
---

# 使用 Node.js 连接到 SQL 数据库

[!INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本主题说明如何使用 Node.js 来连接和查询 Azure SQL 数据库。可以从 Windows、Ubuntu Linux 或 Mac 平台运行此示例。

## 步骤 1：配置开发环境

[使用 Tedious Node.js Driver for SQL Server 的先决条件](https://msdn.microsoft.com/zh-cn/library/mt652094.aspx)

## 步骤 2：创建 SQL 数据库

请参阅[入门页](./sql-database-get-started.md)，以了解如何创建示例数据库。务必根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。

## 步骤 3：获取连接详细信息

[!INCLUDE [sql-database-include-connection-string-details-20-portalshots](../../includes/sql-database-include-connection-string-details-20-portalshots.md)]

## 步骤 4：运行示例代码

[“使用 Node.js 连接到 SQL”概念证明](https://msdn.microsoft.com/zh-cn/library/mt715784.aspx)

## 后续步骤

* 参阅 [SQL 数据库开发概述](./sql-database-develop-overview.md)
* 有关 [Microsoft Node.js Driver for SQL Server](https://msdn.microsoft.com/zh-cn/library/mt652093.aspx) 的详细信息

## 其他资源 

* [多租户 SaaS 应用程序和 Azure SQL 数据库的设计模式](./sql-database-design-patterns-multi-tenancy-saas-applications.md)
* 浏览所有 [SQL 数据库的功能](https://www.azure.cn/home/features/sql-database/)

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: wording update-->