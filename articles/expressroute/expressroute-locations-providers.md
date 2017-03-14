---
title: ExpressRoute 位置 | Azure
description: 本文详细介绍了提供服务的区域以及如何连接到 Azure 区域。
services: expressroute
documentationCenter: na
authors: cherylmc
manager: carmonm
editor: ''

ms.service: expressroute
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/12/2017
wacn.date: 02/24/2017
ms.author: cherylmc
---

# ExpressRoute 合作伙伴和对等位置

本文中的表格提供有关 ExpressRoute 连接服务提供商、ExpressRoute 地理覆盖范围、通过 ExpressRoute 支持的 Azure 云服务以及 ExpressRoute 系统集成商 \(SI\) 的信息。

## <a name="partners"></a>ExpressRoute 连接服务提供商

所有 Azure 区域和位置都支持 ExpressRoute。以下地图提供了 Azure 区域和 ExpressRoute 位置的列表。ExpressRoute 位置是指 Microsoft 与多个服务提供商对等互连的位置。

![位置地图][0]

如果你至少与地缘政治区域内的一个 ExpressRoute 位置连接，你将有权访问地缘政治区域内所有区域中的 Azure 服务。

### 地缘政治区域内 ExpressRoute 位置与 Azure 区域的映射
下表提供了地缘政治区域内 ExpressRoute 位置与 Azure 区域的映射。

| **地缘政治区域** | **Azure 区域** | **ExpressRoute 位置** |
| --- | --- | --- |
| **中国** |中国北部、中国东部 |北京、上海 |

标准 ExpressRoute SKU 不支持跨地缘政治区域的连接。需要启用 ExpressRoute 高级外接程序以支持全球连接。

## <a name="locations"></a>连接服务提供商位置

| **位置** | **服务提供商** |
| --- | --- |
| **北京** |中国电信 |
| **上海** |中国电信 |

若要了解详细信息，请参阅[位于中国的 ExpressRoute](https://www.azure.cn/home/features/expressroute/)。 

## 后续步骤

- 有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 常见问题](./expressroute-faqs.md)。
- 确保符合所有先决条件。请参阅 [ExpressRoute 先决条件](./expressroute-prerequisites.md)。

<!--Image References-->
[0]: ./media/expressroute-locations/expressroute-locations-map.png "位置地图"

<!---HONumber=Mooncake_0220_2017-->