---
title: Power BI Embedded 中的新增功能
description: 获取 Power BI Embedded 中的新增功能的最新信息
services: power-bi-embedded
documentationcenter: ''
author: guyinacube
manager: erikre
editor: ''
tags: ''

ms.assetid: 2794ae98-b9a7-45df-b6e1-962a395b91fa
ms.service: power-bi-embedded
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: powerbi
ms.date: 01/06/2017
wacn.date: 02/22/2017
ms.author: asaxton
---

# Power BI Embedded 中的新增功能
**Power BI Embedded** 的更新会定期发布。但是，并非每个版本都包括面向用户的新功能；某些版本聚焦于后端的服务功能。此处将重点介绍面向用户的新功能。请务必经常查看。

## 2016 年 12 月

- [新 JavaScript 嵌入示例](https://microsoft.github.io/PowerBI-JavaScript/demo/)

## 2016 年 10 月

- [Power BI Embedded 和 R 的高级分析](https://powerbi.microsoft.com/blog/r-in-pbie/)

## 2016 年 8 月 31日
此版本中包括的新增功能：

- 支持[高级筛选和页面导航](./power-bi-embedded-interact-with-reports.md)的全新 JavaScript SDK。

## 2016 年 7 月 11日
此版本中包括的新增功能：

- **好消息！** Power BI Embedded 服务现在不再是预览版 - 它现在已是 GA 版（公开发布了）。
- 所有 REST API 都已从 **/beta** 迁移到 **/v1.0**。
- .NET 和 JavaScript SDK 已更新为 **v1.0**。
- 现在可以通过使用 API 密钥直接对 Power BI API 调用进行身份验证。只有对于嵌入请求，才需要使用应用令牌。作为此更新的一部分，预配和开发令牌在 v1.0 API 中已被弃用，但它们在 beta 版本中将继续工作到 2016 年 12 月 30 日。若要了解详细信息，请参阅[通过 Power BI Embedded 进行身份验证和授权](./power-bi-embedded-app-token-flow.md)。
- 对应用令牌和嵌入的报表的行级别安全性 (RLS) 支持。若要了解详细信息，请参阅 [Power BI Embedded 中的行级别安全性](./power-bi-embedded-rls.md)。
- 为所有 **v1.0** API 调用更新了示例应用程序。
- 提供了对 Azure SDK、PowerShell 和 CLI 的 Power BI Embedded 支持。
- 用户可以将可视化数据导出到 **.csv** 文件。
- 现在，与 Azure 相同的所有语言/区域设置都支持 Power BI Embedded。若要了解详细信息，请参阅 [Azure - 语言](http://social.technet.microsoft.com/wiki/contents/articles/4234.windows-azure-extent-of-localization.aspx)。

<!---HONumber=Mooncake_0213_2017-->