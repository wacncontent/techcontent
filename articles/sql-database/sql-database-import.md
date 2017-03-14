---
title: 导入 BACPAC 文件以创建 Azure SQL 数据库 | Azure
description: 通过导入现有 BACPAC 文件创建 Azure SQL 数据库。
services: sql-database
documentationCenter: ''
authors: stevestein
manager: jhubbard
editor: ''

ms.service: sql-database
ms.devlang: NA
ms.date: 08/31/2016
wacn.date: 12/19/2016
ms.author: sstein
ms.workload: data-management
ms.topic: article
ms.tgt_pltfrm: NA
---

# 导入 BACPAC 文件以创建 Azure SQL 数据库

**单一数据库**

> [!div class="op_single_selector"]
- [Azure 门户预览](./sql-database-import.md)
- [PowerShell](./sql-database-import-powershell.md)
- [SSMS](./sql-database-cloud-migrate-compatible-import-bacpac-ssms.md)
- [SqlPackage](./sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage.md)

本文说明如何使用 [Azure 门户预览](https://portal.azure.cn)通过 BACPAC 文件创建 Azure SQL 数据库。

BACPAC 是包含数据库架构和数据的 .bacpac 文件。数据库通过从 Azure 存储 blob 容器导入的 BACPAC 创建。如果 Azure 存储中没有 .bacpac 文件，你可以按照[创建和导出 Azure SQL 数据库的 BACPAC](./sql-database-export.md) 中的步骤创建一个。

> [!NOTE]
> Azure SQL 数据库会自动为你可以还原的每个用户数据库创建和维护备份。有关详细信息，请参阅[业务连续性概述](./sql-database-business-continuity.md)。

若要从 .bacpac 导入 SQL 数据库，需要以下内容：

- Azure 订阅。
- Azure SQL 数据库 V12 服务器。如果没有 V12 服务器，可以按照本文中的以下步骤创建一个：[创建你的第一个 Azure SQL 数据库](./sql-database-get-started.md)。
- 要导入 [Azure 存储帐户（标准）](../storage/storage-create-storage-account.md)blob 容器中的数据库的 .bacpac 文件。

> [!IMPORTANT]
> 从 Azure Blob 存储导入 BACPAC 时，请使用标准存储。不支持从高级存储导入 BACPAC。

## 选择用于托管数据库的服务器

打开 SQL Server 边栏选项卡：

1. 转到 [Azure 门户预览](https://portal.azure.cn)。
2. 单击“SQL Server”。
3. 单击要将数据库还原到的服务器。
4. 在 SQL Server 边栏选项卡中，单击“导入数据库”以打开“导入数据库”边栏选项卡：

    ![导入数据库][1]

1.  单击“存储”并依次选择存储帐户、blob 容器和 .bacpac 文件，然后单击“确定”。

    ![配置存储选项][2]

1.  为新数据库选择定价层，然后单击“选择”。不支持直接将数据库导入弹性池，但可先导入单一数据库，然后将数据库移入池中。

    ![选择定价层][3]

1.  针对要从 BACPAC 文件创建的数据库输入**数据库名称**。
2.  选择身份验证类型，然后向服务器提供身份验证信息。
3.  单击“创建”以从 BACPAC 创建数据库。

    ![创建数据库][4]

单击“创建”会将导入数据库请求提交到服务。根据数据库的大小，导入操作可能需要一些时间才能完成。

## 监视导入操作的进度

1. 单击“SQL Server”。
2. 单击要还原到的服务器。
3. 在 SQL Server 边栏选项卡的操作区域中，单击“导入/导出历史记录”：

    ![导入导出历史记录][5] 
    ![导入导出历史记录][6]

## 验证数据库是否位于服务器上

1. 单击“SQL 数据库”并验证新数据库是否处于“联机”状态。

## 后续步骤

- 若要了解如何连接到并查询导入的 SQL 数据库，请参阅[使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](./sql-database-connect-query-ssms.md)

<!--Image references-->

[1]: ./media/sql-database-import/import-database.png
[2]: ./media/sql-database-import/storage-options.png
[3]: ./media/sql-database-import/pricing-tier.png
[4]: ./media/sql-database-import/create.png
[5]: ./media/sql-database-import/import-history.png
[6]: ./media/sql-database-import/import-status.png

<!---HONumber=Mooncake_Quality_Review_1202_2016-->