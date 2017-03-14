---
title: SQL 数据库性能：服务层 | Azure
description: 比较 SQL 数据库服务层。
keywords: 数据库选项,数据库性能
services: sql-database
documentationcenter: ''
author: CarlRabeler
manager: jhubbard
editor: ''

ms.assetid: f5c5c596-cd1e-451f-92a7-b70d4916e974
ms.service: sql-database
ms.custom: overview
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: data-management
ms.date: 01/11/2017
wacn.date: 02/14/2017
ms.author: carlrab; janeng
---

# SQL 数据库选项和性能：了解每个服务层提供的功能
[Azure SQL 数据库](./sql-database-technical-overview.md)提供了三个具有多个性能级别的服务层（**基本**、**标准**和**高级**），用于处理不同的工作负荷。更高的性能级别提供不断增加的资源，旨在递增式提供更高的吞吐量。无需停机[即可动态更改服务层和性能级别](./sql-database-scale-up.md)。基本、标准和高级服务层都提供 99.99% 的运行时间 SLA、灵活的业务连续性选项、安全功能和按小时计费。

可以在所选[性能级别](./sql-database-service-tiers.md#single-database-service-tiers-and-performance-levels)上使用专用资源创建单一数据库。还可以在资源在数据库之间共享的[弹性池](./sql-database-service-tiers.md#elastic-pool-service-tiers-and-performance-in-edtus)中管理多个数据库。可用于单一数据库的资源以数据库事务单位 (DTU) 表示，可用于弹性池的资源则用弹性 DTU (eDTU) 表示。有关 DTU 和 eDTU 的详细信息，请参阅[什么是 DTU？](./sql-database-what-is-a-dtu.md)。

在这两种情况下，服务层包括“基本”、“标准”和“高级”。

## 选择服务层
下表提供了最适用于不同应用程序工作负荷的层的示例。

| 服务层 | 目标工作负荷 |
| :--- | --- |
| **基本** | 最适合小型数据库，通常支持在给定时间执行一个活动操作。示例包括用于开发或测试的数据库，或不常使用的小型应用程序。 |
| **标准** |云应用程序的转到选项，提供低到中等的 IO 性能要求，并支持多个并发查询。示例包括工作组或 Web 应用程序。 |
| **高级** | 专为高事务量设计，提供高 IO 性能要求，并支持许多并发用户。示例包括支持任务关键型应用程序的数据库。 |

首先确定是要运行单一数据库，还是要运行共享资源的组数据库。查看[弹性池注意事项](./sql-database-elastic-pool-guidance.md)。若要确定服务层，首先确定所需的最小数据库功能：

* 单个数据库的最大数据库大小（在高端性能级别，基本数据库最大 2 GB、标准数据库最大 250 GB、高级数据库 500 GB 到 1 TB）
* 弹性池的最大总存储（基本池为 117 GB，标准池为 1200 GB，高级池为 750 GB）
* 每个池的最大数据库数（基本池 400 个，标准池 400 个，高级池 50 个）
* 数据库备份保留期（基本数据库 7 天、标准数据库和高级数据库 35 天）

确定最低服务层后，就可以确定数据库的性能级别（DTU 数）。标准 S2 和 S3 性能级别在许多情况下是好的起点。对于具有较高 CPU 或 IO 要求的数据库，高级性能级别是合适的起点。与最高的标准性能级别相比，高级性能级别提供更多 CPU 和至少高 10 倍的 IO。

初始选取性能级别后，可以根据实际经验动态增加或减少[单个数据库](./sql-database-scale-up.md)或[弹性池](./sql-database-elastic-pool-manage-portal.md#change-performance-settings-of-a-pool)。对于迁移方案，还可以使用 [DTU 计算器](http://dtucalculator.azurewebsites.net/)估计所需的 DTU 数。

##<a name="standalone-database-service-tiers-and-performance-levels"></a> 单一数据库服务层和性能级别
对于单一数据库，每个服务层内都具有多个性能级别。你可以灵活选择最能满足你的工作负荷需求的级别。如果你需要增加或减少工作负荷，可以轻松更改数据库层。有关详细信息，请参阅[更改数据库服务层和性能级别](./sql-database-scale-up.md)。

尽管有多个托管的数据库，你的数据库仍可确保获得一组资源，并且数据库的预期性能特征不受影响。

[!INCLUDE [SQL 数据库服务层表](../../includes/sql-database-service-tiers-table.md)]

>[!NOTE]
> 如需本服务层表中所有其他行的详细说明，请参阅[服务层功能和限制](./sql-database-performance-guidance.md#service-tier-capabilities-and-limits)。

##<a name="elastic-pool-service-tiers-and-performance-in-edtus"></a><a name="elastic-database-pool-service-tiers-and-performance-in-edtus"></a><a name="single-database-service-tiers-and-performance-levels"></a> 弹性池服务层和性能 (eDTU)

池允许数据库共享和使用 eDTU 资源，而无需为该池中的每个数据库分配特定性能级别。例如，标准池中的单一数据库可使用 0 个 eDTU 到最大数据库 eDTU 数（配置池时设置的）运转。弹性池允许多个具有不同工作负荷的数据库有效地使用在整个池中都可用的 eDTU 资源。有关详细信息，请参阅[弹性池的价格和性能注意事项](./sql-database-elastic-pool-guidance.md)。

下表描述了池服务层的特征。

[!INCLUDE [用于弹性池的 SQL 数据库服务层表](../../includes/sql-database-service-tiers-table-elastic-db-pools.md)]

池中的每个数据库也遵循该层的单一数据库特征。例如，基本池具有每池最大会话数为 4800 - 28800 的限制，但该基本池中单个数据库具有 300 个会话数的数据库限制。

## 后续步骤

* 了解有关[弹性池](./sql-database-elastic-pool-guidance.md)和[弹性池的价格和性能注意事项](./sql-database-elastic-pool-guidance.md)的详细信息。
* 了解如何[监视、管理弹性池和调整其大小](./sql-database-elastic-pool-manage-portal.md)以及如何[监视单个数据库的性能](./sql-database-single-database-monitor.md)。
* 现在，你已了解有关 SQL 数据库层的信息，可使用 [1 元帐户](https://www.azure.cn/pricing/1rmb-trial/)来试用这些层，并了解[如何创建你的第一个 SQL 数据库](./sql-database-get-started.md)。

<!---HONumber=Mooncake_0120_2017-->
<!--update: add sub title "单一数据库服务层和性能级别"-->