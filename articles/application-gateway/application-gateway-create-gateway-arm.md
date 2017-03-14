---
title: 使用 Azure Resource Manager 创建、启动或删除应用程序网关 | Azure
description: 本页提供有关使用 Azure Resource Manager 创建、配置、启动和删除 Azure 应用程序网关的说明
documentationCenter: na
services: application-gateway
authors: georgewallace
manager: carmonm
editor: tysonn

ms.service: application-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/12/2016
wacn.date: 01/25/2017
ms.author: gwallace
---

# 使用 Azure Resource Manager 创建、启动或删除应用程序网关
> [!div class="op_single_selector"]
- [Azure 门户预览](./application-gateway-create-gateway-portal.md)
- [Azure Resource Manager PowerShell](./application-gateway-create-gateway-arm.md)
- [Azure 经典 PowerShell](./application-gateway-create-gateway.md)
- [Azure Resource Manager 模板](./application-gateway-create-gateway-arm-template.md)
- [Azure CLI](./application-gateway-create-gateway-cli.md)

Azure 应用程序网关是第 7 层负载均衡器。它在不同服务器之间提供故障转移和性能路由 HTTP 请求，而不管它们是在云中还是本地。应用程序网关提供许多应用程序传送控制器 (ADC) 功能，包括 HTTP 负载均衡、基于 cookie 的会话相关性、安全套接字层 (SSL) 卸载、自定义运行状况探测、多站点支持，以及许多其他功能。若要查找支持的功能的完整列表，请参阅[应用程序网关概述](./application-gateway-introduction.md)

本文将指导你完成创建、配置、启动和删除应用程序网关的步骤。

> [!IMPORTANT]
在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：Resource Manager 部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](../azure-classic-rm.md)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文档介绍如何使用 Azure Resource Manager 创建应用程序网关。若要使用经典版本，请转到[使用 PowerShell 创建应用程序网关经典部署](./application-gateway-create-gateway.md)。
> 
> 

## 开始之前

1. 使用 Web 平台安装程序安装最新版本的 Azure PowerShell cmdlet。可以从“[下载](/downloads/)”页的“Windows PowerShell”部分下载并安装最新版本。
2. 如果有现有的虚拟网络，请选择现有的一个空子网，或者在现有虚拟网络中创建一个子网，专门供应用程序网关使用。应用程序网关部署到的虚拟网络必须与要部署在应用程序网关后面的资源相同。
3. 必须存在配置为使用应用程序网关的服务器，或者必须在虚拟网络中为其创建终结点，或者必须为其分配公共 IP/VIP。

## 创建应用程序网关需要什么？

* **后端服务器池：**后端服务器的 IP 地址列表。列出的 IP 地址应属于虚拟网络子网，或者是公共 IP/VIP。
* **后端服务器池设置：**每个池都有一些设置，例如端口、协议和基于 Cookie 的关联性。这些设置绑定到池，并会应用到池中的所有服务器。
* **前端端口：**此端口是应用程序网关上打开的公共端口。流量将抵达此端口，然后重定向到后端服务器之一。
* **侦听器：**侦听器具有前端端口、协议（Http 或 Https，这些值区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。
* **规则：**规则将会绑定侦听器和后端服务器池，并定义当流量抵达特定侦听器时应定向到的后端服务器池。

## 创建应用程序网关

使用 Azure 经典部署和 Azure Resource Manager 部署的差别在于创建应用程序网关的顺序和需要配置的项。

使用 Resource Manager 时，组成应用程序网关的所有项都将分开配置，然后放在一起创建应用程序网关资源。

以下是创建应用程序网关所需执行的步骤。

## 创建资源管理器的资源组

确保使用最新版本的 Azure PowerShell。[将 Windows PowerShell 与 Resource Manager 配合使用](../azure-resource-manager/powershell-azure-resource-manager.md)中提供了详细信息。

### 步骤 1

登录 Azure

```
Login-AzureRmAccount -EnvironmentName AzureChinaCloud
```

