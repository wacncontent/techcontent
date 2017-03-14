---
title: 使用 Azure CLI 控制路由和虚拟设备 | Azure
description: 了解如何使用 Azure CLI 控制路由和虚拟设备。
services: virtual-network
documentationcenter: na
author: jimdial
manager: carmonm
editor: ''
tags: azure-resource-manager

ms.assetid: 5452a0b8-21a6-4699-8d6a-e2d8faf32c25
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/15/2016
wacn.date: 12/26/2016
ms.author: jdial
---

# 使用 Azure CLI 创建用户定义的路由 (UDR)
> [!div class="op_single_selector"]
- [PowerShell](./virtual-network-create-udr-arm-ps.md)
- [Azure CLI](./virtual-network-create-udr-arm-cli.md)
- [模板](./virtual-network-create-udr-arm-template.md)
- [PowerShell（经典）](./virtual-network-create-udr-classic-ps.md)
- [CLI（经典）](./virtual-network-create-udr-classic-cli.md)

[!INCLUDE [virtual-network-create-udr-intro-include.md](../../includes/virtual-network-create-udr-intro-include.md)]

> [!IMPORTANT]
在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：Azure Resource Manager 部署模型和经典部署模型。在使用任何 Azure 资源前，请确保了解[部署模型和工具](../azure-resource-manager/resource-manager-deployment-model.md)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍 Resource Manager 部署模型。还可在[经典部署模型](./virtual-network-create-udr-classic-cli.md)中创建 UDR。

[!INCLUDE [virtual-network-create-udr-scenario-include.md](../../includes/virtual-network-create-udr-scenario-include.md)]

