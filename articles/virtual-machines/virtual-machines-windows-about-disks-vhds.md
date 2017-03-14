---
title: 关于 Windows VM 的磁盘和 VHD | Azure
description: 了解 Azure 中 Windows 虚拟机磁盘和 VHD 的基础知识。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: timlt
editor: tysonn
tags: azure-resource-manager,azure-service-management

ms.assetid: 0142c64d-5e8c-4d62-aa6f-06d6261f485a
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 11/18/2016
wacn.date: 01/20/2017
ms.author: cynthn
---

# 关于 Azure 虚拟机的磁盘和 VHD
如其他任何计算机一样，Azure 中的虚拟机将磁盘用作存储操作系统、应用程序和数据的位置。所有 Azure 虚拟机都至少有两个磁盘，即 Windows 操作系统磁盘和临时磁盘。操作系统磁盘通过映像创建，操作系统磁盘和该映像都存储在 Azure 存储帐户中的虚拟硬盘 (VHD)。虚拟机还可以有一个或多个数据磁盘，而这些磁盘也存储为 VHD。本文也适用于 [Linux 虚拟机](./virtual-machines-linux-about-disks-vhds.md)。

[!INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-both-include.md)]

## 操作系统磁盘
每个虚拟机都附加了一个操作系统磁盘。默认情况下，它注册为 SATA 驱动器并标为 C: 盘。此磁盘的最大容量为 1023 GB。

## 临时磁盘
临时磁盘是自动为你创建的。临时磁盘默认标记为 D: 盘，用于存储 pagefile.sys。

临时磁盘的大小因虚拟机的大小而异。有关详细信息，请参阅 [Sizes for Windows virtual machines](./virtual-machines-windows-sizes.md)（Windows 虚拟机的大小）。

> [!WARNING]
不要在临时磁盘上存储数据。该磁盘为应用程序和进程提供临时存储空间，只用于存储页面文件或交换文件等数据。若要将此磁盘重新映射到其他驱动器号，请参阅 [Change the drive letter of the Windows temporary disk](./virtual-machines-windows-classic-change-drive-letter.md)（更改 Windows 临时磁盘的驱动器号）。
> 
> 

有关 Azure 如何使用临时磁盘的详细信息，请参阅 [Understanding the temporary drive on Azure Virtual Machines](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)（了解 Azure 虚拟机上的临时驱动器）

## 数据磁盘
数据磁盘是附加到虚拟机的 VHD，用于存储应用程序数据或其他需要保留的数据。数据磁盘注册为 SCSI 驱动器并且带有所选择的字母标记。每个数据磁盘的最大容量为 1023 GB。虚拟机的大小决定可附加的磁盘数量，以及可用于托管磁盘的存储类型。

> [!NOTE]
有关虚拟机容量的详细信息，请参阅 [Windows 虚拟机的大小](./virtual-machines-windows-sizes.md)。
> 
> 

在通过映像创建虚拟机时，Azure 将会创建操作系统磁盘。如果你使用包含数据磁盘的映像，则 Azure 还会在创建虚拟机时创建数据磁盘。否则，你需要在创建虚拟机后添加数据磁盘。

随时可以将数据磁盘添加到虚拟机，只需将该磁盘**附加**到虚拟机即可。你可以使用已上载或复制到存储帐户的 VHD，也可以让 Azure 为你创建 VHD。附加数据磁盘会将 VHD 文件与 VM 关联，方法是在 VHD 上放置“租约”，因此在仍附加 VHD 时无法从存储中删除它。

## 关于 VHD
Azure 中使用的 VHD 是在 Azure 的标准或高级存储帐户中作为页 Blob 存储的 .vhd 文件。有关页 Blob 的详细信息，请参阅[了解块 Blob 和页 Blob](https://msdn.microsoft.com/zh-cn/library/ee691964.aspx)。有关高级存储的详细信息，请参阅[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](../storage/storage-premium-storage.md)。

Azure 支持固定的磁盘 VHD 格式。固定格式在文件内将逻辑磁盘以线性方式布局，使磁盘偏移量 X 存储在 Blob 偏移量 X 的位置。在 Blob 末尾有一小段脚注，描述了 VHD 的属性。通常，由于大多数磁盘中都有较大的未使用区域，因此固定格式会浪费空间。但是，Azure 以稀疏格式存储 .vhd 文件，使用户可兼获固定和动态格式磁盘的优点。有关详细信息，请参阅[虚拟硬盘入门](https://technet.microsoft.com/zh-cn/library/dd979539.aspx)。

Azure 中所有要用作创建磁盘或映像的源的 .vhd 文件都是只读文件。在创建磁盘或映像时，Azure 将生成 .vhd 文件的副本。这些副本可以是只读文件，也可以是读写文件，具体取决于使用 VHD 的方式。

当你基于映像创建虚拟机时，Azure 将为虚拟机创建磁盘，该磁盘是源 .vhd 文件的副本。为防止意外删除，Azure 对任何用于创建映像、操作系统磁盘或数据磁盘的源 .vhd 文件都设置了租约。

在删除源 .vhd 文件之前，需要先通过删除磁盘或映像来解除租约。可以通过删除虚拟机并删除所有关联的磁盘，一次性删除虚拟机、操作系统磁盘和源 .vhd 文件。但是，删除用作数据磁盘来源的 .vhd 文件需要按一定顺序执行几个步骤。首先从虚拟机分离该磁盘，再删除该磁盘，然后才能删除 .vhd 文件。

> [!WARNING]
如果从存储中删除了源 .vhd 文件或删除了存储帐户，Microsoft 则无法为用户恢复数据。
> 
> 

## 在标准存储中使用 TRIM

如果使用标准存储 (HDD)，应启用 TRIM。TRIM 会丢弃磁盘上未使用的块，使用户只需为实际使用的存储付费。如果创建了较大的文件，然后将其删除，这样可以节省成本。

可以运行此命令来检查 TRIM 设置。在 Windows VM 上打开命令提示符，然后键入：

```
fsutil behavior query DisableDeleteNotify
```

如果该命令返回 0，则表示正确启用了 TRIM。如果返回 1，请运行以下命令启用 TRIM：

```
fsutil behavior set DisableDeleteNotify 0
```

## 后续步骤
* [附加磁盘](./virtual-machines-windows-attach-disk-portal.md)可为 VM 添加额外的存储。
* [将 Windows VM 映像上载到 Azure](./virtual-machines-windows-upload-image.md)，以便在创建新的 VM 时使用。
* [更改 Windows 临时磁盘的驱动器号](./virtual-machines-windows-classic-change-drive-letter.md)，使应用程序能够将 D: 盘用于数据。

<!---HONumber=Mooncake_0116_2017-->
<!--Update_Description: update meta properties & wording update & add support for TRIM-->