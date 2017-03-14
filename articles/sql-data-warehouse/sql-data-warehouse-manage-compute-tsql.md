---
title: 管理 Azure SQL 数据仓库中的计算能力 (REST) | Azure
description: 可通过调整 DWU 来提高性能的 Transact-SQL (T-SQL) 任务。通过在非高峰期缩减性能来节省成本。
services: sql-data-warehouse
documentationCenter: NA
authors: barbkess
manager: barbkess
editor: ''

ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.date: 10/31/2016
wacn.date: 12/19/2016
ms.author: barbkess;sonyama
---

# 管理 Azure SQL 数据仓库中的计算能力 (T-SQL)

> [!div class="op_single_selector"]
- [概述](./sql-data-warehouse-manage-compute-overview.md)
- [门户](./sql-data-warehouse-manage-compute-portal.md)
- [PowerShell](./sql-data-warehouse-manage-compute-powershell.md)
- [REST](./sql-data-warehouse-manage-compute-rest-api.md)
- [TSQL](./sql-data-warehouse-manage-compute-tsql.md)

通过扩大计算资源和内存来提升性能，从而满足工作负荷不断变化的需求。通过在非高峰时段缩减资源或同时暂停计算来节省成本。

此任务集合使用 T-SQL 门户实现：

- 查看当前的 DWU 设置
- 通过调整 DWU 更改计算资源

若要暂停或恢复某个数据库，请在本文顶部选择其他平台选项之一。

若要了解相关信息，请参阅[管理计算能力概述][Manage compute power overview]。

<a name="current-dwu-bk"></a>

## 查看当前的 DWU 设置
若要查看数据库的当前 DWU 设置，请执行以下操作：

1. 在 Visual Studio 2015 中打开“SQL Server 对象资源管理器”。
2. 连接到与逻辑 SQL 数据库服务器关联的 master 数据库。
3. 从 sys.database\_service\_objectives 动态管理视图中选择。下面是一个示例：

    ```
    SELECT
     db.name [Database],
     ds.edition [Edition],
     ds.service_objective [Service Objective]
    FROM
     sys.database_service_objectives ds
     JOIN sys.databases db ON ds.database_id = db.database_id
    ```

<a name="scale-dwu-bk"></a>
<a name="scale-compute-bk"></a>

## 缩放计算
[!INCLUDE [SQL Data Warehouse scale DWUs description（SQL 数据仓库缩放 DWU 说明）](../../includes/sql-data-warehouse-scale-dwus-description.md)]

若要更改 DWU，请执行以下操作：

1. 连接到与逻辑 SQL 数据库服务器关联的 master 数据库。
2. 使用 [ALTER DATABASE][ALTER DATABASE] TSQL 语句。以下示例将数据库 MySQLDW 的服务级别目标设置为 DW1000。

    ```
    ALTER DATABASE MySQLDW
    MODIFY (SERVICE_OBJECTIVE = 'DW1000')
    ;
    ```

<a name="next-steps-bk"></a>

## 后续步骤
有关其他管理任务，请参阅[管理概述][Management overview]。

<!--Image references-->

<!--Article references-->
[Service capacity limits]: ./sql-data-warehouse-service-capacity-limits.md
[Management overview]: ./sql-data-warehouse-overview-manage.md
[Manage compute power overview]: ./sql-data-warehouse-manage-compute-overview.md

<!--MSDN references-->

[ALTER DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204042.aspx

<!--Other Web references-->

[Azure portal]: http://portal.azure.cn/

<!---HONumber=Mooncake_1212_2016-->