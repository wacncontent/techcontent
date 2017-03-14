---
title: 使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库 | Azure
description: 了解如何通过使用 Azure Active Directory 身份验证连接到 SQL 数据库
services: sql-database
documentationcenter: ''
author: BYHAM
manager: jhubbard
editor: ''
tags: ''

ms.assetid: 7e2508a1-347e-4f15-b060-d46602c5ce7e
ms.service: sql-database
ms.custom: authentication and authorization
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: data-management
ms.date: 11/22/2016
wacn.date: 01/20/2017
ms.author: rick.byham@microsoft.com
---

# 使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库
Azure Active Directory 身份验证是使用 Azure Active Directory (Azure AD) 中的标识连接到 Azure SQL 数据库和 [SQL 数据仓库](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)的一种机制。通过 Azure AD 身份验证，可在一个中心位置中集中管理数据库用户和其他 Microsoft 服务的标识。集中 ID 管理提供一个单一位置来管理数据库用户，并简化权限管理。包括如下优点：

- 提供一个 SQL Server 身份验证的替代方法。
- 帮助阻止用户标识在数据库服务器之间激增。
- 允许在单一位置中轮换密码
- 客户可以使用外部 (AAD) 组管理数据库权限。
- 它可以通过启用集成的 Windows 身份验证和 Azure Active Directory 支持的其他形式的身份验证来消除存储密码。
- Azure AD 身份验证使用包含的数据库用户以数据库级别对标识进行身份验证。
- Azure AD 支持对连接到 SQL 数据库的应用程序进行基于令牌的身份验证。
- Azure AD 身份验证支持对本地 Azure Active Directory 进行 ADFS（域联合）或本机用户/密码身份验证，无需进行域同步。
- Azure AD 支持从 SQL Server Management Studio 进行连接，后者使用 Active Directory 通用身份验证，其中包括多重身份验证 (MFA)。MFA 包括利用一系列简单的验证选项进行的强身份验证，这些选项包括电话、短信、含有 PIN 码的智能卡或移动应用通知。有关详细信息，请参阅 [SQL 数据库和 SQL 数据仓库针对 Azure AD MFA 的 SSMS 支持](./sql-database-ssms-mfa-authentication.md)。

配置步骤包括配置和使用 Azure Active Directory 身份验证的以下过程。