系统会提示使用凭据进行身份验证。

### 步骤 2

检查该帐户的订阅。

```
Get-AzureRmSubscription
```

### 步骤 3

选择要使用的 Azure 订阅。

```
Select-AzureRmSubscription -Subscriptionid "GUID of subscription"
```

### 步骤 4

创建资源组（如果要使用现有的资源组，请跳过此步骤）。

```
New-AzureRmResourceGroup -Name appgw-rg -Location "China North"
```

Azure 资源管理器要求所有资源组指定一个位置。此位置将用作该资源组中的资源的默认位置。请确保用于创建应用程序网关的所有命令都使用相同的资源组。

在上面的示例中，我们在位置“中国北部”创建了名为“appgw-RG”的资源组。

> [!NOTE]
如果你需要为应用程序网关配置自定义探测，请参阅 [Create an application gateway with custom probes by using PowerShell](./application-gateway-create-probe-ps.md)（使用 PowerShell 创建带自定义探测的应用程序网关）。有关详细信息，请查看 [custom probes and health monitoring](./application-gateway-probe-overview.md)（自定义探测和运行状况监视）。
> 
> 

## 为应用程序网关创建虚拟网络和子网

以下示例演示如何使用 Resource Manager 创建虚拟网络。

### 步骤 1

将地址范围 10.0.0.0/24 分配给用于创建虚拟网络的子网变量。

```
$subnet = New-AzureRmVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24
```

### 步骤 2

使用前缀 10.0.0.0/16 和子网 10.0.0.0/24，在中国北部区域的“appgw-rg”资源组中创建名为“appgwvnet”的虚拟网络。

```
$vnet = New-AzureRmVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-rg -Location "China North" -AddressPrefix 10.0.0.0/16 -Subnet $subnet
```

### 步骤 3

分配子网变量，以完成后面的创建应用程序网关的步骤。

```
$subnet=$vnet.Subnets[0]
```

## 创建前端配置的公共 IP 地址

在中国北部区域的“appgw-rg”资源组中创建公共 IP 资源“publicIP01”。

```
$publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-rg -name publicIP01 -location "China North" -AllocationMethod Dynamic
```

## 创建应用程序网关配置对象

在创建应用程序网关前，必须设置所有配置项。以下步骤将创建应用程序网关资源所需的配置项。

### 步骤 1

创建名为“gatewayIP01”的应用程序网关 IP 配置。应用程序网关启动时，它会从配置的子网获取 IP 地址，再将网络流量路由到后端 IP 池中的 IP 地址。请记住，每个实例需要一个 IP 地址。

```
$gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet
```

### 步骤 2

配置名为“pool01”的后端 IP 地址池，其 IP 地址为“134.170.185.46”、“134.170.188.221”、“134.170.185.50”。这些 IP 地址是接收来自前端 IP 终结点的网络流量的 IP 地址。替换上述 IP 地址，添加自己的应用程序 IP 地址终结点。

```
$pool = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221,134.170.185.50
```

### 步骤 3

为后端池中进行了负载均衡的网络流量配置应用程序网关设置“poolsetting01”。

```
$poolSetting = New-AzureRmApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Disabled
```

### 步骤 4

为公共 IP 终结点配置名为“frontendport01”的前端 IP 端口。

```
$fp = New-AzureRmApplicationGatewayFrontendPort -Name frontendport01  -Port 80
```

### 步骤 5

创建名为“fipconfig01”的前端 IP 配置，并将公共 IP 地址与前端 IP 配置相关联。

```
$fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name fipconfig01 -PublicIPAddress $publicip
```

### 步骤 6

创建名为“listener01”的侦听器，并将前端端口与前端 IP 配置相关联。

```
$listener = New-AzureRmApplicationGatewayHttpListener -Name listener01  -Protocol Http -FrontendIPConfiguration $fipconfig -FrontendPort $fp
```

### 步骤 7

创建名为“rule01”的负载均衡器路由规则，并配置负载均衡器的行为。

```
$rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool
```

### 步骤 8

