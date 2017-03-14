---
title: 什么是 SQL 数据库？ SQL 数据库简介 | Azure
description: 获取 SQL 数据库简介：Microsoft 在云中的关系数据库管理系统 (RDBMS) 的技术详细信息与功能。
keywords: SQL 简介, 什么是 SQL 数据库
services: sql-database
documentationcenter: ''
author: shontnew
manager: jhubbard
editor: cgronlun

ms.assetid: c561f600-a292-4e3b-b1d4-8ab89b81db48
ms.service: sql-database
ms.custom: overview
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: data-management
ms.date: 12/20/2016
wacn.date: 01/25/2017
ms.author: shkurhek;carlrab
---

# 什么是 SQL 数据库？ SQL 数据库简介
SQL 数据库是云中的关系数据库服务，它基于行业领先的 Microsoft SQL Server 引擎，能够处理任务关键型工作负荷。SQL 数据库在多个服务级别提供可预测的性能、支持在不停机的情况下进行缩放、内置业务连续性和数据保护 — 所有这些功能几乎都不需要管理。凭借这些功能，客户可将注意力集中在如何快速进行应用开发、加快推向市场，而无需将宝贵的时间和资源投入在管理虚拟机和基础结构上。SQL 数据库基于 [SQL Server](https://msdn.microsoft.com/library/bb545450.aspx) 引擎，支持现有的 SQL Server 工具、库和 API。因此，无需学习新的技能，就能轻松开发新解决方案，迁移现有 SQL Server 解决方案，将现有 SQL Server 解决方案扩展到 Microsoft 云中。

本文介绍 SQL 数据库在性能、缩放性和易管理性方面的核心概念与功能，并提供链接以进一步了解详细信息。如果已准备好学习实践教程，请转到[创建第一个 SQL 数据库](./sql-database-get-started.md)或[创建弹性池](./sql-database-elastic-pool-create-portal.md)。此视频提供了简短演示。

## 无需停机即可调整性能和规模
Azure SQL 数据库服务提供三个服务层：基本、标准和高级。每个服务层提供[不同级别的性能和功能](./sql-database-service-tiers.md)，以支持从轻型到重型的数据库工作负荷。可以在小型数据库中构建第一个应用，每个月只需花费少量的资金。然后可以根据解决方案的需要，随时手动或以编程方式[更改服务层](./sql-database-scale-up.md)。这不会给应用或客户造成任何停机。动态可伸缩性可让数据库以透明方式响应快速变化的资源要求，使用户只需为用到的资源付费。

## 弹性池可以最大化资源利用率
许多业务和应用只要能够创建单一数据库并按需调高或调低性能即可，尤其是当使用模式相对容易预测时。但如果有无法预测的使用模式，则管理成本和业务模式就会变得相当困难。[弹性池](./sql-database-elastic-pool.md)旨在解决此问题。概念很简单。可以向池而不是单个数据库分配性能资源，并且仅需为池的总体性能资源付费，而无需为单一数据库的性能付费。使用弹性池时，不需要在资源需求波动时担心如何上下调节数据库性能。入池的数据库可根据需要使用弹性池的性能资源。入池的数据库会使用该池，但不会超出其限制，因此即使单个数据库的使用情况仍不可预测，成本也仍是可预测的。还可以[向池添加和删除数据库](./sql-database-elastic-pool-manage-portal.md)，将应用从少量数据库扩展到数千个，而一切费用不会超出控制的预算范围。最后，还可以控制池中数据库可用的资源量上限与下限，确保池中不会有任何数据库使用所有的池资源，每个入池数据库的可用资源量都有最低保障。若要深入了解如何通过弹性池设计 SaaS 应用程序，请参阅[多租户 SaaS 应用程序和 Azure SQL 数据库的设计模式](./sql-database-design-patterns-multi-tenancy-saas-applications.md)。

## 混合使用单一数据库与入池数据库
不管采用哪种方式 - 单一数据库或弹性池 - 都不会受到限制。可将单一数据库与弹性池混合使用，快速轻松地更改单一数据库和弹性池的服务层来适应自己的情况。此外，凭借 Azure 的功能和作用范围，可将其他 Azure 服务与 SQL 数据库搭配使用以满足独特的应用程序设计需求，提高成本和资源效益，发掘新的商机。

## 监视和警报
但是，要如何比较单一数据库和数据库池的相对性能呢？ 当调高和调低性能时，如何知道该在何处停止？ 根据[单一数据库的数据库事务单位 (DTU) 和弹性池的弹性 DTU (eDTU)](./sql-database-what-is-a-dtu.md)，结合性能评级使用[内置性能监视](./sql-database-performance.md)和[警报](./sql-database-insights-alerts-portal.md)工具。使用这些工具，可以根据当前需求或项目性能的需求，快速评估调高或调低性能产生的影响。有关详细信息，请参阅 [SQL 数据库选项和性能：了解每个服务层提供的功能](./sql-database-service-tiers.md)。

## 使应用和业务持续运转
Azure 行业领先的 99.99% 可用性服务级别协议 [(SLA)](https://www.azure.cn/support/legal/sla/)，有助于保持应用全天候运行。对于每个 SQL 数据库，可以利用内置的安全性、容错和[数据保护](./sql-database-automated-backups.md)功能。使用其他产品时可能需要另外购买或设计、构建并管理这些功能。对于 SQL 数据库，每个服务层都提供了一组全面的业务连续性功能和选项，可以使用这些功能和选项快速进入工作状态并保持运行。可以使用[时间点还原](./sql-database-recovery-using-backups.md)将数据库还原到以前的状态，最多可还原到 35 天前。此外，如果托管数据库的数据中心发生服务中断，可以通过[最新备份的异地冗余副本](./sql-database-recovery-using-backups.md)还原数据库。如果需要，还可以在一个或多个区域配置[异地冗余的可读副本](./sql-database-geo-replication-overview.md)，以便在发生数据中心服务中断时快速故障转移。还可以使用这些副本在不同的地理区域提高读取性能，或者进行[应用程序升级，而不会造成停机](./sql-database-manage-application-rolling-upgrade.md)。

![SQL 数据库异地复制](./media/sql-database-technical-overview/azure_sqldb_map.png)

有关可用于不同服务层的其他业务连续性功能的详细信息，请参阅[业务连续性](./sql-database-business-continuity.md)。

## 保护数据
SQL Server 的数据安全性一贯可靠，SQL 数据库也包含类似的功能，可以限制访问、保护数据，并帮助你监视活动。有关在 SQL 数据库中拥有的安全选项的快速概览，请参阅[保护你的 SQL 数据库](./sql-database-security.md)。有关更全面的安全功能视图，请参阅 [SQL Server 数据库引擎和 SQL 数据库安全中心](https://msdn.microsoft.com/zh-cn/library/bb510589)。有关 Azure 平台安全性的信息，请访问 [Azure 信任中心](https://www.trustcenter.cn/zh-cn/security/default.html)。

## 后续步骤
现在，你已阅读 SQL 数据库简介并回答了“什么是 SQL 数据库？”这个问题，因此可以继续完成以下内容：

- 有关单一数据库和弹性池的成本比较与计算器，请参阅[定价页](https://www.azure.cn/pricing/details/sql-database/)。
- 了解[弹性池](./sql-database-elastic-pool.md)。
- 从[创建你的第一个数据库](./sql-database-get-started.md)开始。
- 使用 C#、Java、Node.js、PHP、Python 或 Ruby 生成第一个应用：[SQL 数据库和 SQL Server 的连接库](./sql-database-libraries.md)

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: wording update; remove two links-->