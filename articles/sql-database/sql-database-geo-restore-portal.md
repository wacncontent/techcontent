---
title: 从自动备份还原 Azure SQL 数据库（Azure 门户） | Azure
description: 从自动备份还原 Azure SQL 数据库（Azure 门户）。
services: sql-database
documentationcenter: ''
author: stevestein
manager: jhubbard
editor: ''

ms.assetid: 62d94085-d7e7-4f08-bb83-404cf866a6c1
ms.service: sql-database
ms.devlang: NA
ms.date: 10/18/2016
wacn.date: 12/19/2016
ms.author: sstein
ms.workload: NA
ms.topic: article
ms.tgt_pltfrm: NA
---

# 使用 Azure 门户预览从异地冗余备份异地还原 Azure SQL 数据库

> [!div class="op_single_selector"]
- [概述](./sql-database-recovery-using-backups.md#geo-restore)
- [异地还原：PowerShell](./sql-database-geo-restore-powershell.md)

本文演示如何使用 Azure 门户预览通过[异地还原](./sql-database-recovery-using-backups.md#geo-restore)从[自动备份](./sql-database-automated-backups.md)将数据库还原到新服务器中。

## 选择要还原的数据库
若要在 Azure 门户预览中还原数据库，请执行以下步骤：

1. 转到 [Azure 门户预览](https://portal.azure.cn)。
2. 在屏幕左侧选择“+新建”>“数据库”>“SQL 数据库”：

   ![还原 Azure SQL 数据库](./media/sql-database-geo-restore-portal/new-sql-database.png)  

3. 选择“备份”作为源，然后选择要还原的备份。指定数据库名称、要将数据库还原到其中的服务器，然后单击“创建”：

   ![还原 Azure SQL 数据库](./media/sql-database-geo-restore-portal/geo-restore.png)  

单击页面右上方的通知图标，监视还原操作的状态。

## 后续步骤
- 有关业务连续性概述和应用场景，请参阅[业务连续性概述](./sql-database-business-continuity.md)
- 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](./sql-database-automated-backups.md)
- 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](./sql-database-recovery-using-backups.md)
- 若要了解更快的恢复选项，请参阅[活动异地复制](./sql-database-geo-replication-overview.md)
- 若要了解如何使用自动备份进行存档，请参阅[数据库复制](./sql-database-copy.md)

<!---HONumber=Mooncake_1212_2016-->