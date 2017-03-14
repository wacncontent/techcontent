---
title: 登录到经典 Azure VM | Azure
description: 使用 Azure 经典管理门户登录到使用经典部署模型创建的 Windows 虚拟机。
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
ms.date: 07/28/2016
wacn.date: 01/05/2017
ms.author: cynthn
---

# 使用 Azure 经典管理门户登录到 Windows 虚拟机

在 Azure 经典管理门户中，使用“连接”按钮启动远程桌面会话，然后登录到 Windows VM。

是否要连接到 Linux VM？ 请参阅[如何登录到运行 Linux 的虚拟机](./virtual-machines-linux-mac-create-ssh-keys.md)。

了解如何[使用新版 Azure 门户预览执行这些步骤](./virtual-machines-windows-connect-logon.md)。

[!INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-classic-include.md)]

关于登录到资源管理器模型的 VM，请点击[这里](./virtual-machines-windows-connect-logon.md).

## 连接到虚拟机

1. 登录到 Azure 经典管理门户。

2. 单击“虚拟机”，然后选择虚拟机。

3. 在页面底部的命令栏中，单击“连接”。

    ![登录到虚拟机](./media/virtual-machines-windows-classic-connect-logon/connectwindows.png)

> [!TIP]
> 如果“连接”按钮不可用，请参阅本文末尾的故障排除提示。

## 登录到虚拟机

[!INCLUDE [virtual-machines-log-on-win-server](../../includes/virtual-machines-log-on-win-server.md)]

## 后续步骤

- 如果“连接”按钮处于非活动状态或者你在使用远程桌面连接时遇到其他问题，可尝试重置配置。在虚拟机仪表板的“速览”下，单击“重置远程配置”。
- 如果密码出现问题，可尝试重置密码。在虚拟机仪表板的“速览”下，单击“重置密码”。

如果这些提示不起作用或不是你所需的内容，请参阅[对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除](./virtual-machines-windows-troubleshoot-rdp-connection.md)。此文将指导你完成诊断和解决常见问题。

<!---HONumber=Mooncake_0905_2016-->