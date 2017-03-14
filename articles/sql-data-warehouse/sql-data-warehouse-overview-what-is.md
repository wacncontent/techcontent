---
title: 什么是 Azure SQL 数据仓库？| Azure
description: 企业级分布式数据库，可处理 1 千万亿字节的关系数据和非关系数据量。它是行业首个云数据仓库，可以在数秒内增长、收缩和暂停。
services: sql-data-warehouse
documentationcenter: NA
author: jrowlandjones
manager: bjhubbard
editor: ''

ms.assetid: 4006c201-ec71-4982-b8ba-24bba879d7bb
ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: hero-article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.date: 10/31/2016
wacn.date: 02/20/2017
ms.author: jrj;mausher;kevin;barbkess
---

# 什么是 Azure SQL 数据仓库？
Azure SQL 数据仓库是一种基于云的向外扩展数据库，可以处理大量数据（关系数据和非关系数据）。SQL 数据仓库在大规模并行处理 (MPP) 体系结构的基础上构建，可以处理你的企业工作负荷。

SQL 数据仓库：

* 将 SQL Server 关系数据库与 Azure 云向外扩展功能相结合。可以在几分到几秒内增加、减少、暂停或恢复计算。你可通过在需要时向外扩展 CPU，并在非高峰时段削减使用量，来节省成本。
* 利用 Azure 平台。它易于部署、无缝维护并由于自动备份而完全容错。
* 它是 SQL Server 生态系统的补充。你可以使用熟悉的 SQL Server Transact-SQL (T-SQL) 和工具进行开发。

本文介绍 SQL 数据仓库的主要功能。

## 大规模并行处理体系结构
SQL 数据仓库是一种大规模并行处理 (MPP) 分布式数据库系统。通过将数据和处理能力分布于多个节点，SQL 数据仓库能够提供极大的缩放性 - 远超任何单一系统。在后台，SQL 数据仓库会将数据分布在多个无共享的存储和处理单元上。数据存储在高级本地冗余存储中，并链接到用于执行查询的计算节点。使用此体系结构，SQL 数据仓库采用“分而治之”的方法来运行负载和复杂的查询。请求由控制节点接收、优化，然后传递给计算节点以并行完成其工作。

通过将 MPP 体系结构与 Azure 存储空间功能相结合，SQL 数据仓库可以：

* 独立于计算，扩大或缩小存储空间。
* 无需移动数据，即可增加或减少计算。
* 具有暂停计算功能，同时保持数据不变。
* 具有随时恢复计算功能。

下图更详细地说明了此体系结构。

![SQL 数据仓库体系结构][1]  

**控制节点：**控制节点管理和优化查询。它是与所有应用程序和连接进行交互的前端。在 SQL 数据仓库中，控制节点由 SQL 数据库提供支持，并且连接到它时它看起来就像 SQL 数据库。表面之下，控制节点协调对分布式数据运行并行查询所需的所有数据移动和计算。当你将 T-SQL 查询提交到 SQL 数据仓库时，控制节点会将其转换为可在每个计算节点上并行运行的单独查询。

**计算节点：**计算节点将充当 SQL 数据仓库背后的强大功能。它们是用于存储数据并处理查询的 SQL 数据库。当你添加数据时，SQL 数据仓库向计算节点分配行。计算节点是对数据运行并行查询的工作节点。在处理后，它们将结果传递回控制节点。若要完成查询，控制节点聚合结果并返回最终结果。

**存储：**你的数据存储在 Azure Blob 存储中。当计算节点与数据进行交互时，它们直接将数据写入到 blob 存储中并直接从 blob 存储读取数据。由于 Azure 存储可以透明和大幅地进行扩展，因此 SQL 数据仓库也可以这样做。由于计算与存储是独立的，因此 SQL 数据仓库可自动将缩放存储与缩放计算分开进行，反之亦然。Azure Blob 存储还可以完全容错，并简化了备份和还原过程。

