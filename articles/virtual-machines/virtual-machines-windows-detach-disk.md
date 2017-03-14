---
title: 从 Windows VM 分离数据磁盘 | Azure
description: 了解如何从使用资源管理器部署模型的 Azure 中的虚拟机分离磁盘。
services: virtual-machines-windows
documentationCenter: ''
authors: cynthn
manager: timlt
editor: ''
tags: azure-service-management

ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 09/27/2016
wacn.date: 12/16/2016
ms.author: cynthn
---

# 如何从 Windows 虚拟机分离数据磁盘

当不再需要附加到虚拟机的数据磁盘时，可以轻松地分离它。这将从虚拟机中删除该磁盘，但不会从存储中删除它。

> [!WARNING]
> 如果用户分离磁盘，它将不会自动删除。如果用户订阅了高级存储，则将继续承担该磁盘的存储费用。有关详细信息，请参阅[使用高级存储时的定价和计费方式](../storage/storage-premium-storage.md#pricing-and-billing)。

如果希望再次使用该磁盘上的现有数据，可以将其重新附加到相同的虚拟机或另一个虚拟机。

## 使用门户分离数据磁盘

1. 在门户中心中，选择“虚拟机”。

2. 选择要分离的数据磁盘连接的虚拟机，然后单击“所有设置”。

3. 在“设置”边栏选项卡中，选择“磁盘”。

4. 在“磁盘”边栏选项卡中，选择要分离的数据磁盘。

5. 在该数据磁盘的边栏选项卡中，单击“分离”。

    ![显示“分离”按钮的屏幕截图。](./media/virtual-machines-windows-detach-disk/detach-disk.png)

磁盘保留在存储中，但不再附加到虚拟机。

## 使用 PowerShell 分离数据磁盘

在此示例中，第一个命令使用 Get-AzureRmVM cmdlet 获取 **RG11** 资源组中名为 **MyVM07** 的虚拟机。该命令在 **$VirtualMachine** 变量中存储虚拟机。

第二个命令将从该虚拟机中删除名为 DataDisk3 的数据磁盘。

最后一个命令更新虚拟机的状态，以完成删除数据磁盘过程。

```
$VirtualMachine = Get-AzureRmVM -ResourceGroupName "RG11" -Name "MyVM07" 
Remove-AzureRmVMDataDisk -VM $VirtualMachine -Name "DataDisk3"
Update-AzureRmVM -ResourceGroupName "RG11" -Name "MyVM07" -VM $VirtualMachine
```

有关详细信息，请参阅 [Remove-AzureRmVMDataDisk](https://msdn.microsoft.com/zh-cn/library/mt603614.aspx)

## 后续步骤

如果想重新使用数据磁盘，只需将它[附加到另一台 VM](./virtual-machines-windows-attach-disk-portal.md)

<!---HONumber=Mooncake_Quality_Review_1202_2016-->