---
title: 将 Azure SQL 数据库还原到之前的时间点 (PowerShell) | Azure
description: 将 Azure SQL 数据库还原到之前的时间点
services: sql-database
documentationCenter: ''
authors: stevestein
manager: jhubbard
editor: ''

ms.service: sql-database
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: powershell
ms.workload: NA
ms.date: 07/17/2016
wacn.date: 12/26/2016
ms.author: sstein
---

# 使用 PowerShell 将 Azure SQL 数据库还原到之前的时间点

> [!div class="op_single_selector"]
- [概述](./sql-database-recovery-using-backups.md)
- [时间点还原：Azure 门户预览](./sql-database-point-in-time-restore-portal.md)

本文介绍如何将数据库从 [SQL 数据库自动备份](./sql-database-automated-backups.md)还原到以前的时间点。可使用 PowerShell 实现此目的。

[!INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]

## 将数据库还原到作为独立数据库的某个时间点

1. 使用 [Get-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt603648.aspx) cmdlet 获取要还原的数据库。

    ```
    $Database = Get-AzureRmSqlDatabase -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"
    ```

2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390.aspx) cmdlet 将数据库还原到某个时间点。

    ```
    Restore-AzureRmSqlDatabase –FromPointInTimeBackup –PointInTime UTCDateTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "RestoredDatabase" –ResourceId $Database.ResourceID -Edition "Standard" -ServiceObjectiveName "S2"
    ```

## 将数据库还原到某个时间点的弹性数据库池中

1. 使用 [Get-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt603648.aspx) cmdlet 获取要还原的数据库。

    ```
    $Database = Get-AzureRmSqlDatabase -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"
    ```

2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390.aspx) cmdlet 将数据库还原到某个时间点。

    ```
    Restore-AzureRmSqlDatabase –FromPointInTimeBackup –PointInTime UTCDateTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "RestoredDatabase" –ResourceId $Database.ResourceID –ElasticPoolName "elasticpool01"
    ```

## 后续步骤

- 有关业务连续性的概述和应用场景，请参阅[业务连续性概述](./sql-database-business-continuity.md)
- 若要了解 Azure SQL 数据库自动备份，请参阅 [SQL 数据库自动备份](./sql-database-automated-backups.md)
- 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](./sql-database-recovery-using-backups.md)
- 若要了解更快的恢复选项，请参阅[活动异地复制](./sql-database-geo-replication-overview.md)
- 若要了解如何使用自动备份进行存档，请参阅[数据库复制](./sql-database-copy.md)

<!---HONumber=Mooncake_Quality_Review_1215_2016-->