**数据移动服务：**数据移动服务 (DMS) 可在节点之间移动数据。DMS 向计算节点提供访问进行联接和聚合所需数据的权限。DMS 不是一项 Azure 服务。它是在所有节点上与 SQL 数据库一起运行的 Windows 服务。由于 DMS 在后台运行，因此你不会直接与它交互。但是，当你查看查询计划时，你会注意到它们包括某些 DMS 操作，因为在并行运行每个查询时数据移动是必需的。

## 针对数据仓库工作负荷进行了优化
此 MPP 方法得益于许多数据仓库特定性能优化方法，包括：

* 针对所有数据的分布式查询优化器和一组复杂的统计数据。使用有关数据大小和分布的信息，服务就能够通过评估特定分布式查询操作的成本来优化查询。

* 集成到数据移动进程中的高级算法和技术，可以根据需要有效地在计算资源之间移动数据，以便执行查询。这些数据移动操作是内置的，并且对数据移动服务的所有优化是自动进行的。

* 默认情况下，为聚集**列存储**编制索引。与传统的行导向存储相比，使用基于列的存储时，SQL 数据仓库平均可将压缩率提高 5 倍，最大可将查询性能提高 10 倍或更多。需要扫描大量行的分析查询非常适合使用列存储索引。

## 可预测且可缩放的性能
SQL 数据仓库对存储和计算功能进行了分隔，允许每项功能单独进行缩放。SQL 数据仓库可以快速简单地扩展以随时添加额外的计算资源。可以借助 Azure Blob 存储来实现这项功能。Blob 不只提供稳定的复制存储，同时还提供低成本的轻松扩展基础结构。SQL 数据仓库运用云规模的存储和 Azure 计算的组合形式，允许你只在需要时支付查询性能和存储的费用。更改计算量就像是在 Azure 门户中将滑块向左或向右移动一样简单，但也可以使用 T-SQL 和 PowerShell 对其进行计划。

SQL 数据仓库提供独立于存储且可完全控制计算量的功能，可让你完全暂停数据仓库，这意味着在你不需要计算功能时可以不为其付费。在就地保留存储的同时，所有计算资源都将释放到 Azure 的主池中，从而为你节省费用。必要时，你只需恢复计算，让工作负荷使用数据和计算资源即可。

<a name="data-warehouse-units"></a>
## 数据仓库单位
对 SQL 数据仓库的资源分配以数据仓库单位 (DWU) 来度量。DWU 是分配给 SQL 数据仓库的基础资源（如 CPU、内存、IOPS）的度量值。增加 DWU 数可增加资源和提高性能。具体而言，DWU 可帮助确保：

* 可以轻松缩放数据仓库，而不必考虑基础硬件或软件。

* 在更改数据仓库的大小之前，可以预测 DWU 级别的性能改进。

* 可以更改或移动实例的基础硬件和软件，而不影响工作负荷性能。

* Microsoft 可以调整服务的基础体系结构，而不影响工作负荷的性能。

* Microsoft 可以用可缩放及适度影响系统的方式快速提升 SQL 数据仓库的性能。

数据仓库单位提供了与数据仓库工作负荷性能高度相关的三个精确度量的度量值。目标是以下关键工作负荷度量值以针对数据仓库选择的 DWU 进行线性缩放。

**扫描/聚合：**此工作负荷度量值采用标准数据仓库查询，该查询扫描大量的行，然后执行复杂聚合。这是一种 I/O 和 CPU 密集型操作。

**负载：**此度量值度量将数据引入服务的能力。使用 PolyBase 从 Azure Blob 存储加载具代表性的数据集来完成负载。此度量值旨在增加服务的网络和 CPU 方面的压力。

**Create Table As Select \(CTAS\)：**CTAS 度量复制表的能力。这涉及到从存储读取数据、将数据分发到设备的节点上，以及重新将数据写入到存储。这是一种 CPU、IO 和网络密集型操作。

