---
title: VM 的常用 PowerShell 命令 | Azure
description: 可用于在 Windows 设备上的 Azure 中创建和管理 VM 的常用 PowerShell 命令
services: virtual-machines-windows
documentationCenter: ''
authors: davidmu1
manager: timlt
editor: tysonn
tags: azure-resource-manager

ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 09/27/2016
wacn.date: 01/05/2017
ms.author: davidmu
---

# 创建和管理 VM 的常用 PowerShell 命令

本文介绍可用于在 Azure 订阅中创建和管理虚拟机的一些 Azure PowerShell 命令。若要获取特定命令行开关和选项的详细帮助，可以使用 **Get-Help** *命令*。

有关安装最新版本的 Azure PowerShell、选择订阅和登录帐户的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

## 创建 VM

任务 | 命令
-------------- | -------------------------
创建 VM 配置 | $vm = [New-AzureRmVMConfig](https://msdn.microsoft.com/zh-cn/library/mt603727.aspx) -VMName "vm\_name" -VMSize "vm\_size"<BR></BR><BR></BR>VM 配置用于定义或更新 VM 的设置。使用 VM 的名称及其[大小](./virtual-machines-windows-sizes.md)对配置进行初始化。
添加配置设置 | $vm = [Set-AzureRmVMOperatingSystem](https://msdn.microsoft.com/zh-cn/library/mt603843.aspx) -VM $vm -Windows -ComputerName "computer\_name" -Credential $cred -ProvisionVMAgent -EnableAutoUpdate<BR></BR><BR></BR>将包含[凭据](https://technet.microsoft.com/zh-cn/library/hh849815.aspx)的操作系统设置添加到你以前使用 New-AzureRmVMConfig 创建的配置对象。
添加网络接口 | $vm = [Add-AzureRmVMNetworkInterface](https://msdn.microsoft.com/zh-cn/library/mt619351.aspx) -VM $vm -Id $nic.Id<BR></BR><BR></BR>VM 必须具有[网络接口](./virtual-machines-windows-ps-create.md)才能在虚拟网络中通信。还可使用 [Get-AzureRmNetworkInterface](https://msdn.microsoft.com/zh-cn/library/mt619434.aspx) 检索现有网络接口对象。
指定平台映像 | $vm = [Set-AzureRmVMSourceImage](https://msdn.microsoft.com/zh-cn/library/mt619344.aspx) -VM $vm -PublisherName "publisher\_name" -Offer "publisher\_offer" -Skus "product\_sku" -Version "latest"<BR></BR><BR></BR>将[映像信息](./virtual-machines-windows-cli-ps-findimage.md)添加到以前使用 New-AzureRmVMConfig 创建的配置对象。仅当将操作系统磁盘设置为使用平台映像时，才使用此命令返回的对象。
设置要使用平台映像的操作系统磁盘 | $vm = [Set-AzureRmVMOSDisk](https://msdn.microsoft.com/zh-cn/library/mt603746.aspx) -VM $vm -Name "disk\_name" -VhdUri "http://mystore1.blob.core.chinacloudapi.cn/vhds/disk\_name.vhd" -CreateOption FromImage<BR></BR><BR></BR>将操作系统磁盘的名称及其在[存储](../storage/storage-powershell-guide-full.md)中的位置添加到以前创建的配置对象。
设置要使用一般化映像的操作系统磁盘 | $vm = Set-AzureRmVMOSDisk -VM $vm -Name "disk\_name" -SourceImageUri "https://mystore1.blob.core.chinacloudapi.cn/system/Microsoft.Compute/Images/myimages/myprefix-osDisk.{guid}.vhd" -VhdUri "https://mystore1.blob.core.chinacloudapi.cn/vhds/disk\_name.vhd" -CreateOption FromImage -Windows<BR></BR><BR></BR>将操作系统磁盘的名称、源映像的位置和磁盘在[存储](../storage/storage-powershell-guide-full.md)中的位置添加到配置对象。
设置要使用特殊化映像的操作系统磁盘 | $vm = Set-AzureRmVMOSDisk -VM $vm -Name "name\_of\_disk" -VhdUri "http://mystore1.blob.core.chinacloudapi.cn/vhds/" -CreateOption Attach -Windows
创建 VM | [New-AzureRmVM](https://msdn.microsoft.com/zh-cn/library/mt603754.aspx) -ResourceGroupName "resource\_group\_name" -Location "location\_name" -VM $vm<BR></BR><BR></BR>在[资源组](../azure-resource-manager/powershell-azure-resource-manager.md)中创建所有资源。必须在资源组所在[位置](https://msdn.microsoft.com/zh-cn/library/azure/dn495177.aspx)创建 VM。运行此命令之前，请运行 New-AzureRmVMConfig、Set-AzureRmVMOperatingSystem、Set-AzureRmVMSourceImage、Add-AzureRmVMNetworkInterface 和 Set-AzureRmVMOSDisk。

## 获取有关 VM 的信息

任务 | 命令
-------------- | -------------------------
列出订阅中的 VM| [Get-AzureRmVM](https://msdn.microsoft.com/zh-cn/library/mt603718.aspx)
列出资源组中的 VM | Get AzureRmVM ResourceGroupName"resource\_group\_name"<BR></BR><BR></BR>若要获取订阅中资源组的列表，请使用 [Get-AzureRmResourceGroup](https://msdn.microsoft.com/zh-cn/library/mt679016.aspx)。
获取有关 VM 的信息 | Get-AzureRmVM -ResourceGroupName "resource\_group\_name" -Name "vm\_name"

## 管理 VM

任务 | 命令
-------------- | -------------------------
启动 VM | [Start-AzureRmVM](https://msdn.microsoft.com/zh-cn/library/mt603453.aspx) -ResourceGroupName "resource\_group\_name" -Name "vm\_name"
停止 VM | [Stop-AzureRmVM](https://msdn.microsoft.com/zh-cn/library/mt603483.aspx) -ResourceGroupName "resource\_group\_name" -Name "vm\_name"
重启正在运行的 VM | [Restart-AzureRmVM](https://msdn.microsoft.com/zh-cn/library/mt603775.aspx) -ResourceGroupName "resource\_group\_name" -Name "vm\_name"
删除 VM | [Remove-AzureRmVM](https://msdn.microsoft.com/zh-cn/library/mt603641.aspx) -ResourceGroupName "resource\_group\_name" -Name "vm\_name"
对 VM 进行一般化 | [Set-AzureRmVm](https://msdn.microsoft.com/zh-cn/library/mt603688.aspx) -ResourceGroupName YourResourceGroup -Name "vm\_name" -Generalized<BR></BR><BR></BR>在运行 Save-AzureRmVMImage 之前，请先运行此命令。
捕获 VM | [Save-AzureRmVMImage](https://msdn.microsoft.com/zh-cn/library/mt619423.aspx) -ResourceGroupName "resource\_group\_name" -VMName "vm\_name" -DestinationContainerName "image\_container" -VHDNamePrefix "image\_name\_prefix" -Path "C:\\filepath\\filename.json"<BR></BR><BR></BR>必须将虚拟机[关闭并一般化](./virtual-machines-windows-generalize-vhd.md)，才能将其用于创建映像。运行此命令之前，请运行 Set-AzureRmVm。
更新 VM | [Update-AzureRmVM](https://msdn.microsoft.com/zh-cn/library/mt603662.aspx) -ResourceGroupName "resource\_group\_name" -VM $vm<BR></BR><BR></BR>使用 Get-AzureRmVM 获取当前 VM 配置，更改 VM 对象上的配置设置，然后运行此命令。
将数据磁盘添加到 VM | [Add-AzureRmVMDataDisk](https://msdn.microsoft.com/zh-cn/library/mt603673.aspx) -VM $vm -Name "disk\_name" -VhdUri "https://mystore1.blob.core.chinacloudapi.cn/vhds/disk\_name.vhd" -LUN # -Caching ReadWrite -DiskSizeinGB # -CreateOption Empty<BR></BR><BR></BR>使用 Get-AzureRmVM 获取 VM 对象。指定 LUN 号和磁盘大小。运行 Update-AzureRmVM 将配置更改应用到 VM。添加的磁盘未进行初始化。有关如何初始化添加的磁盘的信息，请参阅[使用 Resource Manager 和 PowerShell 管理 Azure 虚拟机](./virtual-machines-windows-ps-manage.md)。
从 VM 中删除数据磁盘 | [Remove-AzureRmVMDataDisk](https://msdn.microsoft.com/zh-cn/library/mt603614.aspx) -VM $vm -Name "disk\_name"<BR></BR><BR></BR>使用 Get-AzureRmVM 获取 VM 对象。运行 Update-AzureRmVM 将配置更改应用到 VM。
将扩展添加到 VM | [Set-AzureRmVMExtension](https://msdn.microsoft.com/zh-cn/library/mt603745.aspx) -ResourceGroupName "resource\_group\_name" -Location "azure\_location" -VMName "vm\_name" -Name "extension\_name" -Publisher "publisher\_name" -Type "extension\_type" -TypeHandlerVersion "#.#" -Settings $Settings -ProtectedSettings $ProtectedSettings<BR></BR><BR></BR>使用要安装的扩展的适当[配置信息](./virtual-machines-windows-extensions-configuration-samples.md)运行此命令。
删除 VM 扩展 | [Remove-AzureRmVMExtension](https://msdn.microsoft.com/zh-cn/library/mt603782.aspx) -ResourceGroupName "resource\_group\_name" -Name "extension\_name" -VMName "vm\_name"

## 后续步骤

- 有关创建虚拟机的基本步骤，请参阅[使用 Resource Manager 和 PowerShell 创建 Windows VM](./virtual-machines-windows-ps-create.md)。

<!---HONumber=Mooncake_Quality_Review_1202_2016-->