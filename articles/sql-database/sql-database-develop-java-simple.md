---
title: 在 Windows 上配合使用 Java 和 JDBC 连接到 SQL 数据库 | Azure
description: 演示了一个可以用来连接到 Azure SQL 数据库的 Java 代码示例。该示例使用 JDBC，并在 Windows 客户端计算机上运行。
services: sql-database
documentationCenter: ''
authors: LuisBosquez
manager: jhubbard
editor: genemi

ms.service: sql-database
ms.workload: drivers
ms.tgt_pltfrm: na
ms.devlang: java
ms.topic: article
ms.date: 10/03/2016
wacn.date: 12/26/2016
ms.author: lbosq
---

# 在 Windows 上配合使用 Java 和 JDBC 连接到 SQL 数据库

[!INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本主题演示可用于连接到 Azure SQL 数据库的 Java 代码示例。该 Java 示例依赖于 Java 开发工具包 (JDK) 版本 1.8。该示例使用 JDBC 驱动程序连接到 Azure SQL 数据库。

## 步骤 1：配置开发环境

[配置用于 Java 开发的开发环境](https://msdn.microsoft.com/zh-cn/library/mt720658.aspx)

## 步骤 2：创建 SQL 数据库

请参阅[入门页](./sql-database-get-started.md)，以了解如何创建数据库。

## 步骤 3：获取连接字符串

[!INCLUDE [sql-database-include-connection-string-jdbc-20-portalshots](../../includes/sql-database-include-connection-string-jdbc-20-portalshots.md)]

> [!NOTE]
> 如果使用的是 JTDS JDBC 驱动程序，则需要将“ssl=require”添加到连接字符串的 URL，并需要设置以下 JVM 选项：“-Djsse.enableCBCProtection=false”。此 JVM 选项会禁用针对某个安全漏洞的修复程序，因此在设置此选项之前，请确保已了解涉及哪些风险。

## 步骤 4：运行示例代码

* [“使用 Java 连接到 SQL”概念证明](https://msdn.microsoft.com/zh-cn/library/mt720656.aspx)

## 后续步骤

* 访问 [Java 开发人员中心](/develop/java/)。
* 参阅 [SQL 数据库开发概述](./sql-database-develop-overview.md)
* 有关 [Microsoft JDBC Driver for SQL Server](https://msdn.microsoft.com/zh-cn/library/mt484311.aspx) 的详细信息

## 其他资源 

* [包含 Azure SQL 数据库的多租户 SaaS 应用程序的设计模式](./sql-database-design-patterns-multi-tenancy-saas-applications.md)
* 浏览所有 [SQL 数据库功能](https://www.azure.cn/home/features/sql-database/)

<!---HONumber=Mooncake_Quality_Review_1215_2016-->