## 按需暂停和缩放
当你需要更快的结果时，可以增加 DWU 并为较高的性能付费。当你需要较小的计算能力时，可以减少 DWU 并且仅针对需要的项目付费。在下列情况下你可能会考虑更改 DWU：

* 在无需运行查询时，可能是在晚上或周末，使你的查询处于静默状态。然后暂停你的计算资源，以避免在不需要它们时为 DWU 付费。

* 当系统的需求较低时，请考虑将 DWU 减少到较小的大小。你仍然可以访问数据，但可以显著节省成本。

* 执行繁重的数据加载或转换操作时，你可能想要增加 DWU 以便可以更快地获取数据。

若要了解理想的 DWU 值，请尝试在加载数据之后增加和减少 DWU 并运行几个查询。由于缩放很快就能完成，你可以在一个小时或更少时间内尝试一些不同级别的性能。请务必记住，SQL 数据仓库设计用于处理大量数据，若要了解其处理大数据的真正能力，尤其是我们提供的较大规模数据，你需要使用接近或超过 1 TB 的大型数据集。

## 在 SQL Server 的基础上构建
SQL 数据仓库基于 SQL Server 关系数据库引擎，包含你希望企业数据仓库拥有的许多功能。如果你已了解 T-SQL，则使用 SQL 数据仓库时可以轻松运用这些知识。无论你是高级用户还是新手，都可以从本文档中的示例开始着手。总体上，你可以考虑我们构建 SQL 数据仓库的语言元素的方式，如下所示：

- SQL 数据仓库的许多操作使用 T-SQL 语法。它还支持一组广泛的传统 SQL 构造，例如存储过程、用户定义的函数、表分区、索引和排序规则。

- SQL 数据仓库还包含一些较新的 SQL Server 功能，包括聚集**列存储**索引、PolyBase 集成和数据审核（随附威胁评估）。

* 对数据仓库工作负荷不常用的某些 T-SQL 语言元素，或 SQL Server 的新增功能当前可能不可用。有关详细信息，请参阅[迁移文档][Migration documentation]。

借助 SQL Server、SQL 数据仓库、SQL 数据库和分析平台系统之间的 Transact-SQL 与功能共通性，能够开发符合数据需求的解决方案。你可以根据性能、安全和缩放要求，决定存储数据的位置，然后根据需要在不同系统之间传输数据。

## 数据保护

SQL 数据仓库将 Azure 高级版中的所有数据存储在本地冗余存储中。将在本地数据中心内维护数据的多个同步副本，确保在发生局部故障时能够保证提供透明的数据保护。此外，SQL 数据仓库会使用 Azure 存储空间快照，定期自动备份活动（未暂停）数据库。若要详细了解备份和还原的工作原理，请参阅[备份和还原概述][Backup and restore overview]。

## 与 Microsoft 工具相集成
SQL 数据仓库还集成了 SQL Server 用户可能会熟悉的许多工具。其中包括：

**传统的 SQL Server 工具：**SQL 数据仓库完全集成了 SQL Server Analysis Services、Integration Services 和 Reporting Services。

**基于云的工具：**SQL 数据仓库可与 Azure 中的多个新工具一起使用，包括数据工厂、流分析、机器学习和 Power BI。有关更完整的列表，请参阅[集成工具概述][Integrated tools overview]。

**第三方工具：**大量的第三方工具提供商都已证明其工具能够与 SQL 数据仓库集成。
<!-- Not available for SQL Data Warehouse solution partners -->
<!-- 有关完整列表，请参阅 [SQL 数据仓库解决方案合作伙伴][SQL Data Warehouse solution partners]。 -->

## 混合数据源方案

通过将 SQL 数据仓库与 PolyBase 结合使用，给予用户前所未有的能力，在其生态系统之间移动数据，从而解锁使用非关系与本地数据源设置高级混合方案的能力。

