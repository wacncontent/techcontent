---
title: 捕获 Azure Windows VM 的映像| Azure
description: 捕获使用经典部署模型创建的 Azure Windows 虚拟机的映像。
services: virtual-machines-windows
documentationCenter: ''
authors: cynthn
manager: timlt
editor: tysonn
tags: azure-service-management

ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 09/27/2016
wacn.date: 11/28/2016
ms.author: cynthn
---

#捕获使用经典部署模型创建的 Azure Windows 虚拟机的映像。

> [!IMPORTANT]
> Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](../azure-resource-manager/resource-manager-deployment-model.md)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。有关 Resource Manager 模型的信息，请参阅[为 Azure 中运行的 Windows VM 创建副本](./virtual-machines-windows-vhd-copy.md)。

本文将演示如何捕获运行 Windows 的 Azure 虚拟机，可以将它用作映像来创建其他虚拟机。此映像包含操作系统磁盘和任何附加到虚拟机的数据磁盘。由于它不包括网络配置，因此在使用此映像创建其他虚拟机时，需要进行相关配置。

Azure 将映像存储在**“我的映像”**下。上传的任何映像都会存储在同一位置。有关映像的详细信息，请参阅[关于虚拟机的映像](./virtual-machines-linux-classic-about-images.md)。

##开始之前##

这些步骤假定已创建了 Azure 虚拟机并配置了操作系统，包括附加任何数据磁盘。如果尚未执行此操作，请参阅以下说明：

- [从映像创建虚拟机](./virtual-machines-windows-classic-createportal.md)
- [如何将数据磁盘附加到虚拟机](./virtual-machines-windows-classic-attach-disk.md)
- 确保 Sysprep 支持服务器角色。有关详细信息，请参阅 [Sysprep Support for Server Roles](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)（Sysprep 对服务器角色的支持）。

> [!WARNING]
> 此过程会在捕获原始虚拟机后将其删除。

在捕获 Azure 虚拟机映像之前，建议备份目标虚拟机。可以使用 Azure 备份来备份 Azure 虚拟机。有关详细信息，请参阅[备份 Azure 虚拟机](../backup/backup-azure-vms.md)。认证合作伙伴提供了其他解决方案。若要了解目前提供的内容，请搜索 Azure 库或应用商店。

##捕获虚拟机

1. 在 [Azure 经典管理门户](http://manage.windowsazure.cn)中**连接**到虚拟机。有关说明，请参阅[如何登录到运行 Windows Server 的虚拟机][]。

2. 以管理员身份打开“命令提示符”窗口。

3. 将目录更改为 `%windir%\system32\sysprep`，然后运行 sysprep.exe。

4. 	此时会显示**“系统准备工具”**对话框。请执行以下操作：

    - 在**“系统清理操作”**中，选择**“进入系统全新体验(OOBE)”**，并确保选中**“一般化”**。有关使用 Sysprep 的详细信息，请参阅[如何使用 Sysprep：简介][]。

    - 在**“关机选项”**中选择**“关机”**。

    - 单击**“确定”**。

    ![运行 Sysprep](./media/virtual-machines-windows-classic-capture-image/SysprepGeneral.png)

7. sysprep 命令将关闭虚拟机，这会在 Azure 经典管理门户中将虚拟机的状态更改为“已停止”。

8. 在 Azure 经典管理门户中，单击“虚拟机”，然后选择要捕获的虚拟机。

9. 在命令栏中，单击**“捕获”**。

    ![捕获虚拟机](./media/virtual-machines-windows-classic-capture-image/CaptureVM.png)  

    此时将显示**“捕获虚拟机”**对话框。

10. 在**“映像名称”**中，键入新映像的名称。

11. 在将 Windows Server 映像添加到自定义映像集之前，必须先按前面步骤中的说明通过运行 Sysprep 使该映像通用化。单击**“我已经在虚拟机上运行了 Sysprep”**以指明你已完成此操作。

12. 单击复选标记以捕获映像。新映像现在将显示在**“映像”**下。

     ![成功捕获映像](./media/virtual-machines-windows-classic-capture-image/VMCapturedImageAvailable.png)

##后续步骤

该映像已就绪，可用于创建虚拟机了。为此，你将通过使用**“从库中”**菜单项并选择你刚创建的映像来创建一个虚拟机。有关说明，请参阅[从映像创建虚拟机](./virtual-machines-windows-classic-createportal.md)。

[如何登录到运行 Windows Server 的虚拟机]: ./virtual-machines-windows-classic-connect-logon.md
[如何使用 Sysprep：简介]: http://technet.microsoft.com/zh-cn/library/bb457073.aspx
[Run Sysprep.exe]: ./media/virtual-machines-capture-image-windows-server/SysprepCommand.png
[Enter Sysprep.exe options]: ./media/virtual-machines-windows-classic-capture-image/SysprepGeneral.png
[The virtual machine is stopped]: ./media/virtual-machines-capture-image-windows-server/SysprepStopped.png
[Capture an image of the virtual machine]: ./media/virtual-machines-windows-classic-capture-image/CaptureVM.png
[Enter the image name]: ./media/virtual-machines-capture-image-windows-server/Capture.png
[Image capture successful]: ./media/virtual-machines-capture-image-windows-server/CaptureSuccess.png
[Use the captured image]: ./media/virtual-machines-capture-image-windows-server/MyImagesWindows.png

<!---HONumber=Mooncake_1121_2016-->