配置应用程序网关的实例大小。

```
$sku = New-AzureRmApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2
```

> [!NOTE]
**InstanceCount** 的默认值为 2，最大值为 10。**GatewaySize** 的默认值为 Medium。可以在 **Standard\_Small**、**Standard\_Medium** 和 **Standard\_Large** 之间进行选择。
> 
> 

## 使用 New-AzureRmApplicationGateway 创建应用程序网关

创建包含前述步骤中所有配置项的应用程序网关。示例中的应用程序网关名为“appgwtest”。

```
$appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location "China North" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku
```

### 步骤 9
从附加到应用程序网关的公共 IP 资源中检索应用程序网关的 DNS 和 VIP 详细信息。

```
Get-AzureRmPublicIpAddress -Name publicIP01 -ResourceGroupName appgw-rg  
```

## 删除应用程序网关

若要删除应用程序网关，请执行以下步骤：

### 步骤 1

获取应用程序网关对象，并将其关联到变量 `$getgw`。

```
$getgw = Get-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg
```

### 步骤 2

使用 `Stop-AzureRmApplicationGateway` 停止应用程序网关。

```
Stop-AzureRmApplicationGateway -ApplicationGateway $getgw  
```

应用程序网关进入停止状态后，请使用 `Remove-AzureRmApplicationGateway` cmdlet 删除该服务。

```
Remove-AzureRmApplicationGateway -Name $appgwtest -ResourceGroupName appgw-rg -Force
```

> [!NOTE]
可以使用 **-force** 开关来禁止显示该删除的确认消息。
> 
> 

若要验证是否已删除服务，可以使用 `Get-AzureRmApplicationGateway` cmdlet。此步骤不是必需的。

```
Get-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg
```

## 获取应用程序网关 DNS 名称

创建网关后，下一步是配置用于通信的前端。使用公共 IP 时，应用程序网关需要动态分配的 DNS 名称，这会造成不方便。若要确保最终用户能够访问应用程序网关，可以使用指向应用程序网关的公共终结点的 CNAME 记录。为此，可使用附加到应用程序网关的 PublicIPAddress 元素检索应用程序网关及其关联的 IP/DNS 名称的详细信息。应使用应用程序网关的 DNS 名称来创建 CNAME 记录，使两个 Web 应用程序都指向此 DNS 名称。不建议使用 A 记录，因为重新启动应用程序网关后 VIP 可能会变化。

```
Get-AzureRmPublicIpAddress -ResourceGroupName appgw-RG -Name publicIP01
```


```
Name                     : publicIP01
ResourceGroupName        : appgw-RG
Location                 : chinanorth
Id                       : /subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/publicIPAddresses/publicIP01
Etag                     : W/"00000d5b-54ed-4907-bae8-99bd5766d0e5"
ResourceGuid             : 00000000-0000-0000-0000-000000000000
ProvisioningState        : Succeeded
Tags                     : 
PublicIpAllocationMethod : Dynamic
IpAddress                : xx.xx.xxx.xx
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : {
                                "Id": "/subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/applicationGateways/appgwtest/frontendIP
                            Configurations/frontend1"
                            }
DnsSettings              : {
                                "Fqdn": "00000000-0000-xxxx-xxxx-xxxxxxxxxxxx.chinacloudapp.cn"
                            }
```

## 后续步骤

如果你要配置 SSL 卸载，请参阅 [Configure an application gateway for SSL offload](./application-gateway-ssl.md)（配置应用程序网关以进行 SSL 卸载）。

如果你想要将应用程序网关配置为与内部负载均衡器配合使用，请参阅 [Create an application gateway with an internal load balancer (ILB)](./application-gateway-ilb.md)（创建具有内部负载均衡器 (ILB) 的应用程序网关）。

如需负载均衡选项的其他常规信息，请参阅：

* [Azure Load Balancer](../load-balancer/index.md)
* [Azure 流量管理器](../traffic-manager/index.md)

<!---HONumber=Mooncake_Quality_Review_1230_2016-->