Polybase 可让你使用熟悉的 T-SQL 命令利用不同源中的数据。Polybase 使你可以像查询普通表一样查询 Azure Blob 存储中的非关系数据。使用 Polybase 可以查询非关系数据，或者将非关系数据导入 SQL 数据仓库。

* PolyBase 使用外部表访问非关系数据。表定义存储在 SQL 数据仓库中，可以使用 SQL 和用于访问普通关系数据的工具进行访问。

* Polybase 并不知道其集成状态。它为支持的所有源提供相同的特性和功能。Polybase 可读取各种格式的数据，包括分隔文件或 ORC 文件。

* PolyBase 可用于访问 Blob 存储，该存储也用作 HDInsight 群集的存储。这样就可以使用关系和非关系工具访问同一数据。

## SLA
SQL 数据仓库提供产品级别的服务级别协议 (SLA) 作为 Microsoft Online Services SLA 的一部分。有关详细信息，请访问 [SQL 数据仓库的 SLA][SLA for SQL Data Warehouse]。有关所有其他产品的 SLA 信息，可访问[服务级别协议] Azure 页面，或在[批量许可][Volume Licensing]页面下载。

## 后续步骤

对 SQL 数据仓库有了初步的认识后，请继续了解如何快速[创建 SQL 数据仓库][create a SQL Data Warehouse]和[加载示例数据][load sample data]。如果用户不熟悉 Azure，可在遇到新术语时查看 [Azure 术语表][Azure glossary]。或者，查看以下一些其他 SQL 数据仓库资源。

<!-- Not available for custom success case -->
* [博客]
* [功能请求]
* [视频]
* [客户顾问团队博客]
<!-- Not available for sql-data-warehouse-get-started-create-support-ticket -->
* [MSDN 论坛]
* [Stack Overflow 论坛]
* [Twitter]

<!--Image references-->

[1]: ./media/sql-data-warehouse-overview-what-is/dwarchitecture.png

<!--Article references-->

[创建支持票证]: /documentation/articles/sql-data-warehouse-get-started-create-support-ticket/
[load sample data]: ./sql-data-warehouse-load-sample-databases.md
[create a SQL Data Warehouse]: ./sql-data-warehouse-get-started-provision.md
[Migration documentation]: ./sql-data-warehouse-overview-migrate.md
[SQL Data Warehouse solution partners]: /documentation/articles/sql-data-warehouse-partner-business-intelligence/
[Integrated tools overview]: ./sql-data-warehouse-overview-integrate.md
[Backup and restore overview]: ./sql-data-warehouse-restore-database-overview.md
[Azure glossary]: ../azure-glossary-cloud-terminology.md

<!--MSDN references-->

<!--Other Web references-->

[客户成功案例]: https://customers.microsoft.com/search?sq=&ff=story_products_services%26%3EAzure%2FAzure%2FAzure%20SQL%20Data%20Warehouse%26%26story_product_families%26%3EAzure%2FAzure%26%26story_product_categories%26%3EAzure&p=0
[博客]: https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/
[客户顾问团队博客]: https://blogs.msdn.microsoft.com/sqlcat/tag/sql-dw/
[功能请求]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[MSDN 论坛]: https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=AzureSQLDataWarehouse
[Stack Overflow 论坛]: http://stackoverflow.com/questions/tagged/azure-sqldw
[Twitter]: https://twitter.com/hashtag/SQLDW
[视频]: https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse
[SLA for SQL Data Warehouse]: https://azure.microsoft.com/zh-cn/support/legal/sla/sql-data-warehouse/v1_0/
[Volume Licensing]: http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=37
[服务级别协议]: https://www.azure.cn/support/legal/sla/

<!---HONumber=Mooncake_Quality_Review_1230_2016-->
<!--Update_Description:meta-data; words update; updating link reference -->