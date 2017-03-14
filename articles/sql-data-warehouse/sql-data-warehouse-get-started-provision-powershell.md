---
title: 使用 PowerShell 创建 SQL 数据仓库 | Azure
description: 使用 PowerShell 创建 SQL 数据仓库
services: sql-data-warehouse
documentationCenter: NA
authors: lodipalm
manager: barbkess
editor: ''

ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: get-started-article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.date: 10/31/2016
wacn.date: 01/03/2017
ms.author: lodipalm;barbkess;sonyama
---

# 使用 PowerShell 创建 SQL 数据仓库

> [!div class="op_single_selector"]
- [Azure 门户](./sql-data-warehouse-get-started-provision.md)
- [TSQL](./sql-data-warehouse-get-started-create-database-tsql.md)
- [PowerShell](./sql-data-warehouse-get-started-provision-powershell.md)

本文说明如何使用 PowerShell 创建 SQL 数据仓库。

## 先决条件
若要开始，您需要：

- **Azure 帐户**：访问 [Azure 试用版][]，以创建帐户。
- **Azure SQL Server**：有关详细信息，请参阅[使用 Azure 门户创建 Azure SQL 数据库逻辑服务器][Create an Azure SQL Database logical server with the Azure Portal]或[使用 PowerShell 创建 Azure SQL 数据库逻辑服务器][Create an Azure SQL Database logical server with PowerShell]。
- **资源组**：可使用同一资源组作为 Azure SQL Server，或参阅[如何创建资源组][how to create a resource group]。
- **PowerShell 1.0.3 或更高版本**：可以通过运行 **Get-Module -ListAvailable -Name Azure** 来检查版本。可以通过 [Microsoft Web 平台安装程序][Microsoft Web Platform Installer]安装最新版本。有关安装最新版本的详细信息，请参阅[如何安装和配置 Azure PowerShell][How to install and configure Azure PowerShell]。

> [!NOTE]
> 创建 SQL 数据仓库可能会导致新的计费服务。有关定价的详细信息，请参阅 [SQL 数据仓库定价][]。

## 创建 SQL 数据仓库
1. 打开 Windows PowerShell。
2. 运行此 cmdlet 以登录到 Azure Resource Manager 中。

    ```
    Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    ```

3. 选择要用于当前会话的订阅。

    ```
    Get-AzureRmSubscription	-SubscriptionName "MySubscription" | Select-AzureRmSubscription
    ```

4.  创建数据库。此示例将在名为“mywesteuroperesgp1”的资源组中的名为“sqldwserver1”的服务器中创建一个名为“mynewsqldw”且服务目标级别为“DW400”的新数据库。**注意：创建新的 SQL 数据仓库数据库可能会导致新的计费费用。有关定价的详细信息，请参阅 [SQL 数据仓库定价][]。**

    ```
    New-AzureRmSqlDatabase -RequestedServiceObjectiveName "DW400" -DatabaseName "mynewsqldw" -ServerName "sqldwserver1" -ResourceGroupName "mywesteuroperesgp1" -Edition "DataWarehouse"
    ```

所需的参数包括：

* **RequestedServiceObjectiveName**：请求的 [DWU][DWU] 的数量。支持的值包括：DW100、DW200、DW300、DW400、DW500、DW600、DW1000、DW1200、DW1500、DW2000、DW3000 和 DW6000。
* **DatabaseName**：要创建的 SQL 数据仓库的名称。
* **ServerName**：用于创建过程的服务器名称（必须是 V12）。
* **ResourceGroupName**：要使用的资源组。若要查找订阅中可用的资源，请使用 Get-AzureResource。
* **Edition**：必须为“DataWarehouse”才能创建 SQL 数据仓库。

可选参数包括：

- **CollationName**：在不指定的情况下，默认排序规则是 SQL\_Latin1\_General\_CP1\_CI\_AS。在数据库上不能更改排序规则。
- **MaxSizeBytes**：数据库的默认最大大小为 10 GB。

有关参数选项的更多详细信息，请参阅 [New-AzureRmSqlDatabase][New-AzureRmSqlDatabase] 和[创建数据库（Azure SQL 数据仓库）][Create Database (Azure SQL Data Warehouse)]。

## 后续步骤
完成 SQL 数据仓库预配之后，可能需要尝试[加载示例数据][loading sample data]或了解如何[开发][develop]、[加载][load]或[迁移][migrate]数据。

如果有兴趣进一步了解如何以编程方式管理 SQL 数据仓库，请查看有关如何使用 [PowerShell cmdlet 和 REST API][PowerShell cmdlets and REST APIs] 的文章。

<!--Image references-->

<!--Article references-->
[DWU]: ./sql-data-warehouse-overview-what-is.md#data-warehouse-units
[migrate]: ./sql-data-warehouse-overview-migrate.md
[develop]: ./sql-data-warehouse-overview-develop.md
[load]: ./sql-data-warehouse-load-with-bcp.md
[loading sample data]: ./sql-data-warehouse-load-sample-databases.md
[PowerShell cmdlets and REST APIs]: ./sql-data-warehouse-reference-powershell-cmdlets.md
[firewall rules]: ../sql-database/sql-database-configure-firewall-settings.md

[How to install and configure Azure PowerShell]: ../powershell-install-configure.md
[how to create a SQL Data Warehouse from the Azure Portal]: ./sql-data-warehouse-get-started-provision.md
[Create an Azure SQL Database logical server with the Azure Portal]: ../sql-database/sql-database-get-started.md#create-a-new-logical-sql-server
[Create an Azure SQL Database logical server with PowerShell]: ../sql-database/sql-database-get-started-powershell.md#complete-azure-powershell-script-to-create-a-server-firewall-rule-and-database
[how to create a resource group]: ../azure-resource-manager/resource-group-portal.md

<!--MSDN references--> 
[MSDN]: https://msdn.microsoft.com/zh-cn/library/azure/dn546722.aspx
[New-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619339.aspx
[Create Database (Azure SQL Data Warehouse)]: https://msdn.microsoft.com/zh-cn/library/mt204021.aspx

<!--Other Web references-->
[Microsoft Web Platform Installer]: https://aka.ms/webpi-azps
[SQL 数据仓库定价]: https://www.azure.cn/pricing/details/sql-data-warehouse/
[Azure 试用版]: https://www.azure.cn/pricing/free-trial/?WT.mc_id=A261C142F
[MSDN Azure 信用额度]: https://www.azure.cn/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F

<!---HONumber=Mooncake_Quality_Review_1230_2016-->