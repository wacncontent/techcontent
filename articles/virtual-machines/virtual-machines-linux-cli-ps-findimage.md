---
title: 导航和选择 Linux VM 映像 | Azure
description: 了解在使用资源管理器部署模型创建 Azure 虚拟机时如何确定映像的确定发布者、产品和 SKU。
services: virtual-machines-linux
documentationCenter: ''
authors: squillace
manager: timlt
editor: ''
tags: azure-resource-manager

ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/23/2016
wacn.date: 10/25/2016
ms.author: rasquill
---

# 使用 Azure CLI 来选择 Linux 虚拟机映像

> [!NOTE]
> Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](../azure-resource-manager/resource-manager-deployment-model.md)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。

## 常用 Linux 映像表

| PublisherName | 产品 | SKU |
|:---------------------------------|:-------------------------------------------|:---------------------------------|:--------------------|
| credativ                         | Debian                                     | 8                                | 
| SUSE                             | openSUSE                                   | 13.2                             |
| SUSE                             | SLES                                       | 12-SP1                           |
| OpenLogic                        | CentOS                                     | 7.1                              |
| Canonical                        | UbuntuServer                               | 14.04.3-LTS                      |
| CoreOS                           | CoreOS                                     | Stable                           |

[!INCLUDE [virtual-machines-common-cli-ps-findimage](../../includes/virtual-machines-common-cli-ps-findimage.md)]

<!---HONumber=Mooncake_0118_2016-->