---
title: 使用 T-SQL 配置 Azure SQL 数据库服务器级别和数据库级别防火墙规则 | Azure
description: 了解如何为访问 Azure SQL 数据库的 IP 地址配置防火墙。
services: sql-database
documentationCenter: ''
authors: BYHAM
manager: jhubbard
editor: ''

ms.service: sql-database
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 08/30/2016
wacn.date: 12/26/2016
ms.author: rickbyh
---

#<a name="manage-server-level-firewall-rules-through-transact-sql"></a> 使用 T-SQL 配置 Azure SQL 数据库服务器级别和数据库级别防火墙规则

> [!div class="op_single_selector"]
- [概述](./sql-database-firewall-configure.md)
- [Azure 门户预览](./sql-database-configure-firewall-settings.md)
- [TSQL](./sql-database-configure-firewall-settings-tsql.md)
- [PowerShell](./sql-database-configure-firewall-settings-powershell.md)
- [REST API](./sql-database-configure-firewall-settings-rest.md)

Azure SQL 数据库使用防火墙规则，以便允许连接到服务器和数据库。可在 Azure SQL 数据库服务器中为 master 数据库或用户数据库定义服务器级别和数据库级别防火墙设置，从而有选择地允许对数据库的访问。

> [!IMPORTANT]
> 若要允许来自 Azure 的应用程序连接到数据库服务器，必须启用 Azure 连接。有关防火墙规则和启用来自 Azure 的连接的详细信息，请参阅 [Azure SQL 数据库防火墙](./sql-database-firewall-configure.md)。如果要在 Azure 云边界内建立连接，可能需要打开一些其他的 TCP 端口。有关详细信息，请参阅[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](./sql-database-develop-direct-route-ports-adonet-v12.md)中的 **SQL 数据库 V12：内部与外部**部分

## 服务器级别防火墙规则

只有服务器级别主体登录名或 Azure Active Directory 管理员才能使用 Transact-SQL 创建服务器级别防火墙规则。

1. 启动查询窗口，并使用 SQL Server Management Studio 连接到虚拟 master 数据库。
2. 可以从查询窗口选择、创建、更新或删除服务器级别防火墙规则。
3. 若要创建或更新服务器级别防火墙规则，请执行 `sp_set_firewall_rule` 存储过程。以下示例启用服务器 Contoso 上的某一 IP 地址范围。<br/>首先，查看已存在哪些规则。

    ```
    SELECT * FROM sys.firewall_rules ORDER BY name;
    ```

    其次，添加防火墙规则。

    ```
    EXECUTE sp_set_firewall_rule @name = N'ContosoFirewallRule',
        @start_ip_address = '192.168.1.1', @end_ip_address = '192.168.1.10'
    ```

    若要删除服务器级别防火墙规则，请执行 sp\_delete\_firewall\_rule 存储过程。以下示例删除名为 ContosoFirewallRule 的规则。

    ```
    EXECUTE sp_delete_firewall_rule @name = N'ContosoFirewallRule'
    ```

 有关这些存储过程的详细信息，请参阅 [sp\_set\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270017.aspx) 和 [sp\_delete\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270024.aspx)。

##<a name="database-level-firewall-rules"></a> 数据库级别防火墙规则

只有对数据库具备**控制**权限的数据库用户（例如数据库所有者）才能创建数据库级别防火墙规则。

1. 为 IP 地址创建服务器级别防火墙后，通过经典管理门户或 SQL Server Management Studio 启动一个查询窗口。
2. 连接到要为其创建数据库级别防火墙规则的数据库。

    若要创建新的或更新现有的数据库级别防火墙规则，请执行 `sp_set_database_firewall_rule` 存储过程。以下示例创建名为 ContosoFirewallRule 的新防火墙规则。

    ```
    EXEC sp_set_database_firewall_rule @name = N'ContosoFirewallRule', 
        @start_ip_address = '192.168.1.11', @end_ip_address = '192.168.1.11'
    ```

    若要删除现有的数据库级别防火墙规则，请执行 `sp_delete_database_firewall_rule` 存储过程。以下示例删除名为 ContosoFirewallRule 的规则。

    ```
    EXEC sp_delete_database_firewall_rule @name = N'ContosoFirewallRule'
    ```

有关这些存储过程的详细信息，请参阅 [sp\_set\_database\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270010.aspx) 和 [sp\_delete\_database\_firewall\_rule](https://msdn.microsoft.com/zh-cn/library/dn270030.aspx)。

## 后续步骤

有关如何使用其他方式创建服务器级别防火墙规则的指导文章，请参阅：

- [使用 Azure 门户预览配置 Azure SQL 数据库服务器级别防火墙规则](./sql-database-configure-firewall-settings.md)
- [使用 PowerShell 配置 Azure SQL 数据库服务器级别防火墙规则](./sql-database-configure-firewall-settings-powershell.md)
- [使用 REST API 配置 Azure SQL 数据库服务器级别防火墙规则](./sql-database-configure-firewall-settings-rest.md) 有关创建数据库的教程，请参阅[使用 Azure 门户在几分钟内创建 SQL 数据库](./sql-database-get-started.md)。有关从开放源代码或第三方应用程序连接到 Azure SQL 数据库的帮助信息，请参阅 [SQL 数据库的客户端快速入门代码示例](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。若要了解如何导航到数据库，请参阅[管理数据库的访问和登录安全](./sql-database-manage-logins.md)。

## 其他资源

- [保护数据库的安全](./sql-database-security.md)
- [SQL Server 数据库引擎和 Azure SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)

<!---HONumber=Mooncake_Quality_Review_1215_2016-->