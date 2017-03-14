---
title: SQL Server 数据库迁移到 SQL 数据库 | Azure
description: 了解如何将本地 SQL Server 数据库迁移到云中的 Azure SQL 数据库。在迁移数据库前，使用数据库迁移工具测试兼容性。
keywords: 数据库迁移, sql server 数据库迁移, 数据库迁移工具, 迁移数据库, 迁移 sql 数据库
services: sql-database
documentationcenter: ''
author: CarlRabeler
manager: jhubbard
editor: ''

ms.assetid: 9cf09000-87fc-4589-8543-a89175151bc2
ms.service: sql-database
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: sqldb-migrate
ms.date: 11/08/2016
wacn.date: 12/19/2016
ms.author: carlrab
---

# 将 SQL Server 数据库迁移到云中的 SQL 数据库

本文介绍如何将本地 SQL Server 2005 或更高版本的数据库迁移到 Azure SQL 数据库。在此数据库迁移过程中，需从当前环境中的 SQL Server 数据库将架构和数据迁移到 SQL 数据库。为了成功，必须确保现有数据库通过兼容性测试。使用 [SQL 数据库 V12](./sql-database-v12-whats-new.md) 时，除了服务器级别操作和跨数据库操作的相关问题，还有一些兼容性问题。依赖[部分支持或不受支持的函数](./sql-database-transact-sql-information.md)的数据库和应用程序需要进行重新设计来修复这些不兼容性，然后才能迁移 SQL Server 数据库。

若要迁移，需执行以下步骤：

- **兼容性测试**：验证数据库是否与 [SQL 数据库 V12](./sql-database-v12-whats-new.md) 兼容。
- **解决兼容性问题（如果有）**：如果验证失败，必须先解决验证错误。
- **运行迁移**：如果数据库兼容，则可使用一种或多种方法来执行迁移。

