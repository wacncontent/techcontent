---
title: 从异地冗余备份还原 Azure SQL 数据库 (PowerShell) | Azure
description: 从异地冗余备份将 Azure SQL 数据库还原到新的服务器中
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
wacn.date: 09/19/2016
ms.author: sstein
---

# 使用 PowerShell 从异地冗余备份还原 Azure SQL 数据库

> [!div class="op_single_selector"]
- [概述](./sql-database-recovery-using-backups.md)
- [PowerShell](./sql-database-geo-restore-powershell.md)

本文演示了如何使用异地还原将数据库还原到新的服务器中。可通过 PowerShell 执行此操作。

[!INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]

## 将数据库异地还原到独立的数据库中

1. 使用 [Get-AzureRmSqlDatabaseGeoBackup](https://msdn.microsoft.com/zh-cn/library/azure/mt693388.aspx) cmdlet 获取要还原的数据库的异地冗余备份。

    ```
    $GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"
    ```

2. 借助 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390.aspx) cmdlet，从异地冗余备份中开始还原。

    ```
    Restore-AzureRmSqlDatabase –FromGeoBackup -ResourceGroupName "TargetResourceGroup" -ServerName "TargetServer" -TargetDatabaseName "RestoredDatabase" –ResourceId $GeoBackup.ResourceID -Edition "Standard" -RequestedServiceObjectiveName "S2"
    ```

## 将数据库异地还原到弹性数据库池中

1. 使用 [Get-AzureRmSqlDatabaseGeoBackup](https://msdn.microsoft.com/zh-cn/library/azure/mt693388.aspx) cmdlet 获取要还原的数据库的异地冗余备份。

    ```
    $GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"
    ```

2. 借助 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390.aspx) cmdlet，从异地冗余备份中开始还原。指定要将数据库还原到其中的池的名称。

    ```
    Restore-AzureRmSqlDatabase –FromGeoBackup -ResourceGroupName "TargetResourceGroup" -ServerName "TargetServer" -TargetDatabaseName "RestoredDatabase" –ResourceId $GeoBackup.ResourceID –ElasticPoolName "elasticpool01"  
    ```

## 后续步骤

- 有关业务连续性概述和应用场景，请参阅[业务连续性概述](./sql-database-business-continuity.md)。
- 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](./sql-database-automated-backups.md)。
- 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](./sql-database-recovery-using-backups.md)。
- 若要了解更快的恢复选项，请参阅[活动异地复制](./sql-database-geo-replication-overview.md)。
- 若要了解如何使用自动备份进行存档，请参阅[数据库复制](./sql-database-copy.md)。

<!---HONumber=Mooncake_0912_2016-->