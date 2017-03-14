---
title: Azure Active Directory 版本 | Azure
description: 本文介绍 Azure Active Directory 的免费版和付费版选项。Azure Active Directory Basic、Azure Active Directory Premium P1 和 Azure Active Directory Premium P2 是付费版本。
services: active-directory
documentationcenter: ''
author: curtand
manager: femila
editor: ''

ms.assetid: dcaf8939-7633-40a8-bd76-27dedbb6083a
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/16/2016
ms.author: curtand
wacn.date: 01/03/2017
---

# Azure Active Directory 版本
所有 Microsoft Online 业务服务都依赖于 Azure Active Directory (Azure AD) 进行登录及满足其他标识需求。如果订阅了任何 Microsoft Online 业务服务（例如，Office 365 或 Azure），则表示已获得 Azure AD，可用于访问下述所有免费功能。

Azure Active Directory 是一个综合性的、高度可用的标识和访问管理云解决方案，它将核心目录服务、高级标识监管和应用程序访问管理结合起来。Azure Active Directory 还提供了功能丰富的基于标准的平台，该平台支持开发人员根据集中的策略和规则为应用程序提供访问控制。使用 Azure Active Directory 免费版，你可以管理用户和组、与本地目录同步，以及在 Azure、Office 365 和数千种主流 SaaS 应用程序（如 Salesforce、Workday、Concur、DocuSign、Google Apps、Box、ServiceNow、Dropbox 等）上单一登录。若要了解有关 Azure Active Directory 的详细信息，请阅读[什么是 Azure AD？](./active-directory-whatis.md)

若要增强 Azure Active Directory，可以使用 Azure Active Directory Basic、Premium P1、和 Premium P2 版添加付费功能。Azure Active Directory 付费版建立在现有免费目录基础之上，提供企业级功能，包括自助服务、增强型监视、安全报告、多重身份验证 (MFA) 和移动工作者安全访问。

Office 365 订阅包括以下比较表中所述的其他 Azure Active Directory 功能。

