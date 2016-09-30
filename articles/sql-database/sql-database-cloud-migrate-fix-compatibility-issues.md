<properties
   pageTitle="在迁移到 SQL 数据库之前，修复 SQL Server 数据库兼容性问题"
   description="Azure SQL 数据库, 数据库迁移, 兼容性, SQL Azure 迁移向导"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="06/07/2016"
   wacn.date="07/11/2016"/>

# 在迁移到 Azure SQL 数据库之前使用 SQL Azure 迁移向导解决 SQL Server 数据库兼容性问题

> [AZURE.SELECTOR]
- 使用 [SQL Azure 迁移向导](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues/)
- 使用 [SSDT](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/)
- 使用 [SSMS](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssms/)

在本文中，你将了解在迁移到 Azure SQL 数据库之前如何使用 SQL Azure 迁移向导检测并解决 SQL Server 数据库的兼容性问题。

## 使用 SQL Azure 迁移向导

可以使用 [SQL Azure 迁移向导](http://sqlazuremw.codeplex.com) CodePlex 工具从不兼容的源数据库生成 T-SQL 脚本，向导随后将转换该脚本使其与 SQL 数据库兼容，然后连接到 Azure SQL 数据库以执行该脚本。此工具还将分析跟踪文件以确定兼容性问题。生成的脚本可以只包含架构，也可以包含 BCP 格式的数据。其他文档（包括分步指南）可在 CodePlex 上的 [SQL Azure 迁移向导](http://sqlazuremw.codeplex.com)中找到。

 ![SAMW 迁移示意图](./media/sql-database-cloud-migrate/02SAMWDiagram.png)

  > [AZURE.NOTE] 请注意，向导的内置转换并非能够修复它可检测到的所有不兼容的架构。无法解决的非兼容脚本将报告为错误，将在生成的脚本中注入注释。如果检测到许多错误，请使用 Visual Studio 或 SQL Server Management Studio 来单步执行并修复无法使用 SQL Server 迁移向导修复的每个错误。

## 后续步骤

- [最新版本的 SSDT](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)
- [最新版本的 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)
- [将兼容的 SQL Server 数据库迁移到 SQL 数据库](/documentation/articles/sql-database-cloud-migrate/#migrate-a-compatible-sql-server-database-to-sql-database)

## 其他资源

- [SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new/)
- [Transact-SQL 部分支持或不支持的函数](/documentation/articles/sql-database-transact-sql-information/)
- [使用 SQL Server 迁移助手迁移非 SQL Server 数据库](http://blogs.msdn.com/b/ssma/)

<!---HONumber=Mooncake_0704_2016-->