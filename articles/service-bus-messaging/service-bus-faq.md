---
title: 服务总线常见问题解答 | Azure
description: 回答了一些关于 Azure 服务总线的常见问题。
services: service-bus
documentationCenter: na
authors: sethmanheim
manager: ''
editor: ''

ms.service: service-bus
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/04/2016
ms.author: sethm;juconway
wacn.date: 12/26/2016
---

# 服务总线常见问题解答

本文回答了一些关于 Microsoft Azure 服务总线的常见问题。你还可以访问 [Azure 支持 FAQ](http://go.microsoft.com/fwlink/?LinkID=185083) 了解一般的 Azure 定价和支持信息。本文涵盖以下主题：

- [关于 Azure 服务总线消息传送的一般问题](#general-questions-about-azure-service-bus-messaging)
- [服务总线最佳实践](#service-bus-best-practices)
- [服务总线定价](#service-bus-pricing)
- [服务总线配额](#service-bus-quotas)
- [订阅和命名空间管理](#subscription-and-namespace-management)
- [故障排除](#service-bus-troubleshooting)

## <a name="general-questions-about-azure-service-bus-messaging"></a> 关于 Azure 服务总线消息传送的一般问题

### 什么是 Azure 服务总线消息传送？

[Azure 服务总线消息传送](./service-bus-messaging-overview.md)是一个异步消息传送云平台，可让你在分离的系统之间发送数据。Microsoft 以服务的形式提供 Azure 服务总线消息传送，这表示你不需要托管任何自有硬件就能使用它。

### 什么是服务总线命名空间？

命名空间提供了用于对应用程序中的服务总线资源进行寻址的范围容器。必须创建命名空间才能使用服务总线，而且这也是入门的第一步。

### 什么是 Azure 服务总线队列？

[服务总线队列](./service-bus-queues-topics-subscriptions.md)是用于存储消息的实体。当你有多个应用程序，或者多个需要彼此通信的分布式应用程序部分时，队列特别有用。队列和发行中心的相似之处在于，两者都会接收多个产品（消息），再从该处送出。

### 什么是 Azure 服务总线主题和订阅？

主题可以可视化为队列，而且在使用多个订阅时，主题会变成更丰富的消息传送模型；实质上是一对多的通信工具。此发布/订阅模型（或 *pub/sub*）可让应用程序将消息发送到具有多个订阅的主题，以便让多个应用程序接收该消息。

### 什么是分区实体？

传统的队列或主题由单个消息中转站进行处理并存储在一个消息存储中。[分区队列或主题](./service-bus-partitioning.md)由多个消息代理处理，并存储在多个消息传送存储中。这意味着分区的队列或主题的总吞吐量不再受到单个消息中转站或消息存储的性能所限制。此外，消息传送存储的临时中断不会导致分区的队列或主题不可用。

请注意，使用分区实体时不保证排序。如果某个分区不可用，你仍可从其他分区发送和接收消息。

## <a name="service-bus-best-practices"></a> 服务总线最佳实践

### Azure 服务总线的最佳实践有哪些？

-   [使用服务总线中转消息传送改进性能的最佳实践][] - 本文说明如何在交换中转消息时优化性能。

### 创建消息传送实体前的须知事项有哪些？
队列和主题的以下属性是固定不变的。预配实体时请考虑到这一点，因为若要修改属性，就必须创建新的替代实体。

-   大小

-   分区

-   会话

-   重复检测

-   快速实体

## <a name="service-bus-pricing"></a> 服务总线定价

本部分回答了一些关于服务总线定价结构的常见问题。你还可以访问 [Azure 支持常见问题](http://go.microsoft.com/fwlink/?LinkID=185083)了解一般的 Azure 定价信息。有关服务总线定价的完整信息，请参阅[服务总线定价](https://www.azure.cn/pricing/details/messaging/)。

### 服务总线如何收取费用？

有关服务总线定价的完整信息，请参阅[服务总线定价详细信息][Pricing overview]。除标示的价格外，你还需为在其中部署应用程序的数据中心之外的相关数据输出支付费用。

### 服务总线的哪些使用情况受数据传输限制？ 哪些不受其限制？

在给定的 Azure 区域内的任何数据传输均不收费。

### 服务总线是否对存储收费？

否，服务总线不对存储收费。但是，对每个队列/主题可以保留的最大数据量设有配额限制。请参阅下一个常见问题。

## <a name="service-bus-quotas"></a> 服务总线配额

有关服务总线限制和配额的列表，请参阅[配额概述][]。

### 服务总线是否有任何使用率配额？

默认情况下，对于任何云服务，Microsoft 每月都会设置聚合的使用率配额，此配额基于客户的所有订阅进行计算。由于我们清楚你可能需要这些限制之外的更多限制，因此，请随时联系客户服务，以便我们了解你的需求并相应地调整这些限制。对于服务总线，聚合的使用率配额为如下所示：

- 50 亿条消息

虽然我们保留禁用在给定月份超过使用率配额的客户帐户的权利，但我们仍然会在采取任何措施前发送电子邮件通知并会多次尝试与客户联系。超过这些配额的客户仍将负责超出配额的费用。

至于 Azure 上的其他服务，服务总线会强制使用一组特定配额，以确保资源的公平使用。服务强制执行的使用率配额如下：

#### 队列/主题大小

在创建队列或主题时指定最大队列或主题大小。此配额的值可以是 1 GB、2 GB、3 GB、4 GB 或 5 GB。如果达到最大大小，则将拒绝其他传入的消息，调用代码将收到一个异常。

#### 命名限制

服务总线命名空间名称的长度只能介于 6 到 50 个字符之间。每个队列、主题或订阅的字符计数限制为 1 到 50 个字符之间。

#### 并发连接数

队列/主题/订阅 - 队列/主题/订阅上的并发 TCP 连接数限制为 100 个。如果达到此配额，将拒绝后续的其他连接请求，调用代码将收到一个异常。对于每个消息工厂，如果由该消息传送工厂创建的任何客户端具有活动的操作挂起时，或者刚完成操作不超过 60 秒时间，则服务总线都会保持一个 TCP 连接。REST 操作不计入并发 TCP 连接数。

#### 每个服务命名空间的主题/队列数

服务命名空间上的主题/队列（持久存储备份的实体）的最大数限制为 10,000 个。如果达到此配额，将拒绝后续的在服务命名空间上创建新主题/队列的请求。在这种情况下，Azure 经典管理门户将显示一条错误消息或调用客户端代码将收到一个异常，具体取决于创建尝试是通过门户还是通过客户端代码完成的。

### 消息大小配额

#### 队列/主题/订阅

**消息大小** – 每条消息的总大小限制为 256 KB，包括消息标头。

**消息标头大小** – 每条消息标头限制为 64 KB。

系统将拒绝超过这些大小配额的消息，调用代码将收到一个异常。

**每个主题的订阅数** - 每个主题的最大订阅数限制为 2,000。如果达到此配额，则将拒绝创建该主题的附加订阅的后续请求。在这种情况下，Azure 经典门户将显示一条错误消息或调用客户端代码将收到一个异常，具体取决于创建尝试是通过门户还是通过客户端代码完成的。

**每个主题的 SQL 筛选器数** – 每个主题的最大 SQL 筛选器数被限制为 2,000。如果达到此配额，将拒绝任何后续的创建该主题的其他筛选器的请求，调用代码将收到一个异常。

**每个主题的相关筛选器数** - 每个主题的最大相关筛选器数限制为 100,000。如果达到此配额，将拒绝任何后续的创建该主题的其他筛选器的请求，调用代码将收到一个异常。

## <a name="subscription-and-namespace-management"></a> 订阅和命名空间管理

### 如何将命名空间迁移到另一个 Azure 订阅？

可以使用 PowerShell 命令（可在[此处][]的文章中找到）将命名空间从某个 Azure 订阅移到另一个 Azure 订阅。若要执行此操作，命名空间必须已处于活动状态。此外，执行命令的用户必须是源订阅和目标订阅的管理员。

## <a name="service-bus-troubleshooting"></a> 服务总线故障排除

[异常概述][]

### Azure 服务总线消息传送 API 所生成的异常有哪些？建议采取什么操作？

消息传送 API 可能生成的异常分为以下类别：

-   用户编码错误

-   设置/配置错误

-   临时异常

-   其他异常

[服务总线消息传送异常][Exceptions overview]一文介绍了部分异常和建议的操作。

### 什么是共享访问签名？哪些语言支持生成签名？

共享访问签名是基于 SHA–256 安全哈希或 URI 的身份验证机制。有关如何在 Node、PHP、Java 和 C# 中生成自己的签名的信息，请参阅[共享访问签名][]一文。

## 后续步骤

若要了解有关服务总线消息传送的详细信息，请参阅以下主题。

- [服务总线消息传送概述](./service-bus-messaging-overview.md)
- [Azure 服务总线体系结构概述](./service-bus-fundamentals-hybrid-solutions.md)
- [服务总线队列入门](./service-bus-dotnet-get-started-with-queues.md)

[使用服务总线中转消息传送改进性能的最佳实践]: ./service-bus-performance-improvements.md
[使应用程序免受服务总线中断和灾难影响的最佳实践]: ./service-bus-outages-disasters.md
[Pricing overview]: https://www.azure.cn/pricing/details/messaging/
[配额概述]: ./service-bus-quotas.md
[此处]: ./service-bus-powershell-how-to-provision.md#migrate-a-namespace-to-another-azure-subscription
[Exceptions overview]: ./service-bus-messaging-exceptions.md
[异常概述]: ./service-bus-messaging-exceptions.md
[共享访问签名]: ./service-bus-sas-overview.md

<!---HONumber=Mooncake_1219_2016-->