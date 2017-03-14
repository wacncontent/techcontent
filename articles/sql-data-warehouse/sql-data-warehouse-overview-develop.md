---
title: SQL 数据仓库开发的设计决策和编码技术 | Azure
description: SQL 数据仓库的开发概念、设计决策、建议和编程技术。
services: sql-data-warehouse
documentationCenter: NA
authors: jrowlandjones
manager: barbkess
editor: ''

ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.date: 10/31/2016
wacn.date: 12/19/2016
ms.author: jrj;barbkess;sonyama
---

# SQL 数据仓库的设计决策和编程技术
请阅读以下开发文章，以更好地了解 SQL 数据仓库的关键设计决策、建议和编程技术。

## 关键设计决策
以下文章重点介绍了在使用 SQL 数据仓库开发分布式数据仓库时，需要了解的一些关键概念和设计决策：

- [连接][]
- [并发][]
- [事务][]
- [用户定义的架构][]
- [表分布][]
- [表索引][]
- [表分区][]
- [CTAS][]
- [统计信息][]

## 开发建议和编程技术
以下文章重点介绍了用于开发 SQL 数据仓库的具体编程技术、技巧和建议：

- [存储过程][]
- [标签][]
- [视图][]
- [临时表][]
- [动态 SQL][]
- [循环][]
- [Group By 选项][]
- [变量赋值][]

## 后续步骤
阅读开发文章后，请浏览 [Transact-SQL 参考][]页，以了解有关 SQL 数据仓库支持的语法的更多详细信息。

<!--Image references-->

<!--Article references-->
[并发]: ./sql-data-warehouse-develop-concurrency.md
[连接]: ./sql-data-warehouse-connect-overview.md
[CTAS]: ./sql-data-warehouse-develop-ctas.md
[动态 SQL]: ./sql-data-warehouse-develop-dynamic-sql.md
[Group By 选项]: ./sql-data-warehouse-develop-group-by-options.md
[标签]: ./sql-data-warehouse-develop-label.md
[循环]: ./sql-data-warehouse-develop-loops.md
[统计信息]: ./sql-data-warehouse-tables-statistics.md
[存储过程]: ./sql-data-warehouse-reference-tsql-statements.md
[表分布]: ./sql-data-warehouse-tables-statistics.md
[表索引]: ./sql-data-warehouse-tables-overview.md
[表分区]: ./sql-data-warehouse-tables-partition.md
[临时表]: ./sql-data-warehouse-tables-temporary.md
[事务]: ./sql-data-warehouse-develop-transactions.md
[用户定义的架构]: ./sql-data-warehouse-develop-user-defined-schemas.md
[变量赋值]: ./sql-data-warehouse-develop-variable-assignment.md
[视图]: ./sql-data-warehouse-develop-views.md
[Transact-SQL 参考]: ./sql-data-warehouse-overview-reference.md

<!--MSDN references-->
[renaming objects]: https://msdn.microsoft.com/zh-cn/library/mt631611.aspx

<!--Other Web references-->

<!---HONumber=Mooncake_1212_2016-->