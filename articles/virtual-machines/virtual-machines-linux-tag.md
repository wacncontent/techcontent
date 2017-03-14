---
title: 如何标记 Linux 虚拟机 | Azure
description: 了解如何标记使用 Resource Manager 部署模型在 Azure 中创建的 Linux 虚拟机。
services: virtual-machines-linux
documentationCenter: ''
authors: mmccrory
manager: timlt
editor: tysonn
tags: azure-resource-manager

ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/05/2016
wacn.date: 12/26/2016
ms.author: memccror
---

# 如何在 Azure 中标记 Linux 虚拟机

本文介绍在 Azure 中通过 Resource Manager 部署模型标记 Linux 虚拟机的不同方式。标记是用户定义的键/值对，可直接放置在资源或资源组中。目前，Azure 支持每个资源和资源组最多 15 个标记。标记可以在创建时放置在资源中，也可以添加到现有资源中。请注意，只有通过 Resource Manager 部署模型创建的资源支持标记。

[!INCLUDE [virtual-machines-common-tag](../../includes/virtual-machines-common-tag.md)]

## 使用 Azure CLI 进行标记

还支持通过 Azure CLI 对已创建的资源进行标记。若要开始，请设置 [Azure CLI 环境](../azure-resource-manager/xplat-cli-azure-resource-manager.md)。通过 Azure CLI 登录到你的订阅，并切换到 Resource Manager 模式 (`azure config mode arm`)。

你可以使用以下命令查看给定虚拟机的所有属性，包括标记：

```
    azure vm show -g MyResourceGroup -n MyTestVM
```

若要通过 Azure CLI 添加新的 VM 标记，可以使用 `azure vm set` 命令以及标记参数 **-t**：

```
    azure vm set -g MyResourceGroup -n MyTestVM -t myNewTagName1=myNewTagValue1;myNewTagName2=myNewTagValue2
```

若要删除所有标记，可以在 `azure vm set` 命令中使用 **-T** 参数。

```
    azure vm set - g MyResourceGroup -n MyTestVM -T
```

既然我们已通过 Azure CLI 和门户预览将标记应用到资源中，那就让我们看一看使用情况详细信息，以在计费门户中的查看标记。

[!INCLUDE [virtual-machines-common-tag-usage](../../includes/virtual-machines-common-tag-usage.md)]

## 后续步骤

* 若要详细了解如何标记 Azure 资源，请参阅 [Azure Resource Manager Overview][]（Azure Resource Manager 概述）和 [Using Tags to organize your Azure Resources][]（使用标记来组织 Azure 资源）。
* 若要了解标记如何帮助你管理 Azure 资源的使用，请参阅 [Understanding your Azure Bill][]（了解你的 Azure 帐单）。

[Azure CLI 环境]: ../azure-resource-manager/xplat-cli-azure-resource-manager.md
[Azure Resource Manager Overview]: ../azure-resource-manager/resource-group-overview.md
[Using Tags to organize your Azure Resources]: ../azure-resource-manager/resource-group-using-tags.md
[Understanding your Azure Bill]: ../billing-understand-your-bill.md

<!---HONumber=Mooncake_Quality_Review_1215_2016-->