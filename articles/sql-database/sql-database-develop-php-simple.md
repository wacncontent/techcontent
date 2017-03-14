---
title: 在 Windows上使用 PHP 连接到 SQL 数据库 | Azure
description: 演示了一个示例 PHP 程序，该程序可以从 Windows 客户端连接到 Azure SQL 数据库，并与客户端所需的软件组件建立链接。
services: sql-database
documentationCenter: ''
authors: meet-bhagdev
manager: jhubbard
editor: ''

ms.service: sql-database
ms.workload: drivers
ms.tgt_pltfrm: na
ms.devlang: php
ms.topic: article
ms.date: 10/03/2016
wacn.date: 12/26/2016
ms.author: meetb
---

# 在 Windows上使用 PHP 连接到 SQL 数据库

[!INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本主题演示了如何从 Windows 上运行的、以 PHP 编写的客户端应用程序连接到 Azure SQL 数据库。

## 步骤 1：配置开发环境

[配置用于 PHP 开发的开发环境](https://msdn.microsoft.com/zh-cn/library/mt720663.aspx)

## 步骤 2：创建 SQL 数据库

请参阅[入门页](./sql-database-get-started.md)，以了解如何创建示例数据库。务必根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。

## 步骤 3：获取连接详细信息

[!INCLUDE [sql-database-include-connection-string-details-20-portalshots](../../includes/sql-database-include-connection-string-details-20-portalshots.md)]

## 步骤 4：运行示例代码

* [“使用 PHP 连接到 SQL”概念证明](https://msdn.microsoft.com/zh-cn/library/mt720665.aspx)
* [使用 PHP 弹性连接到 SQL](https://msdn.microsoft.com/zh-cn/library/mt720667.aspx)

## 后续步骤

* 参阅 [SQL 数据库开发概述](./sql-database-develop-overview.md)
* 有关 [Microsoft PHP Driver for SQL Server](https://msdn.microsoft.com/zh-cn/library/dn865013.aspx) 的详细信息
* 有关 PHP 安装和用法的详细信息，请参阅[使用 PHP 访问 SQL Server 数据库](http://social.technet.microsoft.com/wiki/contents/articles/1258.accessing-sql-server-databases-from-php.aspx)。

## 其他资源 

* [包含 Azure SQL 数据库的多租户 SaaS 应用程序的设计模式](./sql-database-design-patterns-multi-tenancy-saas-applications.md)
* 浏览所有 [SQL 数据库功能](https://www.azure.cn/home/features/sql-database/)

<!---HONumber=Mooncake_Quality_Review_1215_2016-->