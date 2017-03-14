---
title: Azure AD Connect：用户登录 | Azure
description: Azure AD Connect 用户登录的自定义设置。
services: active-directory
documentationcenter: ''
author: billmath
manager: femila
editor: curtand

ms.assetid: 547b118e-7282-4c7f-be87-c035561001df
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/06/2016
wacn.date: 01/24/2017
ms.author: billmath
---

# Azure AD Connect 用户登录选项
Azure AD Connect 可让用户使用同一组密码登录云和本地资源。本文介绍每个标识模型的重要概念，帮助用户选择登录到 Azure AD 时需要使用的标识。

如果用户已熟悉 Azure AD 标识模型，只想详细了解某个特定的方法，则可直接单击下面的相应标题。

- 使用[单一登录 (SSO)](./active-directory-aadconnect-sso.md) 的[密码同步](#password-synchronization)
- [直通身份验证](./active-directory-aadconnect-pass-through-authentication.md)
- [联合 SSO（使用 ADFS）](#federation-using-a-new-or-existing-ad-fs-in-windows-server-2012-r2-farm)

##为你的组织选择用户登录方法
由于大多数组织只想让用户登录 Office 365、SaaS 应用程序和其他基于 Azure AD 的资源，因此建议使用默认的密码同步选项。但是，一些组织由于特殊原因不能使用此选项，而可以选择联合登录选项（如 AD FS）或者直通身份验证。下表可帮助你做出正确的选择。

我需要 | PHS 和 SSO| 使用 SSO 的 PTA| AD FS |
 --- | --- | --- | --- |
将本地 Active Directory 中创建的新用户、联系人和组帐户自动同步到云|x|x|x|
为 Office 365 混合方案设置我的租户|x|x|x|
使用户能够使用其本地密码登录并访问云服务|x|x|x|
使用公司凭据实现单一登录|x|x|x|
确保未在云中存储密码||x*|x|
启用本地多重身份验证解决方案|||x|

- 通过轻型连接器。

>[!NOTE] 
目前直通身份验证对于富客户端具有某些限制。有关详细信息，请参阅[直通身份验证](./active-directory-aadconnect-pass-through-authentication.md)。

### 密码同步 <a name="password-synchronization"></a>
使用密码同步可将用户密码的哈希从本地 Active Directory 同步到 Azure AD。当密码在本地更改或重置时，新密码将立即同步到 Azure AD，使用户用来访问云资源的密码始终与本地相同。密码绝不将发送到 Azure AD，也不会以明文存储在 Azure AD 中。可将密码同步与密码写回一起使用，以便在 Azure AD 中启用自助密码重置。

此外，还可以对公司网络中的已加入域的计算机上的用户启用[单一登录 (SSO)](./active-directory-aadconnect-sso.md)。通过单一登录，受支持的用户只需输入用户名即可安全访问云资源。

![云](./media/active-directory-aadconnect-user-signin/passwordhash.png)  

[有关密码同步的详细信息](./active-directory-aadconnectsync-implement-password-synchronization.md)

### 直通身份验证
在直通身份验证中，使用本地 Active Directory 控制器验证用户的密码，并且密码无需以任何形式存在于 Azure AD 中。此身份验证允许在对云服务的身份验证过程中评估本地策略，例如登录时间限制。直通身份验证使用本地环境中已加入域的 Windows Server 2012 R2 计算机上的简单代理，此代理侦听密码验证请求。此代理不需要打开至 Internet 的任何入站端口。

此外，还可以对公司网络中的已加入域的计算机上的用户启用单一登录。通过单一登录，受支持的用户只需输入用户名即可安全访问云资源。
![直通身份验证](./media/active-directory-aadconnect-user-signin/pta.png)

有关[直通身份验证](./active-directory-aadconnect-pass-through-authentication.md)和[单一登录](./active-directory-aadconnect-sso.md)的详细信息。

### 使用 Windows Server 2012 R2 场中新的或现有 AD FS 的联合 <a name="federation-using-a-new-or-existing-ad-fs-in-windows-server-2012-r2-farm"></a>
通过联合登录，用户可以使用本地密码登录基于 Azure AD 的服务，并且在公司网络中时，甚至无需输入密码。使用 AD FS 的联合选项可让你在 Windows Server 2012 R2 场中部署新的或指定现有的 AD FS。如果你选择指定现有的场，Azure AD Connect 将在场与 Azure AD 之间配置信任，使用户能够登录。

<center>![云](./media/active-directory-aadconnect-user-signin/federatedsignin.png)</center>

#### 若要在 Windows Server 2012 R2 中部署使用 AD FS 的联合身份验证，你需要以下各项
如果你要部署新场：

- 用于联合服务器的 Windows Server 2012 R2 服务器。
- 用于 Web 应用程序代理的 Windows Server 2012 R2 服务器。
- 一个 .pfx 文件，其中包含目标联合服务名称（例如 fs.contoso.com）的 SSL 证书。

如果你要部署新场或使用现有场：

- 联合服务器上的本地管理员凭据。
- 要部署 Web 应用程序代理角色的任何工作组（未加入域）服务器上的本地管理员凭据。
- 执行向导的计算机必须能够通过 Windows 远程管理连接到要安装 AD FS 或 Web 应用程序代理的任何其他计算机。

[使用 AD FS 配置 SSO](./active-directory-aadconnect-get-started-custom.md#configuring-federation-with-ad-fs)

#### 使用早期版本的 AD FS 或第三方解决方案登录
如果你已使用早期版本的 AD FS（例如 AD FS 2.0）或第三方联合身份验证提供者配置了云登录，则可以通过 Azure AD Connect 选择跳过用户登录配置。这样，你便可以获取最新的同步和 Azure AD Connect 的其他功能，同时仍可使用现有的解决方案进行登录。

[Azure AD 第三方联合身份验证兼容性列表](./active-directory-aadconnect-federation-compatibility.md)

## 用户登录名和用户主体名 (UPN)
### 了解用户主体名
在 Active Directory 中，默认的 UPN 后缀是在其中创建用户帐户的域的 DNS 名称。在大多数情况下，这是在 Internet 上注册为企业域的域名。但是，你可以使用 Active Directory 域和信任来添加其他 UPN 后缀。

用户的 UPN 格式为 username@domain。例如，对于名为 contoso.com 的 Active Directory 域，名为 John 的用户的 UPN 为 john@contoso.com。用户的 UPN 基于 RFC 822。尽管 UPN 和电子邮件具有相同的格式，但用户的 UPN 值不一定等于用户的电子邮件地址。

### Azure AD 中的用户主体名
Azure AD Connect 向导将使用 userPrincipalName 属性，或让你指定要从本地用作 Azure AD 中的用户主体名的属性（在自定义安装中）。这是要用于登录 Azure AD 的值。如果用户主体名属性的值不对应于 Azure AD 中已验证的域，则 Azure AD 会将该值替换为默认的 .partner.onmschina.cn 值。

Azure Active Directory 中的每个目录随附内置域名，格式为 contoso.partner.onmschina.cn，可让你开始使用 Azure 或其他 Microsoft 服务。你可以使用自定义域来改善和简化登录体验。有关 Azure AD 中的自定义域名以及如何验证域的信息，请阅读 [Add your custom domain name to Azure Active Directory](./active-directory-add-domain.md#add-a-custom-domain-name-to-your-directory)（将自定义域名添加到 Azure Active Directory）

## Azure AD 登录配置 <a name="azure-ad-sign-in-configuration"></a>
### 使用 Azure AD Connect 配置 Azure AD 登录
Azure AD 登录体验取决于 Azure AD是否能够匹配要同步到 Azure AD 目录中已验证的某个自定义域的用户的用户主体名后缀。Azure AD Connect 在配置 Azure AD 登录设置时将提供帮助，以便用户在云中的登录体验类似于本地体验。Azure AD Connect 将列出针对域定义的 UPN 后缀，尝试将它们与 Azure AD 中的自定义域进行匹配，并帮助你根据需要采取的相应操作。Azure AD 的登录页列出了针对本地 Active directory 定义的 UPN 后缀，并根据每个后缀显示相应的状态。状态值可以是下列其中一项：

| 状态 | 说明 | 所需操作 |
|:--- |:--- |:--- |
| 已验证 |Azure AD Connect 在 Azure AD 中找到匹配的已验证域。此域的所有用户均可使用其本地凭据登录 |无需操作 |
| 未验证 |Azure AD Connect 可在 Azure AD 中找到匹配的但未验证的自定义域。如果域未验证，则在同步后会将此域的用户的 UPN 后缀更改为默认的 .partner.onmschina.cn 后缀。 |在 Azure AD 中验证自定义域。[了解详细信息](./active-directory-add-domain.md#verify-the-domain-name-with-azure-ad) |
| 未添加 |Azure AD Connect 找不到对应于 UPN 后缀的自定义域。如果未在 Azure 中添加和验证域，则会将此域的用户的 UPN 后缀更改为默认的 .partner.onmschina.cn。 |添加和验证与 UPN 后缀相对应的自定义域 [了解更多](./active-directory-add-domain.md) |

Azure AD 的登录页列出了针对本地 Active Directory 定义的 UPN 后缀，以及 Azure AD 中对应的自定义域与当前验证状态。在自定义安装中，现在你可以在“Azure AD 登录”页上选择用户主体名的属性。

![Azure AD 登录页](./media/active-directory-aadconnect-user-signin/custom_azure_sign_in.png)

可以单击“刷新”按钮，从 Azure AD 中重新提取最新的自定义域状态。

### 选择 Azure AD 中用户主体名的属性
UserPrincipalName - 属性 userPrincipalName 是用户登录 Azure AD 和 Office 365 时使用的属性。应在同步处理用户前在 Azure AD 中对使用的域（也称为 UPN 后缀）进行验证。强烈建议保留默认属性 userPrincipalName。如果此属性不可路由且无法验证，则可以选择另一个属性，例如，选择 email 作为保存登录 ID 的属性。这就是所谓的替代 ID。“替代 ID”属性值必须遵循 RFC822 标准。替代 ID 可以配合密码单一登录 (SSO) 和联合 SSO 作为登录解决方案来使用。

> [!NOTE]
所有 Office 365 工作负荷和直通身份验证都不允许使用替代 ID。有关详细信息，请参阅[配置替代登录 ID](https://technet.microsoft.com/zh-cn/library/dn659436.aspx)。
> 
> 

#### 不同的自定义域状态及其对 Azure 登录体验的影响
请务必要了解 Azure AD 目录中的自定义域状态与本地定义的 UPN 后缀之间的关系。让我们逐步了解当你使用 Azure AD Connnect 设置同步时可能遇到的不同 Azure 登录体验。

对于下面的信息，假设我们所关注的 UPN 后缀 contoso.com 在本地目录中用作 UPN 的一部分，例如 user@contoso.com。

###### 快速设置/密码同步
| 状态 | 对 Azure 用户登录体验的影响 |
|:---:|:--- |
| 未添加 |在这种情况下，Azure AD 目录中并未针对 contoso.com 添加任何自定义域。在本地具有后缀 @contoso.com 的 UPN 的用户将无法使用其本地 UPN 来登录 Azure。他们必须为默认的 Azure AD 目录添加后缀，以改用 Azure AD 提供的新 UPN。例如，如果你要将用户同步到 Azure AD 目录 azurecontoso.partner.onmschina.cn，则为本地用户 user@contoso.com 指定 UPN user@azurecontoso.partner.onmschina.cn |
| 未验证 |在这种情况下，我们已在 Azure AD 目录中添加自定义域 contoso.com，但它尚未验证。如果你不先验证域就继续同步用户，则 Azure AD 将为用户分配新 UPN，如同“未添加”方案中所做的一样。 |
| 已验证 |在这种情况下，我们已在 Azure AD 中为 UPN 后缀添加并验证自定义域 contoso.com。用户可以使用其本地用户主体名（例如 user@contoso.com），在其同步到 Azure AD 之后登录到 Azure |

###### AD FS 联合
你无法与 Azure AD 中的默认 .partner.onmschina.cn 域或 Azure AD 中未验证的自定义域创建联合。当你运行 Azure AD Connect 向导时，如果选择要与未验证的域创建联合，Azure AD Connect 将发出提示，并提供要创建的、包含域 DNS 托管位置的所需记录。有关详细信息，请参阅[此文](./active-directory-aadconnect-get-started-custom.md#verify-the-azure-ad-domain-selected-for-federation)。

如果选择的用户登录选项为“与 AD FS 联合”，则必须有一个自定义域才能继续在 Azure AD 中创建联合。在我们的介绍中，这意味着应在 Azure AD 目录中添加自定义域 contoso.com。

| 状态 | 对 Azure 用户登录体验的影响 |
|:---:|:--- |
| 未添加 |在这种情况下，Azure AD Connect 无法在 Azure AD 目录中找到 UPN 后缀 contoso.com 的匹配自定义域。如果需要让用户在 AD FS 中使用其本地 UPN（例如 user@contoso.com）登录，则需要添加自定义域 contoso.com |
| 未验证 |在这种情况下，Azure AD Connect 将发出提示，并提供有关如何在后面的阶段验证域的相应详细信息 |
| 已验证 |在这种情况下，你可以继续进行配置，而不需要采取任何进一步的操作 |

## 更改用户登录方法 <a name="changing-user-sign-in-method"></a>
可以在使用向导完成 Azure AD Connect 的初始配置后，使用 Azure AD Connect 中的可用任务，将用户的登录方法从“联合”更改为“密码同步”或“直通身份验证”。再次运行 Azure AD Connect 向导，随后你将看到可执行的任务列表。在任务列表中选择“更改用户登录”。

![更改用户登录](./media/active-directory-aadconnect-user-signin/changeusersignin.png)

在下一页，系统将请求你提供 Azure AD 的凭据。

![连接到 Azure AD](./media/active-directory-aadconnect-user-signin/changeusersignin2.png)  

在“用户登录”页上，选择所需的用户登录选项。

![连接到 Azure AD](./media/active-directory-aadconnect-user-signin/changeusersignin2a.png)  

> [!NOTE]
如果你只是要暂时切换到密码同步，请选中“不要转换用户帐户”。不选中该选项会导致将每个用户转换为联合用户，并可能需要花费几小时。
> 
> 

## 后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](./active-directory-aadconnect.md)的详细信息。

了解有关 [Azure AD Connect：设计概念](./active-directory-aadconnect-design-concepts.md)的详细信息

<!---HONumber=Mooncake_1226_2016-->