---
title: Azure AD 和应用程序：开发人员指导 | Azure
description: 本文专门为 IT 专业人员编写，提供有关将 Azure 应用程序与 Active Directory 集成的指导。
services: active-directory
documentationCenter: ''
authors: kgremban
manager: femila
editor: ''

ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/03/2016
wacn.date: 01/19/2017
ms.author: kgremban
---

# Azure AD 和应用程序：开发业务线应用

本指南提供开发用于 Azure Active Directory (AD) 的业务线 (LoB) 应用程序的概述，是专门为 Active Directory/Office 365 全局管理员编写的。

## 概述

构建集成 Azure AD 的应用程序可让组织的用户使用 Office 365 单一登录。在 Azure AD 中拥有应用程序可让你控制为应用程序设置的身份验证策略。

注册应用程序以使用 Azure Active Directory。注册应用程序意味着开发人员可以使用 Azure AD 对用户进行身份验证并请求访问用户资源（如电子邮件、日历和文档）。

目录的任何成员（不是来宾）都可以注册应用程序，也称为 *创建应用程序对象* 。

注册应用程序后，可让任何用户执行以下操作：

- 获取 Azure AD 识别的应用程序标识
- 获取应用程序可用于向 AD 验证其身份的一个或多个机密/密钥
- 在 Azure 门户预览中使用自定义名称、徽标等指定应用程序的品牌。
- 对应用应用 Azure AD 授权功能，包括：
  - 基于角色的访问控制 (RBAC)
  - 使用 Azure Active Directory 作为 oAuth 授权服务器（保护应用程序公开的 API）

- 声明让应用程序按预期运行所需的权限，包括：
      - 应用权限（仅限全局管理员）。例如：另一个 Azure AD 应用程序中的角色成员身份，或相对于 Azure 资源、资源组或订阅的角色成员身份
      - 委派的权限（任何用户）。例如：Azure AD、登录和读取配置文件

> [!NOTE]
>默认情况下，任何成员都可以注册应用程序。若要了解如何限定只有特定成员拥有注册应用程序的权限，请参阅 [How applications are added to Azure AD（如何将应用程序添加到 Azure AD）](./active-directory-how-applications-are-added.md#who-has-permission-to-add-applications-to-my-azure-ad-instance)文档。

下面是全局管理员需要执行哪些操作，才能帮助开发人员将其应用程序投放到生产环境：

- 配置访问规则（访问策略/MFA）
- 将应用程序配置为要求用户分配，并分配用户
- 抑制默认的用户同意体验

## 配置访问规则

在 SaaS 应用中配置基于应用程序的访问规则。这可以包括要求 MFA，或只允许受信任网络上的用户访问。

## 将应用程序配置为要求用户分配，并分配用户

默认情况下，用户无需分配即可访问应用程序。不过，如果应用程序公开角色或者你希望应用程序出现在用户的访问面板上，则你应该要求用户分配。

[要求用户分配](./active-directory-applications-guiding-developers-requiring-user-assignment.md)

如果你是 Azure AD Premium 或 Enterprise Mobility Suite (EMS) 的订阅者，我们强烈建议使用组。将组分配到应用程序可让你将持续进行的访问管理委派给组所有者。你可以创建组，或使用组管理功能请求组织中负责人创建组。

[将用户分配到应用程序](./active-directory-applications-guiding-developers-assigning-users.md)  

## 抑制用户同意

默认情况下，每个用户通过同意体验进行登录。对于不太熟悉做出此类决定的用户而言，同意体验（请求用户授予对应用程序的权限）可能会令其不安。

对于你信任的应用程序，你可以代表组织来同意应用程序，以简化用户体验。

有关 Azure 中的用户同意和同意体验的详细信息，请参阅 [Integrating Applications with Azure Active Directory（将应用程序与 Azure Active Directory 集成）](./active-directory-integrating-applications.md)。

##相关文章

- [有关 Azure Active Directory 中应用程序管理的文章索引](./active-directory-apps-index.md)

<!---HONumber=Mooncake_0829_2016-->