SQL Server 提供多种方法来完成每个任务。本文概述了每个任务的可用方法。下图演示了步骤和方法。

  ![VSSSDT 迁移示意图](./media/sql-database-cloud-migrate/03VSSSDTDiagram.png)

 > [!NOTE]
 > 若要将非 SQL Server 数据库（包括 Microsoft Access、Sybase、MySQL Oracle 和 DB2）迁移到 Azure SQL 数据库，请参阅 [SQL Server 迁移助手](http://blogs.msdn.com/b/ssma)。

## 数据库迁移工具将测试 SQL Server 数据库与 SQL 数据库的兼容性
开始数据库迁移过程之前，请使用以下方法之一测试 SQL 数据库的兼容性问题：

> [!div class="op_single_selector"]
- [SSDT](./sql-database-cloud-migrate-fix-compatibility-issues-ssdt.md)
- [SqlPackage](./sql-database-cloud-migrate-determine-compatibility-sqlpackage.md)
- [SSMS](./sql-database-cloud-migrate-determine-compatibility-ssms.md)
- [SAMW](./sql-database-cloud-migrate-fix-compatibility-issues.md)

* [SQL Server Data Tools for Visual Studio (SSDT)](./sql-database-cloud-migrate-fix-compatibility-issues-ssdt.md)：SSDT 使用最新的兼容性规则来检测 SQL 数据库 V12 不兼容性。如果检测到不兼容，则可以直接在此工具中解决检测到的问题。此方法是目前用于测试和解决 SQL 数据库 V12 兼容性问题的建议方法。
* [SqlPackage](./sql-database-cloud-migrate-determine-compatibility-sqlpackage.md)：SqlPackage 是一个命令行实用程序，用于测试兼容性问题，并生成包含检测到的兼容性问题的报告。如果使用此工具，请确保使用最新版本，以便使用最新的兼容性规则。如果检测到错误，必须使用其他工具来解决任何检测到的兼容性问题 - 建议使用 SSDT。
* [SQL Server Management Studio 中的“导出数据层”应用程序向导](./sql-database-cloud-migrate-determine-compatibility-ssms.md)：此向导可以检测错误并在屏幕上报告错误。如果未检测到错误，则可以继续迁移到 SQL 数据库。如果检测到错误，必须使用其他工具来解决任何检测到的兼容性问题 - 建议使用 SSDT。
* [SQL Azure Migration Wizard ("SAMW")](./sql-database-cloud-migrate-fix-compatibility-issues.md)（SQL Azure 迁移向导 (SAMW)）：SAMW 是一个 codeplex 工具，使用 Azure SQL 数据库 V11 兼容性规则来检测 Azure SQL 数据库 V12 的不兼容性。如果检测到不兼容，某些问题可直接在此工具中解决。此工具可以发现无需修复的不兼容性。该工具是第一个可以使用的 Azure SQL 数据库迁移协助工具，受到 SQL Server 社区的积极支持。此外，此工具可在工具本身内部完成迁移。

##<a name="fix-database-migration-compatibility-issues"></a> 修复数据库迁移的兼容性问题

如果检测到兼容性问题，必须先修复这些兼容性问题，才能继续 SQL Server 数据库迁移。根据源数据库中的 SQL Server 版本以及正在迁移的数据库复杂性，可能会发现各种不同的不兼容性问题。旧版 SQL Server 的兼容性问题更多。除了使用所选搜索引擎的目标 Internet 搜索以外，还可以使用以下资源：

- [Azure SQL 数据库中不支持的 SQL Server 数据库功能](./sql-database-transact-sql-information.md)
- [SQL Server 2016 中已停用的数据库引擎功能](https://msdn.microsoft.com/zh-cn/library/ms144262%28v=sql.130%29)
- [SQL Server 2014 中已停用的数据库引擎功能](https://msdn.microsoft.com/zh-cn/library/ms144262%28v=sql.120%29)
- [SQL Server 2012 中已停用的数据库引擎功能](https://msdn.microsoft.com/zh-cn/library/ms144262%28v=sql.110%29)
- [SQL Server 2008 R2 中已停用的数据库引擎功能](https://msdn.microsoft.com/zh-cn/library/ms144262%28v=sql.105%29)
- [SQL Server 2005 中已停用的数据库引擎功能](https://msdn.microsoft.com/zh-cn/library/ms144262%28v=sql.90%29)

除了搜索 Internet 和使用这些资源，还可以使用 [MSDN SQL Server 社区论坛](https://social.msdn.microsoft.com/Forums/sqlserver/home?category=sqlserver)或 [StackOverflow](http://stackoverflow.com/)。

使用以下数据库迁移工具之一解决检测到的问题：

> [!div class="op_single_selector"]
- [SSDT](./sql-database-cloud-migrate-fix-compatibility-issues-ssdt.md)
- [SSMS](./sql-database-cloud-migrate-fix-compatibility-issues-ssms.md)
- [SAMW](./sql-database-cloud-migrate-fix-compatibility-issues.md)

- 使用 [SQL Server Data Tools for Visual Studio (SSDT)](./sql-database-cloud-migrate-fix-compatibility-issues-ssdt.md)：若要使用 SSDT，请将数据库架构导入 SQL Server Data Tools for Visual Studio (SSDT)，然后生成进行 SQL 数据库 V12 部署的项目。然后，在 SSDT 中修复所有检测到的兼容性问题。完成后，将所做的更改同步回源数据库或源数据库的副本。SSDT 是目前用于测试和解决 SQL 数据库 V12 兼容性问题的建议方法。请单击[使用 SSDT 进行演练](./sql-database-cloud-migrate-fix-compatibility-issues-ssdt.md)的链接。
- 使用 [SQL Server Management Studio (SSMS)](./sql-database-cloud-migrate-fix-compatibility-issues-ssms.md)：若要使用 SSMS，可以执行 Transact-SQL 命令，修复使用其他工具检测到的错误。此方法主要供高级用户直接在源数据库中修改数据库架构。
- 使用 [SQL Azure 迁移向导 (SAMW)](./sql-database-cloud-migrate-fix-compatibility-issues.md)：若要使用 SAMW，请从源数据库生成 Transact-SQL 脚本。该向导会尽可能转换脚本，使架构兼容 SQL 数据库 V12。完成后，SAMW 可以连接到 SQL 数据库 V12 以执行脚本。此工具还可分析跟踪文件，确定兼容性问题。生成的脚本可以只包含架构，也可以包含 BCP 格式的数据。

##<a name="migrate-a-compatible-sql-server-database-to-sql-database"></a> 将兼容的 SQL Server 数据库迁移到 SQL 数据库

为了迁移兼容的 SQL Server 数据库，Microsoft 针对各种方案提供了多个迁移方法。所选择的方法取决于对停机时间的容忍程度、对 SQL Server 数据库的大小和复杂性要求，以及与 Azure 云的连接。

> [!div class="op_single_selector"]
- [SSMS 迁移向导](./sql-database-cloud-migrate-compatible-using-ssms-migration-wizard.md)
- [导出到 BACPAC 文件](./sql-database-cloud-migrate-compatible-export-bacpac-ssms.md)
- [从 BACPAC 文件导入](./sql-database-cloud-migrate-compatible-import-bacpac-ssms.md)
- [事务复制](./sql-database-cloud-migrate-compatible-using-transactional-replication.md)

若要选择迁移方法，需要了解的第一个问题是，是否允许数据库在迁移过程中脱离生产环境。发生活动事务时迁移数据库可能会导致数据库不一致，并且数据库可能会损坏。有许多方法可以使数据库处于静默状态，例如禁用客户端连接以创建[数据库快照](https://msdn.microsoft.com/zh-cn/library/ms175876.aspx)。

若要使迁移时停机时间最短，请使用 [SQL Server 事务复制](./sql-database-cloud-migrate-compatible-using-transactional-replication.md)（如果数据库满足事务复制的要求）。如果可以承受一定的停机时间，或者正在针对以后的迁移执行生产数据库的测试迁移，请考虑以下三种方法之一：

- [SSMS 迁移向导](./sql-database-cloud-migrate-compatible-using-ssms-migration-wizard.md)：对于小型到中型数据库，迁移兼容的 SQL Server 2005 或更高版本数据库就像在 SQL Server Management Studio 中运行[将数据库部署到 Azure 数据库](./sql-database-cloud-migrate-compatible-using-ssms-migration-wizard.md)向导一样简单。
- [导出到 BACPAC 文件](./sql-database-cloud-migrate-compatible-export-bacpac-ssms.md)，然后[从 BACPAC 文件导入](./sql-database-cloud-migrate-compatible-import-bacpac-ssms.md)：如果遇到连接问题（无连接、低带宽或超时问题）并且对于中型到大型数据库，请使用 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件。可以使用此方法将 SQL Server 架构和数据导出到 BACPAC 文件。然后，可以通过 SQL Server Management Studio 中的导出数据层应用程序向导或 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令提示实用工具，将 BACPAC 文件导入 SQL 数据库。
- 将 BACPAC 和 BCP 一起使用：对于非常大的数据库使用 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件和 [BCP](https://msdn.microsoft.com/zh-cn/library/ms162802.aspx) 可实现更高的并行化以提高性能，虽然会具有更高的复杂性。使用此方法，分别迁移架构和数据。
 - [仅将架构导出到 BACPAC 文件](./sql-database-cloud-migrate-compatible-export-bacpac-ssms.md)。
 - [仅从 BACPAC 文件中将架构导入](./sql-database-cloud-migrate-compatible-import-bacpac-ssms.md)到 SQL 数据库。
 - 使用 [BCP](https://msdn.microsoft.com/zh-cn/library/ms162802.aspx) 将数据提取到平面文件中，然后再将这些文件[并行加载](https://technet.microsoft.com/zh-cn/library/dd425070.aspx)到 Azure SQL 数据库。

     ![SQL Server 数据库迁移 - 将 SQL 数据库迁移到云。](./media/sql-database-cloud-migrate/01SSMSDiagram_new.png)  

## 后续步骤
- [最新版本的 SSDT](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)
- [最新版本的 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)

##其他资源

- [SQL 数据库 V12](./sql-database-v12-whats-new.md)
- [Transact-SQL 部分支持或不支持的函数](./sql-database-transact-sql-information.md)
- [使用 SQL Server 迁移助手迁移非 SQL Server 数据库](http://blogs.msdn.com/b/ssma/)

<!---HONumber=Mooncake_1212_2016-->