1. 创建并填充 Azure AD。
2. 确保你的数据库位于 Azure SQL 数据库 V12 中。（SQL 数据仓库无此要求。）
3. 可选：关联或更改当前与 Azure 订阅关联的 Active Directory。
4. 为 Azure SQL Server 或 [Azure SQL 数据仓库](https://www.azure.cn/home/features/sql-data-warehouse/)创建 Azure Active Directory 管理员。
5. 配置客户端计算机。
6. 在映射到 Azure AD 标识的数据库中创建包含的数据库用户。
7. 通过使用 Azure AD 标识连接到你的数据库。

## 信任体系结构
以下高级别关系图概述了将 Azure AD 身份验证用于 Azure SQL 数据库的解决方案体系结构。相同的概念适用于 SQL 数据仓库。若要支持 Azure AD 本机用户密码，只需考虑云部分和 Azure AD/Azure SQL 数据库。若要支持联合身份验证（或 Windows 凭据的用户/密码），需要与 ADFS 块进行通信。箭头表示通信路径。

![AAD 身份验证关系图][1]

下图表明允许客户端通过提交令牌连接到数据库的联合、信任和托管关系。该令牌已由 Azure AD 进行身份验证且受数据库信任。客户 1 可以代表具有本机用户的 Azure Active Directory 或具有联合用户的 Azure AD。客户 2 代表包含已导入用户的可行解决方案；在本例中，来自联合 Azure Active Directory 且 ADFS 正与 Azure Active Directory 进行同步。请务必了解，使用 Azure AD 身份验证访问数据库需要托管订阅与 Azure AD 相关联。必须使用相同的订阅创建托管 Azure SQL 数据库或 SQL 数据仓库的 SQL Server。

![订阅关系][2]

## 管理员结构
使用 Azure AD 身份验证时，SQL 数据库服务器会有两个管理员帐户：原始的 SQL Server 管理员和 Azure AD 管理员。相同的概念适用于 SQL 数据仓库。只有基于 Azure AD 帐户的管理员可以在用户数据库中创建第一个 Azure AD 包含的数据库用户。Azure AD 管理员登录名可以是 Azure AD 用户，也可以是 Azure AD 组。当管理员为组帐户时，可以由任何组成员使用，为 SQL Server 实例启用多个 Azure AD 管理员。以管理员身份使用组帐户时，允许你无需更改 SQL 数据库中的用户或权限，便可集中添加和删除 Azure AD 中的组成员，从而提高可管理性。无论何时都仅可配置一个 Azure AD 管理员（一个用户或组）。

![管理结构][3]

## 权限
若要新建用户，必须具有数据库中的 `ALTER ANY USER` 权限。`ALTER ANY USER` 权限可以授予任何数据库用户。`ALTER ANY USER` 权限还由服务器管理员帐户、具有该数据库的 `CONTROL ON DATABASE` 或 `ALTER ON DATABASE` 权限的数据库用户以及 `db_owner` 数据库角色的成员拥有。

若要在 Azure SQL 数据库或 SQL 数据仓库中创建一个包含的数据库用户，必须使用 Azure AD 标识连接到数据库。若要创建第一个包含数据库用户，必须通过使用 Azure AD 管理员（其是数据库的所有者）连接到数据库。在下面的步骤 4 和 5 中演示了此操作。只有为 Azure SQL 数据库或 SQL 数据仓库服务器创建 Azure AD 管理员之后，才有可能进行任何 Azure AD 身份验证。如果已从服务器删除 Azure Active Directory 管理员，先前在 SQL Server 内创建的现有 Azure Active Directory 用户便无法再使用其 Azure Active Directory 凭据连接到数据库。

## Azure AD 功能和限制
可以在 Azure SQL Server 或 SQL 数据仓库中预配以下 Azure AD 成员：

- 本机成员：在托管域或客户域中的 Azure AD 中创建的成员。有关详细信息，请参阅[将自己的域名添加到 Azure AD](../active-directory/active-directory-add-domain.md)。
- 联合域成员：在联合域的 Azure AD 中创建的成员。有关详细信息，请参阅 [Azure 现在支持与 Windows Server Active Directory 联合](https://azure.microsoft.com/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory)。
- 作为本机或联合域成员从其他 Azure AD 导入的成员。
- 以安全组形式创建的 Active Directory 组。

不支持 Microsoft 帐户（例如 outlook.com、hotmail.com、live.com）或其他来宾帐户（例如 gmail.com、yahoo.com）。如果可以使用帐户和密码登录到 [https://login.live.com](https://login.live.com)，则使用的是 Microsoft 帐户，Azure SQL 数据库或 Azure SQL 数据仓库的 Azure AD 身份验证不支持此类帐户。

### 其他注意事项

- 为了增强可管理性，建议将一个专用 Azure AD 组预配为管理员。
- 无论何时都仅可为 Azure SQL Server 或 Azure SQL 数据仓库配置一个 Azure AD 管理员（一个用户或组）。
- 只有 SQL Server 的 Azure AD 管理员最初可以使用 Azure Active Directory 帐户连接到 Azure SQL Server 或 Azure SQL 数据仓库。Active Directory 管理员可以配置后续的 Azure AD 数据库用户。
- 我们建议将连接超时值设置为 30 秒。
- SQL Server 2016 Management Studio 和 SQL Server Data Tools for Visual Studio 2015（版本 14.0.60311.1（2016 年 4 月）或更高版本）支持 Azure Active Directory 身份验证。（**用于 SqlServer 的 .NET Framework 数据提供程序**（.NET Framework 4.6 或更高版本）支持 Azure AD 身份验证）。因此，这些工具和数据层应用程序（DAC 和 .bacpac）的最新版本可以使用 Azure AD 身份验证。
- 虽然 [ODBC 版本 13.1](https://www.microsoft.com/download/details.aspx?id=53339) 支持 Azure Active Directory 身份验证，但是 `bcp.exe` 无法使用 Azure Active Directory 身份验证进行连接，因为使用的是旧式 ODBC 提供程序。
- `sqlcmd` 从版本 13.1 开始就支持 Azure Active Directory 身份验证，该版本可从[下载中心](http://go.microsoft.com/fwlink/?LinkID=825643)下载。
- SQL Server Data Tools for Visual Studio 2015 至少需要 2016 年 4 月版的 Data Tools（版本 14.0.60311.1）。目前，Azure AD 用户不会显示在 SSDT 对象资源管理器中。解决方法是在 [sys.database\_principals](https://msdn.microsoft.com/zh-cn/library/ms187328.aspx) 中查看这些用户。
- [Microsoft JDBC Driver 6.0 for SQL Server](https://www.microsoft.com/zh-CN/download/details.aspx?id=11774) 支持 Azure AD 身份验证。另外，请参阅[设置连接属性](https://msdn.microsoft.com/zh-cn/library/ms378988.aspx)。
- PolyBase 无法使用 Azure AD 身份验证进行身份验证。
- 不支持 BI 和 Excel 这样的工具。
- Azure 门户预览的“导入数据库”和“导出数据库”边栏选项卡支持 SQL 数据库的 Azure AD 身份验证。PowerShell 命令也支持使用 Azure AD 身份验证的导入和导出。

## 1\.创建并填充 Azure AD
创建 Azure AD 并对其填充用户和组。Azure AD 可以是初始域 Azure AD 托管的域。Azure AD 也可以是本地 Active Directory 域服务，该服务可以与 Azure AD 联合。

有关详细信息，请参阅[将本地标识与 Azure Active Directory 集成](../active-directory/active-directory-aadconnect.md)、[将自己的域名添加到 Azure AD](../active-directory/active-directory-add-domain.md)、[Azure 现在支持与 Windows Server Active Directory 联合](https://azure.microsoft.com/blog/2012/11/28/windows-azure-now-supports-federation-with-windows-server-active-directory)、[管理 Azure AD 目录](https://msdn.microsoft.com/zh-cn/library/azure/hh967611.aspx)、[使用 Windows PowerShell 管理 Azure AD](https://msdn.microsoft.com/zh-cn/library/azure/jj151815.aspx) 和[混合标识所需端口和协议](../active-directory/active-directory-aadconnect-ports.md)。

## 2\.确保 SQL 数据库版本为 12
在最新的 SQL 数据库 V12 中支持 Azure AD 身份验证。有关 SQL 数据库 V12 以及了解其是否已在你所在区域提供的信息，请参阅 [SQL 数据库的功能](./sql-database-features.md)。Azure SQL 数据仓库无需执行此步骤，因为 SQL 数据仓库只适用于 V12。

如果已经有一个数据库，请连接到该数据库（例如，使用 SQL Server Management Studio）并执行 `SELECT @@VERSION;`，验证其是否托管在 SQL 数据库 V12 中。SQL 数据库 V12 中数据库的预期输出至少是 **Microsoft SQL Azure (RTM) - 12.0**。如果数据库未在 SQL 数据库 V12 中托管，请参阅[计划并准备升级到 SQL 数据库 V12](./sql-database-v12-plan-prepare-upgrade.md)，之后访问 Azure 经典管理门户，将数据库迁移到 SQL 数据库 V12。

或者，可以按照[创建第一个 Azure SQL 数据库](./sql-database-get-started.md)中列出的步骤，在 SQL 数据库 V12 中新建数据库。

>  [!TIP]   
 请先阅读下一步，之后为新的数据库选择订阅。

## 3\.可选：关联或更改当前与 Azure 订阅关联的 Active Directory
若要将数据库与组织的 Azure AD 目录相关联，请允许该目录成为托管数据库的 Azure 订阅的一个受信任目录。有关详细信息，请参阅 [Azure 订阅与 Azure AD 的关联方式](https://msdn.microsoft.com/zh-cn/library/azure/dn629581.aspx)。

**其他信息：**每个 Azure 订阅都与某个 Azure AD 实例存在信任关系。这意味着，此订阅信任该目录对用户、服务和设备执行身份验证。多个订阅可以信任同一个目录，但一个订阅只能信任一个目录。可以访问 [https://manage.windowsazure.cn](https://manage.windowsazure.cn)，在“设置”选项卡下查看你的订阅信任的目录。订阅与目录之间的这种信任关系不同于订阅与 Azure 中所有其他资源（网站、数据库等）之间的信任关系，在后一种关系中，这些资源更像是订阅的子资源。如果某个订阅过期，则对该订阅关联的其他那些资源的访问权限也将终止。但是，目录将保留在 Azure 中，并且你可以将另一个订阅与该目录相关联，然后继续管理目录用户。有关资源的详细信息，请参阅[了解 Azure 中的资源访问](https://msdn.microsoft.com/zh-cn/library/azure/dn584083.aspx)。

以下过程提供分步说明，介绍如何更改给定订阅的关联目录。

1. 使用 Azure 订阅管理员连接到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 在左侧标题中，选择“设置”。
3. 你的订阅显示在“设置”屏幕中。如果未显示所需订阅，请单击顶部的“订阅”，下拉“按目录筛选”框，并选择包含你的订阅的目录，然后单击“应用”。

    ![选择订阅][4]
4. 在“设置”区域中，单击你的订阅，然后单击页面底部的“编辑目录”。

    ![AD-设置-门户][5]
5. 在“编辑目录”框中，选择与 SQL Server 或 SQL 数据仓库相关联的 Azure Active Directory，然后单击箭头转到下一步。

    ![编辑-目录-选择][6]
6. 在“确认”目录“映射”对话框中，确认“全部协同管理员都将被删除”。

    ![编辑-目录-确认][7]
7. 单击复选标记以重新加载门户。

> [!NOTE]
> 更改目录时，将删除所有协同管理员、Azure AD 用户和组以及目录支持的资源用户的访问权限，他们不再有权访问此订阅或其资源。只有作为服务管理员时，用户才能基于新的目录配置主体的访问权限。此更改可能需要大量时间来传播到所有资源。更改目录时还会更改 SQL 数据库和 SQL 数据仓库的 Azure AD 管理员，并且不允许任何现有 Azure AD 用户访问数据库。必须重置 Azure AD 管理员（如下所述），并且必须创建新的 Azure AD 用户。

## 4\.为 Azure SQL Server 创建 Azure AD 管理员

每个 Azure SQL Server（托管 SQL 数据库或 SQL 数据仓库）开始时只使用单个服务器管理员帐户，即整个 Azure SQL Server 的管理员。必须创建第二个 SQL Server 管理员，这是一个 Azure AD 帐户。此主体在 master 数据库中作为包含的数据库用户创建。作为管理员，服务器管理员帐户是每个用户数据库中 **db\_owner** 角色的成员，并且以 **dbo** 用户身份输入每个用户数据库。有关服务器管理员帐户的详细信息，请参阅[管理 Azure SQL 数据库中的数据库和登录名](./sql-database-manage-logins.md)以及 [Azure SQL 数据库安全指导原则和限制](./sql-database-security-guidelines.md)中的**登录名和用户**部分。

将 Azure Active Directory 与异地复制结合使用时，必须为主服务器和辅助服务器配置 Azure Active Directory 管理员。如果服务器没有 Azure Active Directory 管理员，则 Azure Active Directory 登录名和用户会收到“无法连接到服务器”错误。

> [!NOTE]
> 如果用户使用的不是基于 Azure AD 的帐户（包括 Azure SQL Server 管理员帐户），则无法创建基于 Azure AD 的用户，这是因为他们无权使用 Azure AD 验证建议的数据库用户。

### 使用 Azure 门户预览为 Azure SQL Server 预配 Azure Active Directory 管理员

1. 在 [Azure 门户预览](https://portal.azure.cn/)右上角，单击相关连接以下拉包含可能 Active Directory 的列表。选择正确的 Active Directory 作为默认的 Azure AD。此步骤将与 Active Directory 关联的订阅链接到 Azure SQL Server，确保为 Azure AD 和 SQL Server 使用相同的订阅。（Azure SQL Server 托管的可能是 Azure SQL 数据库或 Azure SQL 数据仓库。）

    ![选择-AD][8]
2. 在左侧标题中，选择“SQL Server”、选择你的“SQL Server”，然后在“SQL Server”边栏选项卡顶部，单击“设置”。

    ![AD 设置][9]
3. 在“设置”边栏选项卡中，单击**“Active Directory 管理员”。
4. 在“Active Directory 管理员”边栏选项卡中，单击“Active Directory 管理员”，然后在顶部单击“设置管理员”。
5. 在“添加管理员”边栏选项卡中，搜索某个用户、将选择该用户或组作为管理员，然后单击“选择”。（“Active Directory 管理员”边栏选项卡将显示 Active Directory 的所有成员和组。若用户或组为灰显，则无法选择，因为不支持它们作为 Azure AD 管理员。（请参阅上述 **Azure AD 功能和限制**中受支持的管理员列表。） 基于角色的访问控制 (RBAC) 仅适用于该门户，不会传播到 SQL Server。
6. 在“Active Directory 管理员”边栏选项卡顶部，单击“保存”。
    ![选择管理员][10]

    更改管理员的过程可能需要几分钟时间。然后，新管理员将出现在“Active Directory 管理员”框中。

> [!NOTE]
> 设置 Azure AD 管理员时，此新的管理员名称（用户或组）不能已作为 SQL Server 身份验证用户存在于虚拟 master 数据库中。如果存在，Azure AD 管理员设置将失败；将回滚其创建，并指示此管理员（名称）已存在。由于这种 SQL Server 身份验证用户不是 Azure AD 的一部分，因此使用 Azure AD 身份验证连接到服务器的任何尝试都将失败。

之后若要删除管理员，请在“Active Directory 管理员”边栏选项卡顶部，单击“删除管理员”，然后单击“保存”。

### 使用 PowerShell 为 Azure SQL Server 预配 Azure AD 管理员

若要运行 PowerShell cmdlet，需要已安装并运行 Azure PowerShell。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](../powershell-install-configure.md)。

若要预配 Azure AD 管理员，请执行以下 Azure PowerShell 命令：

- Add-AzureRmAccount -EnvironmentName AzureChinaCloud
- Select-AzureRmSubscription

用于预配和管理 Azure AD 管理员的 Cmdlet：

| Cmdlet 名称 | 说明 |
| --- | --- |
| [Set-AzureRmSqlServerActiveDirectoryAdministrator](https://msdn.microsoft.com/zh-cn/library/azure/mt603544.aspx) | 为 Azure SQL Server 或 Azure SQL 数据仓库预配 Azure Active Directory 管理员。（必须来自当前订阅。） |
| [Remove-AzureRmSqlServerActiveDirectoryAdministrator](https://msdn.microsoft.com/zh-cn/library/azure/mt619340.aspx) | 为 Azure SQL Server 或 Azure SQL 数据仓库删除 Azure Active Directory 管理员。 |
| [Get-AzureRmSqlServerActiveDirectoryAdministrator](https://msdn.microsoft.com/zh-cn/library/azure/mt603737.aspx) | 返回有关当前为 Azure SQL Server 或 Azure SQL 数据仓库配置的 Azure Active Directory 管理员的信息。 |

使用 PowerShell 命令 get-help 可查看有关其中每个命令的更多详细信息，例如 ``get-help Set-AzureRmSqlServerActiveDirectoryAdministrator``。

以下脚本为名为 **Group-23** 的资源组中的 **demo\_server** 服务器预配名为 **DBA\_Group** 的 Azure AD 管理员组（对象 ID `40b79501-b343-44ed-9ce7-da4c8cc7353f`）：

```
Set-AzureRmSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23"
–ServerName "demo_server" -DisplayName "DBA_Group"
```

**DisplayName** 输入参数接受 Azure AD 显示名称或用户主体名称。例如，``DisplayName="John Smith"`` 和 ``DisplayName="johns@contoso.com"``。对于 Azure AD 组，只支持 Azure AD 显示名称。

> [!NOTE]
> Azure PowerShell 命令 `Set-AzureRmSqlServerActiveDirectoryAdministrator` 不会阻止你为不受支持的用户预配 Azure AD 管理员。可以预配不受支持的用户，但其无法连接到数据库。（请参阅上述 **Azure AD 功能和限制**中受支持的管理员列表。）

以下示例使用可选的 **ObjectID**：

```
Set-AzureRmSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23"
–ServerName "demo_server" -DisplayName "DBA_Group" -ObjectId "40b79501-b343-44ed-9ce7-da4c8cc7353f"
```

> [!NOTE]
> 在 **DisplayName** 不唯一时，需要使用 Azure AD **ObjectID**。若要检索 **ObjectID** 和 **DisplayName** 值，请使用 Azure 经典管理门户的 Active Directory 部分，并查看用户或组的属性。

下面的示例返回有关针对 Azure SQL Server 的当前 Azure AD 管理员的信息：

```
Get-AzureRmSqlServerActiveDirectoryAdministrator –ResourceGroupName "Group-23" –ServerName "demo_server" | Format-List
```

下面的示例删除一个 Azure AD 管理员：

```
Remove-AzureRmSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" –ServerName "demo_server"
```

也可以使用 REST API 预配 Azure Active Directory 管理员。有关详细信息，请参阅 [Azure SQL 数据库的 Azure SQL 数据库操作的 Service Management REST API 参考和操作](https://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx)

## 5\.配置客户端计算机
在所有客户端计算机上，如果你的应用程序或用户从中使用 Azure AD 标识连接到 Azure SQL 数据库或 Azure SQL 数据仓库，则必须安装以下软件：

- .NET Framework 4.6 或更高版本，可从 [https://msdn.microsoft.com/zh-cn/library/5a4x27ek.aspx](https://msdn.microsoft.com/zh-cn/library/5a4x27ek.aspx) 下载。
- 用于 SQL Server 的 Azure Active Directory 身份验证库 (**ADALSQL.DLL**)，提供多个语言版本（x86 和 amd64），可从下载中心中的[用于 Microsoft SQL Server 的 Active Directory 身份验证库](http://www.microsoft.com/zh-cn/download/details.aspx?id=48742)下载。

### 工具

- 安装 [SQL Server 2016 Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 或 [SQL Server Data Tools for Visual Studio 2015](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx) 符合 .NET Framework 4.6 要求。
- SSMS 安装 **ADALSQL.DLL** 的 x86 版本。
- SSDT 安装 **ADALSQL.DLL** 的 amd64 版本。
- [Visual Studio 下载](https://www.visualstudio.com/downloads/download-visual-studio-vs)提供的最新 Visual Studio 符合 .NET Framework 4.6 要求，但并未安装必需的 amd64 版本的 **ADALSQL.DLL**。

## 6\.在映射到 Azure AD 标识的数据库中创建包含的数据库用户
### 关于包含的数据库用户
Azure Active Directory 身份验证要求以包含的数据库用户的身份创建数据库用户。基于 Azure AD 标识的包含的数据库用户是在 master 数据库中不具有登录名的数据库用户，它映射到与数据库相关联的 Azure AD 目录中的标识。Azure AD 标识可以是单独的用户帐户，也可以是组。有关包含的数据库用户的详细信息，请参阅[包含的数据库用户 - 使你的数据库可移植](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)。

> [!NOTE]
> 不能使用门户创建数据库用户（管理员除外）。RBAC 角色不会传播到 SQL Server、SQL 数据库或 SQL 数据仓库。Azure RBAC 角色用于管理 Azure 资源，不会应用于数据库权限。例如，**SQL Server 参与者**角色不会授予连接到 SQL 数据库或 SQL 数据仓库的访问权限。必须使用 Transact-SQL 语句直接在数据库中授予访问权限。

### 使用 SQL Server Management Studio 或 SQL Server Data Tools 连接到用户数据库或数据仓库
若要确认 Azure AD 管理员已正确设置，请使用 Azure AD 管理员帐户连接到 **master** 数据库。若要预配基于 Azure AD 的包含的数据库用户（而不是拥有数据库的服务器管理员），请使用具有数据库访问权限的 Azure AD 标识连接到数据库。

> [!IMPORTANT]
> [SQL Server 2016 Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 和 Visual Studio 2015 中的 [SQL Server Data Tools](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx) 支持 Azure Active Directory 身份验证。2016 年 8 月版 SSMS 也包括对 Active Directory 通用身份验证的支持，这样管理员就能要求用户使用手机、短信、带 PIN 码的智能卡或移动应用通知进行多重身份验证。

####<a name="connect-using-active-directory-integrated-authentication"></a> 使用 Active Directory 集成的身份验证进行连接

如果你从联合域使用 Azure Active Directory 凭据登录到 Windows，则使用此方法。

1. 启动 Management Studio 或 Data Tools 后，在“连接到服务器”（或“连接到数据库引擎”）对话框的“身份验证”框中，选择“Active Directory 集成身份验证”。由于系统会为连接提供你的现有凭据，因此无需密码，也无法输入密码。
    ![选择 AD 集成身份验证][11]

2. 单击“选项”按钮，在“连接属性”页上的“连接到数据库”框中，键入你所要连接的用户数据库的名称。
    ![选择数据库名称][13]

####<a name="connect-using-active-directory-password-authentication"></a> 使用 Active Directory 密码身份验证进行连接

在使用 Azure AD 托管域与 Azure AD 主体名称连接时使用此方法。你还可以将其用于无需访问该域的联合帐户，例如远程工作时。

如果从未与 Azure 联合的域中使用凭证登录到 Windows，或者基于初始域或客户端域通过 Azure AD 使用 Azure AD 身份验证，请使用此方法。

1. 启动 Management Studio 或 Data Tools，然后在“连接到服务器”（或“连接到数据库引擎”）对话框的“身份验证”框中，选择“Active Directory 密码身份验证”。
2. 在“用户名”框中，以格式 **username@domain.com** 键入你的 Azure Active Directory 用户名。这必须是来自 Azure Active Directory 的帐户或来自与 Azure Active Directory 联合的域的帐户。
3. 在“密码”框中，为 Azure Active Directory 帐户或联合域帐户键入你的用户密码。
    ![选择 AD 密码身份验证][12]

4. 单击“选项”按钮，然后在“连接属性”页上的“连接到数据库”框中，键入你所要连接用户数据库的名称。（请参阅前一选项的图。）

### 在用户数据库中创建 Azure AD 包含的数据库用户
若要创建基于 Azure AD 的包含的数据库用户（而不是拥有数据库的服务器管理员），请以至少具有 **ALTER ANY USER** 权限的用户身份使用 Azure AD 标识连接到数据库。然后，使用以下 Transact-SQL 语法：

```
CREATE USER <Azure_AD_principal_name>
FROM EXTERNAL PROVIDER;
```

*Azure\_AD\_principal\_name* 可以是 Azure AD 用户的用户主体名称，也可以是 Azure AD 组的显示名称。

**示例：**
若要创建代表 Azure AD 联合或托管域用户的包含的数据库用户：

```
CREATE USER [bob@contoso.com] FROM EXTERNAL PROVIDER;
CREATE USER [alice@fabrikam.partner.onmschina.cn] FROM EXTERNAL PROVIDER;
```

若要创建代表 Azure AD 或联合域组的包含的数据库用户，请提供安全组的显示名称：

```
CREATE USER [ICU Nurses] FROM EXTERNAL PROVIDER;
```

若要创建代表可使用 Azure AD 令牌连接的应用程序的包含的数据库用户：

```
CREATE USER [appName] FROM EXTERNAL PROVIDER;
```

>  [!TIP]
除了与 Azure 订阅关联的 Azure Active Directory 以外，无法从 Azure Active Directory 直接创建用户。但是，可将关联的 Active Directory 中导入的用户（称为外部用户）的其他 Active Directory 成员添加到租户 Active Directory 中的 Active Directory 组。通过创建该 AD 组的包含数据库用户，来自外部 Active Directory 的用户可以访问 SQL 数据库。

有关基于 Azure Active Directory 标识创建包含的数据库用户的详细信息，请参阅 [CREATE USER (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms173463.aspx)。

> [!NOTE]
> 删除 Azure SQL Server 的 Azure Active Directory 管理员会阻止所有 Azure AD 身份验证用户连接到服务器。必要时，SQL 数据库管理员可以手动删除无法使用的 Azure AD 用户。

>  [!NOTE]
如果收到“连接超时过期”，可能需要将 `TransparentNetworkIPResolution` 连接字符串的参数设置为 false。有关详细信息，请参阅 [.NET Framework 4.6.1 的连接超时问题 – TransparentNetworkIPResolution](https://blogs.msdn.microsoft.com/dataaccesstechnologies/2016/05/07/connection-timeout-issue-with-net-framework-4-6-1-transparentnetworkipresolution/)。

创建数据库用户时，该用户会收到 **CONNECT** 权限，并能够以 **PUBLIC** 角色的成员身份连接到该数据库。最初，仅供用户使用的权限是授予 **PUBLIC** 角色的任何权限，或者授予其所属任何 Windows 组的任何权限。预配基于 Azure AD 的包含的数据库用户后，你可以授予用户其他权限，方法与向任何其他类型的用户授予权限相同。通常，将权限授予数据库角色，并将用户添加到角色。有关详细信息，请参阅[数据库引擎权限基础知识](http://social.technet.microsoft.com/wiki/contents/articles/4433.database-engine-permission-basics.aspx)。有关特殊 SQL 数据库角色的详细信息，请参阅[在 Azure SQL 数据库中管理数据库和登录名](./sql-database-manage-logins.md)。如果将联合域用户导入到管理域，则此用户必须使用托管域标识。

> [!NOTE]
> Azure AD 用户在数据库元数据中均标记为类型 E (EXTERNAL\_USER)，而组则标记为类型 X (EXTERNAL\_GROUPS)。有关详细信息，请参阅 [sys.database\_principals](https://msdn.microsoft.com/zh-cn/library/ms187328.aspx)。

## 7\.使用 Azure AD 标识进行连接
Azure Active Directory 身份验证支持使用 Azure AD 标识连接到数据库的以下方法：

- 使用集成的 Windows 身份验证
- 使用 Azure AD 主体名称和密码
- 使用应用程序令牌身份验证

### 7\.1.使用集成的 (Windows) 身份验证进行连接
若要使用集成的 Windows 身份验证，域的 Active Directory 必须与 Azure Active Directory 联合。连接到数据库的客户端应用程序（或服务）必须运行在已使用用户的域凭据加入域的计算机上。

若要使用集成的身份验证和 Azure AD 标识连接到数据库，数据库连接字符串中的身份验证关键字必须设置为 Active Directory Integrated。下面的 C# 代码示例使用 ADO.NET。

```
string ConnectionString =
@"Data Source=n9lxnyuzhv.database.chinacloudapi.cn; Authentication=Active Directory Integrated; Initial Catalog=testdb;";
SqlConnection conn = new SqlConnection(ConnectionString);
conn.Open();
```

请注意，不支持使用连接字符串关键字 ``Integrated Security=True`` 连接到 Azure SQL 数据库。
请注意，在进行 ODBC 连接时，需删除空格，将“Authentication”设置为“ActiveDirectoryIntegrated”。

### 7\.2.使用 Azure AD 主体名称和密码进行连接
若要使用集成的身份验证和 Azure AD 标识连接到数据库，必须将“Authentication”关键字设置为“Active Directory Password”。连接字符串必须包含“User ID/UID”和“Password/PWD”关键字和值。下面的 C# 代码示例使用 ADO.NET。

```
string ConnectionString =
  @"Data Source=n9lxnyuzhv.database.chinacloudapi.cn; Authentication=Active Directory Password; Initial Catalog=testdb;  UID=bob@contoso.partner.onmschina.cn; PWD=MyPassWord!";
SqlConnection conn = new SqlConnection(ConnectionString);
conn.Open();
```

通过 [Azure AD 身份验证 GitHub 演示](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/security/azure-active-directory-auth)中提供的演示代码示例，了解有关 Azure AD 身份验证方法的详细信息。

### 7\.3 使用 Azure AD 令牌进行连接
此身份验证方法允许从 Azure Active Directory (AAD) 获取令牌，使中间层服务能够连接到 Azure SQL 数据库或 Azure SQL 数据仓库。这样，便可以实现包含基于证书的身份验证的复杂方案。必须完成四个基本步骤才能使用 Azure AD 令牌身份验证：

1. 在 Azure Active Directory 中注册你的应用程序，并获取代码的客户端 ID。
2. 创建代表应用程序的数据库用户。（前面在步骤 6 中已完成。）
3. 在运行应用程序的客户端计算机上创建证书。
4. 为应用程序添加用作密钥的证书。

示例连接字符串：

```
string ConnectionString =@"Data Source=n9lxnyuzhv.database.chinacloudapi.cn; Initial Catalog=testdb;"
SqlConnection conn = new SqlConnection(ConnectionString);
connection.AccessToken = "Your JWT token"
conn.Open();
```

有关详细信息，请参阅 [SQL Server 安全性博客](https://blogs.msdn.microsoft.com/sqlsecurity/2016/02/09/token-based-authentication-support-for-azure-sql-db-using-azure-ad-auth/)。

### 使用 sqlcmd 进行连接
以下语句使用版本 13.1 的 sqlcmd 进行连接，该版本可从[下载中心](http://go.microsoft.com/fwlink/?LinkID=825643)下载。

```
sqlcmd -S Target_DB_or_DW.testsrv.database.chinacloudapi.cn  -G  
sqlcmd -S Target_DB_or_DW.testsrv.database.chinacloudapi.cn -U bob@contoso.com -P MyAADPassword -G -l 30
```

## 另请参阅

[管理 Azure SQL 数据库中的数据库和登录名](./sql-database-manage-logins.md)

[包含的数据库用户](https://msdn.microsoft.com/zh-cn/library/ff929071.aspx)

[创建用户 (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms173463.aspx)

<!--Image references-->

[1]: ./media/sql-database-aad-authentication/1aad-auth-diagram.png
[2]: ./media/sql-database-aad-authentication/2subscription-relationship.png
[3]: ./media/sql-database-aad-authentication/3admin-structure.png
[4]: ./media/sql-database-aad-authentication/4select-subscription.png
[5]: ./media/sql-database-aad-authentication/5ad-settings-portal.png
[6]: ./media/sql-database-aad-authentication/6edit-directory-select.png
[7]: ./media/sql-database-aad-authentication/7edit-directory-confirm.png
[8]: ./media/sql-database-aad-authentication/8choose-ad.png
[9]: ./media/sql-database-aad-authentication/9ad-settings.png
[10]: ./media/sql-database-aad-authentication/10choose-admin.png
[11]: ./media/sql-database-aad-authentication/11connect-using-int-auth.png
[12]: ./media/sql-database-aad-authentication/12connect-using-pw-auth.png
[13]: ./media/sql-database-aad-authentication/13connect-to-db.png

<!---HONumber=Mooncake_0116_2017-->
<!--Update: change Active Directory to AD ; add three azure.tip-->