> [!NOTE]
> 有关这些版本的定价选项，请参阅 [Azure Active Directory 定价](https://www.azure.cn/pricing/details/identity/)。中国地区目前不支持 Azure Active Directory Premium P1、Premium P2 和 Azure Active Directory Basic。有关详细信息，请通过 Azure Active Directory 论坛与我们联系。

> [!NOTE]
许多 Azure Active Directory 功能以“即付即用”版本的形式提供：
> * 可以通过基于用户或基于身份验证的提供程序使用 Azure 多重身份验证。有关详细信息，请参阅[什么是 Azure 多重身份验证？](../multi-factor-authentication/multi-factor-authentication.md)
>

## 比较正式推出的功能

> [!NOTE]
> 有关此数据的不同视图，请参阅 [Azure Active Directory 功能](https://www.microsoft.com/en/server-cloud/products/azure-active-directory/features.aspx)。

**常用功能**

- [目录对象](#directory-objects)
- [用户/组管理（添加/更新/删除）/基于用户的预配、设备注册](#usergroup-management-addupdatedelete-user-based-provisioning-device-registration)
- [单一登录 (SSO)](#single-sign-on-sso)
- [云用户的自助密码更改](#self-service-password-change-for-cloud-users)
- [Connect（将本地目录扩展到 Azure Active Directory 的同步引擎）](#connect-sync-engine-that-extends-on-premises-directories-to-azure-active-directory)
- [安全性/使用情况报告](#securityusage-reports)

**Basic 功能**

- [云用户的自助密码重置](#self-service-password-reset-for-cloud-users)
- [公司品牌（登录页/访问面板自定义）](#company-branding-logon-pagesaccess-panel-customization)
- [应用程序代理](#application-proxy)
- [SLA 99.9%](#sla-999)

**Premium P1 功能**

- [自助组和应用管理/自助应用程序添加件/动态组](#self-service-group)
- [通过本地写回实现自助密码重置/更改/解锁](#self-service-password-resetchangeunlock-with-on-premises-write-back)
- [多重身份验证（云和本地（MFA 服务器））](#multi-factor-authentication-cloud-and-on-premises-mfa-server)
- [MIM CAL + MIM 服务器](#mim-cal-mim-server)
- [组帐户的自动密码滚动更新](#automatic-password-rollover-for-group-accounts)

## 常用功能
#### 目录对象 <a name="directory-objects"></a>
**类型：**常用功能

默认使用配额为 150,000 个对象。对象是目录服务中的一个实体，由其唯一可分辨名称表示。对象的一个例子就是用于身份验证目的的用户条目。如果你需要超过此默认使用量配额，请与支持人员联系。50 万个对象限制不适用于 Office 365、Microsoft Intune 或任何其他依赖 Azure Active Directory 提供目录服务的 Microsoft 付费联机服务。

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| 最多 500,000 个对象 |无对象限制 |无对象限制 |Office 365 用户帐户没有对象限制 |

#### 用户/组管理（添加/更新/删除）/基于用户的预配、设备注册 <a name="usergroup-management-addupdatedelete-user-based-provisioning-device-registration"></a>
**类型：**常用功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| ![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |

**更多详细信息：**

- [管理 Azure AD 目录](./active-directory-administer.md)
#### 单一登录 (SSO) <a name="single-sign-on-sso"></a>
**类型：**常用功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| 每个用户 10 个应用 (1) |每个用户 10 个应用 (1) |无限制 (2) |每个用户 10 个应用 (1) |

1. 使用 Azure AD 免费版和 Azure AD 基本版，最终用户可以通过单一登录访问多达 10 个应用程序。
2. 使用应用程序库菜单中提供的模板，自助集成支持 SAML、SCIM 或基于窗体的身份验证的任何应用程序。

**更多详细信息：**

- [使用 Azure Active Directory (AD) 管理应用程序](./active-directory-enable-sso-scenario.md)

#### 云用户的自助密码更改 <a name="self-service-password-change-for-cloud-users"></a>
**类型：**常用功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| ![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |

**更多详细信息：**

- [如何更新自己的密码](./active-directory-passwords-update-your-own-password.md)

#### Connect（将本地目录扩展到 Azure Active Directory 的同步引擎）<a name="connect-sync-engine-that-extends-on-premises-directories-to-azure-active-directory"></a>
**类型：**常用功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| ![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |

**更多详细信息：**

- [将本地标识与 Azure Active Directory 集成](./active-directory-aadconnect.md)

#### 安全/使用情况报告 <a name="securityusage-reports"></a>
**类型：**常用功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| 3 个基本报告 |3 个基本报告 |高级报告 |3 个基本报告 |

#### 云用户的自助密码重置 <a name="self-service-password-reset-for-cloud-users"></a>
**类型：**基本功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| |![勾选标记][12] |![勾选标记][12] | ![勾选标记][12] |

**更多详细信息：**

- [用户和管理员的 Azure AD 密码重置](./active-directory-passwords.md)

#### 公司品牌（登录页/访问面板自定义）<a name="company-branding-logon-pagesaccess-panel-customization"></a>
**类型：**基本功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| |![勾选标记][12] |![勾选标记][12] | ![勾选标记][12] |

**更多详细信息：**

- [向“登录”和“访问面板”页添加公司品牌](./active-directory-add-company-branding.md)

#### 应用程序代理 <a name="application-proxy"></a>
**类型：**基本功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| |![勾选标记][12] | ![勾选标记][12]  
 | |

#### SLA 99.9% <a name="sla-999"></a>
**类型：**基本功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| |![勾选标记][12] |![勾选标记][12] | ![勾选标记][12] |

**更多详细信息：**

- [服务级别协议](https://www.azure.cn/support/legal/sla/)

## Premium 功能

#### <a name="self-service-group"></a>自助组和应用管理/自助应用程序添加/动态组
**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| | | ![勾选标记][12]| |

#### 通过本地写回实现自助密码重置/更改/解锁 <a name="self-service-password-resetchangeunlock-with-on-premises-write-back"></a>
**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| | | ![勾选标记][12]  
 | |

#### 多重身份验证（云和本地（MFA 服务器））<a name="multi-factor-authentication-cloud-and-on-premises-mfa-server"></a>
**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| | |![勾选标记][12]  
 |对于 Office 365 应用限制为云 |

**更多详细信息：**

- [什么是 Azure 多重身份验证？](../multi-factor-authentication/multi-factor-authentication.md)

#### <a name="mim-cal-mim-server"></a>MIM CAL + MIM 服务器
Microsoft 标识管理器服务器软件权限随 Windows Server 许可证（任意版本）一起授予。由于 Microsoft 标识管理器在 Windows Server 操作系统上运行，只要服务器正在运行有效的、经过许可的 Windows Server 副本，就能在该服务器上安装和使用 Microsoft 标识管理器。Microsoft 标识管理器服务器不需要其他单独的许可证。

**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| | | ![勾选标记][12]  
 | |

#### 组帐户的自动密码滚动更新 <a name="automatic-password-rollover-for-group-accounts"></a>
**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| | | ![勾选标记][12]  
 | |

#### 标识保护
**类型：**高级功能

| 免费版 | 基本版 | Premium P2 版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| | | ![勾选标记][12]  
 | |

## Azure Active Directory Join - 仅适用于 Windows 10 的相关功能
#### 让设备加入 Azure AD、Desktop SSO、Microsoft Passport for Azure AD 和 Administrator Bitlocker 恢复 <a name="join-a-device-to-azure-ad-desktop-sso-microsoft-passport-for-azure-ad-administrator-bitlocker-recovery"></a>
**类型：**Azure Active Directory Join - 仅适用于 Windows 10 的相关功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| ![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |![勾选标记][12]  
 |

#### <a name="mdm-auto-enrollment"></a>MDM 自动注册、自助 Bitlocker 恢复、通过 Azure AD Join 将其他本地管理员加入 Windows 10 设备
**类型：**Azure Active Directory Join - 仅适用于 Windows 10 的相关功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| | | ![勾选标记][12]  
 | |

#### 企业状态漫游
**类型：**Azure Active Directory Join - 仅适用于 Windows 10 的相关功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| | | ![勾选标记][12]  
 | |

## Azure AD 预览功能
除 Free、Basic 和 Premium（P1 和 P2）版正式推出的功能外，Azure AD 还提供一系列预览功能。你可以通过预览功能了解不久的将来可用的功能，并判断这些功能能否帮助改善你的环境。

**可用的预览功能：**

- [管理单元](./active-directory-administrative-units-management.md)
- [iOS 上基于证书的身份验证](./active-directory-certificate-based-authentication-ios.md)
- [Android 上基于证书的身份验证](./active-directory-certificate-based-authentication-android.md)

## 后续步骤
- [向“登录”和“访问面板”页添加公司品牌](./active-directory-add-company-branding.md)

<!--Image references-->

[12]: ./media/active-directory-editions/ic195031.png

<!---HONumber=Mooncake_Quality_Review_1230_2016-->