---
title: 将本地 AlwaysOn 可用性组扩展到 Azure | Azure
description: 本教程使用通过经典部署模型创建的资源，并介绍如何使用 SQL Server Management Studio (SSMS) 中的“添加副本”向导将 AlwaysOn 可用性组副本添加到 Azure 中。
services: virtual-machines-windows
documentationcenter: na
author: MikeRayMSFT
manager: jhubbard
editor: ''
tags: azure-service-management

ms.assetid: 7ca7c423-8342-4175-a70b-d5101dfb7f23
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: infrastructure-services
ms.date: 07/12/2016
wacn.date: 02/24/2017
ms.author: MikeRayMSFT
---

# 将本地 AlwaysOn 可用性组扩展到 Azure
AlwaysOn 可用性组通过添加辅助副本为数据库组提供高可用性。在发生故障时，可以使用这些副本来故障转移数据库。此外，它们还可用于卸载读取工作负荷或备份任务。

可预配包含 SQL Server 的一个或多个 Azure VM，然后将其作为副本添加到本地可用性组，进而将本地可用性组扩展到 Azure。

本教程假设你具有以下各项：

* 一个有效的 Azure 订阅。你可以注册[试用版](https://www.azure.cn/pricing/1rmb-trial/)。
* 现有本地 AlwaysOn 可用性组。有关可用性组的详细信息，请参阅 [AlwaysOn 可用性组](https://msdn.microsoft.com/zh-cn/library/hh510230.aspx)。
* 本地网络和 Azure 虚拟网络之间的连接。有关创建此虚拟网络的详细信息，请参阅[在 Azure 经典管理门户中配置站点到站点 VPN](../vpn-gateway/vpn-gateway-site-to-site-create.md)。

> [!IMPORTANT] 
Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 模型和经典模型](../azure-resource-manager/resource-manager-deployment-model.md)。本文介绍如何使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。

## 添加 Azure 副本向导
本部分演示如何使用**添加 Azure 副本向导**扩展 AlwaysOn 可用性组解决方案以使其包含 Azure 副本。

1. 从 SQL Server Management Studio 中，展开“AlwaysOn 高可用性”\>“可用性组”\>“\[可用性组的名称\]”。
2. 右键单击“可用性副本”，然后单击“添加副本”。
3. 默认情况下，将显示“将副本添加到可用性组向导”。单击**“下一步”**。如果你以前在启动此向导时在页面底部选择了“不再显示此页”选项，则不会显示此屏幕。

    ![SQL](./media/virtual-machines-windows-classic-sql-onprem-availability/IC742861.png)  

4. 你需要连接到所有现有辅助副本。你可以单击每个副本旁边的“连接...”，或者单击屏幕底部的“全部连接...”。身份验证之后，单击“下一步”转到下一屏幕。
5. 在“指定副本”页上，顶部列出了多个选项卡：“副本”、“终结点”、“备份首选项”和“侦听器”。从“副本”选项卡，单击“添加 Azure 副本...”以启动“添加 Azure 副本向导”。

    ![SQL](./media/virtual-machines-windows-classic-sql-onprem-availability/IC742863.png)  

6. 如果以前已安装 Azure 管理证书，请从本地 Windows 证书存储中选择现有的 Azure 管理证书。如果以前使用过 Azure 订阅的 ID，请选择或输入该 ID。可以单击“下载”以下载并安装 Azure 管理证书，然后使用 Azure 帐户下载订阅列表。

    ![SQL](./media/virtual-machines-windows-classic-sql-onprem-availability/IC742864.png)  

7. 使用将用于创建托管副本的 Azure 虚拟机 \(VM\) 的值，填充页面上的每个字段。

    | 设置 | 说明 |
    | --- | --- |
    | **映像** |选择操作系统和 SQL Server 的所需组合 |
    | **VM 大小** |选择最适合业务需求的 VM 大小 |
    | **VM 名称** |指定新 VM 的唯一名称。该名称必须包含 3 至 15 个字符，只能包含字母、数字和连字符，必须以字母开头，并且必须以字母或数字结尾。 |
    | **VM 用户名** |指定将成为 VM 上的管理员帐户的用户名 |
    | **VM 管理员密码** |指定新帐户的密码 |
    | **确认密码** |确认新帐户的密码 |
    | **虚拟网络** |指定新 VM 应使用的 Azure 虚拟网络。有关虚拟网络的详细信息，请参阅[虚拟网络概述](../virtual-network/virtual-networks-overview.md)。 |
    | **虚拟网络子网** |指定新 VM 应使用的虚拟网络子网 |
    | **域** |确认域的预填充值正确 |
    | **域用户名** |指定本地群集节点上的本地管理员组中的一个帐户 |
    | **密码** |指定域用户名的密码 |
8. 单击“确定”验证部署设置。
9. 接下来将显示法律条款。阅读这些条款，如果同意，请单击“确定”。
10. 再次显示“指定副本”页。在“副本”、“终结点”和“备份首选项”选项卡上验证新的 Azure 副本的设置。修改设置，使其符合业务需求。有关这些选项卡包含的参数的详细信息，请参阅“指定副本”页（新建可用性组向导/添加副本向导）。请注意，对于包含 Azure 副本的可用性组，无法使用“侦听器”选项卡创建侦听器。[](https://msdn.microsoft.com/zh-cn/library/hh213088.aspx)此外，如果侦听器在启动向导之前已经创建，你将收到一条消息，指示 Azure 不支持此功能。在**创建可用性组侦听器**部分，我们将探讨如何创建侦听器。

     ![SQL](./media/virtual-machines-windows-classic-sql-onprem-availability/IC742865.png)  

11. 单击“下一步”。
12. 在“选择初始数据同步”页上，选择你希望使用的数据同步方法，然后单击“下一步”。对于大多数情况，应选择“完全数据同步”。有关数据同步方法的详细信息，请参阅[“选择初始数据同步”页（AlwaysOn 可用性组向导）](https://msdn.microsoft.com/zh-cn/library/hh231021.aspx)。
13. 在“验证”页上查看结果。更正存在的问题，如有必要，请重新运行验证。单击**“下一步”**。

     ![SQL](./media/virtual-machines-windows-classic-sql-onprem-availability/IC742866.png)  

14. 在“摘要”页上查看设置，然后单击“完成”。
15. 预配过程开始。当向导成功完成时，单击“关闭”退出向导。

> [!NOTE]
“添加 Azure 副本向导”将在 Users\\User Name\\AppData\\Local\\SQL Server\\AddReplicaWizard 中创建一个日志文件。此日志文件可用于对出现故障的 Azure 副本部署进行故障排除。如果向导无法执行任何操作，则以前的所有操作都将回滚，包括删除预配的 VM。
> 
> 

## 创建可用性组侦听器
创建可用性组之后，你应该为客户端创建侦听器，以便连接到副本。侦听器可将传入连接定向至主副本或只读辅助副本。有关侦听器的详细信息，请参阅[在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器](./virtual-machines-windows-classic-ps-sql-int-listener.md)。

## 后续步骤
除了使用**添加 Azure 副本向导**将 AlwaysOn 可用性组扩展到 Azure 以外，还可以将某些 SQL Server 工作负荷完全移到 Azure。若要开始，请参阅[在 Azure 上预配 SQL Server 虚拟机](./virtual-machines-windows-portal-sql-server-provision.md)。

有关其他与在 Azure VM 中运行 SQL Server 相关的主题，请参阅 [Azure 虚拟机上的 SQL Server](./virtual-machines-windows-sql-server-iaas-overview.md)。

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->