以下示例 Azure CLI 命令需要基于以上方案创建的简单环境。若要运行本文档中所显示的命令，请首先通过部署[此模板](http://github.com/telmosampaio/azure-templates/tree/master/IaaS-NSG-UDR-Before)构建测试环境。下载该模板，并作必要的修改，然后通过 Azure CLI 发布。

>[!NOTE]
> 你从 GitHub 仓库 "azure-quickstart-templates" 中下载的模板，需要做一些修改才能适用于 Azure 中国云环境。例如，替换一些终结点 -- "blob.core.windows.net" 替换成 "blob.core.chinacloudapi.cn"，"cloudapp.azure.com" 替换成 "chinacloudapp.cn"；改掉一些不支持的 VM 映像，还有，改掉一些不支持的 VM 大小。

[!INCLUDE [azure-cli-prerequisites-include.md](../../includes/azure-cli-prerequisites-include.md)]

## 为前端子网创建 UDR
若要根据上述方案为前端子网创建所需的路由表和路由，请按照下面的步骤操作。

1. 运行以下命令，为前端子网创建路由表：

    ```
    azure network route-table create -g TestRG -n UDR-FrontEnd -l chinanorth
    ```

    输出：

    ```
    info:    Executing command network route-table create
    info:    Looking up route table "UDR-FrontEnd"
    info:    Creating route table "UDR-FrontEnd"
    info:    Looking up route table "UDR-FrontEnd"
    data:    Id                              : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/
    routeTables/UDR-FrontEnd
    data:    Name                            : UDR-FrontEnd
    data:    Type                            : Microsoft.Network/routeTables
    data:    Location                        : chinanorth
    data:    Provisioning state              : Succeeded
    info:    network route-table create command OK
    ```

    参数：

   * **-g（或 --resource-group）**。要在其中创建 UDR 的资源组的名称。对于我们的方案，为 *TestRG* 。
   * **-l（或 --location）**。要在其中创建新的 UDR 的 Azure 区域。对于我们的方案，为 *chinanorth* 。
   * **-n（或 --name）**。新 UDR 的名称。对于我们的方案，为 *UDR-FrontEnd* 。
2. 运行以下命令，在路由表中创建路由，将流向后端子网 (192.168.2.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)：

    ```
    azure network route-table route create -g TestRG -r UDR-FrontEnd -n RouteToBackEnd -a 192.168.2.0/24 -y VirtualAppliance -p 192.168.0.4
    ```

    输出：

    ```
    info:    Executing command network route-table route create
    info:    Looking up route "RouteToBackEnd" in route table "UDR-FrontEnd"
    info:    Creating route "RouteToBackEnd" in a route table "UDR-FrontEnd"
    info:    Looking up route "RouteToBackEnd" in route table "UDR-FrontEnd"
    data:    Id                              : /subscriptions/[Subscription Id]/TestRG/providers/Microsoft.Network/
    routeTables/UDR-FrontEnd/routes/RouteToBackEnd
    data:    Name                            : RouteToBackEnd
    data:    Provisioning state              : Succeeded
    data:    Next hop type                   : VirtualAppliance
    data:    Next hop IP address             : 192.168.0.4
    data:    Address prefix                  : 192.168.2.0/24
    info:    network route-table route create command OK
    ```

    参数：

   * **-r（或 --route-table-name）**。要添加路由的路由表的名称。对于我们的方案，为 *UDR-FrontEnd* 。
   * **-a（或 --address-prefix）**。数据包目标子网的地址前缀。对于我们的方案，为 *192.168.2.0/24* 。
   * **-y（或 --next-hop-type）**。要将流量发送到的对象的类型。可能的值为 *VirtualAppliance* 、 *VirtualNetworkGateway* 、 *VNETLocal* 、 *Internet* 或 *None* 。
   * **-p（或 --next-hop-ip-address**）。下一个跃点的 IP 地址。对于我们的方案，为 *192.168.0.4* 。
3. 运行以下命令，将上面创建的路由表与 **FrontEnd** 子网关联：

    ```
    azure network vnet subnet set -g TestRG -e TestVNet -n FrontEnd -r UDR-FrontEnd
    ```

    输出：

    ```
    info:    Executing command network vnet subnet set
    info:    Looking up the subnet "FrontEnd"
    info:    Looking up route table "UDR-FrontEnd"
    info:    Setting subnet "FrontEnd"
    info:    Looking up the subnet "FrontEnd"
    data:    Id                              : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/
    virtualNetworks/TestVNet/subnets/FrontEnd
    data:    Type                            : Microsoft.Network/virtualNetworks/subnets
    data:    ProvisioningState               : Succeeded
    data:    Name                            : FrontEnd
    data:    Address prefix                  : 192.168.1.0/24
    data:    Network security group          : [object Object]
    data:    Route Table                     : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/
    routeTables/UDR-FrontEnd
    data:    IP configurations:
    data:      /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB1/ipConf
    igurations/ipconfig1
    data:      /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB2/ipConf
    igurations/ipconfig1
    data:    
    info:    network vnet subnet set command OK
    ```

    参数：

   * **-e（或 --vnet-name）**。子网所在的 VNet 的名称。对于我们的方案，为 *TestVNet* 。

## 为后端子网创建 UDR
若要根据上述方案为后端子网创建所需的路由表和路由，请完成以下步骤：

1. 运行以下命令，为后端子网创建路由表：

    ```
    azure network route-table create -g TestRG -n UDR-BackEnd -l chinanorth
    ```

2. 运行以下命令，在路由表中创建路由，将流向前端子网 (192.168.1.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)：

    ```
    azure network route-table route create -g TestRG -r UDR-BackEnd -n RouteToFrontEnd -a 192.168.1.0/24 -y VirtualAppliance -p 192.168.0.4
    ```

3. 运行以下命令，将路由表与 **BackEnd** 子网关联：

    ```
    azure network vnet subnet set -g TestRG -e TestVNet -n BackEnd -r UDR-BackEnd
    ```

## 在 FW1 上启用 IP 转发
若要在 **FW1** 使用的 NIC 中启用 IP 转发，请完成以下步骤：

1. 运行以下命令，并注意“启用 IP 转发”的值。应将它设置为 *false* 。

    ```
    azure network nic show -g TestRG -n NICFW1
    ```

    输出：

    ```
    info:    Executing command network nic show
    info:    Looking up the network interface "NICFW1"
    data:    Id                              : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/
    networkInterfaces/NICFW1
    data:    Name                            : NICFW1
    data:    Type                            : Microsoft.Network/networkInterfaces
    data:    Location                        : chinanorth
    data:    Provisioning state              : Succeeded
    data:    MAC address                     : 00-0D-3A-30-95-B3
    data:    Enable IP forwarding            : false
    data:    Tags                            : displayName=NetworkInterfaces - DMZ
    data:    Virtual machine                 : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/
    virtualMachines/FW1
    data:    IP configurations:
    data:      Name                          : ipconfig1
    data:      Provisioning state            : Succeeded
    data:      Public IP address             : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/
    publicIPAddresses/PIPFW1
    data:      Private IP address            : 192.168.0.4
    data:      Private IP Allocation Method  : Static
    data:      Subnet                        : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/
    virtualNetworks/TestVNet/subnets/DMZ
    data:    
    info:    network nic show command OK
    ```
2. 运行以下命令，启用 IP 转发：

    ```
    azure network nic set -g TestRG -n NICFW1 -f true
    ```

    输出：

    ```
    info:    Executing command network nic set
    info:    Looking up the network interface "NICFW1"
    info:    Updating network interface "NICFW1"
    info:    Looking up the network interface "NICFW1"
    data:    Id                              : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/
    networkInterfaces/NICFW1
    data:    Name                            : NICFW1
    data:    Type                            : Microsoft.Network/networkInterfaces
    data:    Location                        : chinanorth
    data:    Provisioning state              : Succeeded
    data:    MAC address                     : 00-0D-3A-30-95-B3
    data:    Enable IP forwarding            : true
    data:    Tags                            : displayName=NetworkInterfaces - DMZ
    data:    Virtual machine                 : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/
    virtualMachines/FW1
    data:    IP configurations:
    data:      Name                          : ipconfig1
    data:      Provisioning state            : Succeeded
    data:      Public IP address             : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/
    publicIPAddresses/PIPFW1
    data:      Private IP address            : 192.168.0.4
    data:      Private IP Allocation Method  : Static
    data:      Subnet                        : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/
    virtualNetworks/TestVNet/subnets/DMZ
    data:    
    info:    network nic set command OK
    ```

    参数：

   * **-f（或 --enable-ip-forwarding）**。 *true* 或 *false* 。

<!---HONumber=Mooncake_1219_2016-->