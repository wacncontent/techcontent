---
title: 在 Azure Active Directory 中管理密码 | Azure
description: 如何在 Azure Active Directory 中管理密码。
services: active-directory
documentationcenter: ''
author: curtand
manager: femila
editor: ''

ms.assetid: a7679724-0ed5-4973-92e2-bd1285a6ef93
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/09/2016
ms.author: curtand
wacn.date: 01/19/2017
---

# 在 Azure Active Directory 中管理密码
如果你是管理员，可通过 Azure 经典管理门户中的 Azure Active Directory (Azure AD) 重置用户的密码。单击你的目录名称，在“用户”页上单击相应的用户名，然后在门户底部单击“重置密码”。

本主题的余下部分将介绍 Azure AD 支持的整套密码管理功能，包括：

- **自助密码更改**：最终用户或管理员可以更改过期的或未过期的密码，而无需请求管理员或支持人员提供支持。
- **自助密码重置**：最终用户或管理员可以自行重置密码，而无需请求管理员或支持人员提供支持。自助密码重置功能需要 Azure AD 高级或基本版。有关详细信息，请参阅 [Azure Active Directory 版本](./active-directory-editions.md)。
- **管理员启动的密码重置**：管理员可以通过 Azure 经典管理门户重置某个最终用户的密码或其他管理员的密码。
- **密码管理活动报告**：管理员可以深入了解发生在其组织中的密码重置和注册活动。

> [!NOTE]
Azure AD 高级版适用于使用世界范围的 Azure AD 实例的中国客户。由中国的 21Vianet 运营的 Azure 服务目前不支持 Azure AD Premium。有关详细信息，请在 [Azure Active Directory 论坛](https://feedback.azure.com/forums/169401-azure-active-directory/)与我们联系。

使用以下链接可跳转至你最感兴趣的文档：

- [概述：Azure AD 中的密码管理](./active-directory-passwords-how-it-works.md)
- [Azure AD 中的自助密码重置：如何启用、配置和测试自助密码重置](./active-directory-passwords-getting-started.md#enable-users-to-reset-their-azure-ad-passwords)
- [Azure AD 中的自助密码重置：如何根据你的需要自定义密码重置](./active-directory-passwords-customize.md)
- [Azure AD 中的自助密码重置：部署和管理最佳实践](./active-directory-passwords-best-practices.md)
- [Azure AD 中的密码管理报告：如何查看租户中的密码管理活动](./active-directory-passwords-get-insights.md)
- [密码写回：如何配置 Azure AD 以管理本地密码](./active-directory-passwords-getting-started.md#enable-users-to-reset-or-change-their-ad-passwords)
- [Azure AD 密码管理故障排除](./active-directory-passwords-troubleshoot.md)
- [Azure AD 密码管理常见问题](./active-directory-passwords-faq.md)

## 后续步骤
- [管理 Azure AD](./active-directory-administer.md)
- [在 Azure AD 中创建或编辑用户](./active-directory-create-users.md)

<!---HONumber=Mooncake_1205_2016-->