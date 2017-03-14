---
title: 在 Azure 门户预览中使用基于角色的访问控制 | Azure
description: 在 Azure 门户预览中使用基于角色的访问控制开始进行访问权限管理。使用角色分配来分配资源权限。
services: active-directory
documentationCenter: ''
authors: kgremban
manager: femila
editor: ''

ms.service: active-directory
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 10/10/2016
wacn.date: 11/08/2016
ms.author: kgremban
---

# 使用角色分配来管理对 Azure 订阅资源的访问权限

Azure 基于角色的访问控制 (RBAC) 可用于对 Azure 进行细致的访问管理。使用 RBAC，你可以仅授予用户执行其作业所需的访问次数。本文可帮助你在 Azure 门户预览中启动并运行 RBAC。如果希望了解有关 RBAC 如何帮助管理访问权限的更多详细信息，请参阅[什么是基于角色的访问控制](./role-based-access-control-what-is.md)。

## 查看访问权限
你可以在 [Azure 门户预览](https://portal.azure.cn)中的主边栏选项卡上查看谁有权访问资源、资源组或订阅。例如，我们想要查看谁有权访问其中一个资源组：

1. 在左侧的导航栏中选择“资源组”。
    ![资源组 - 图标](./media/role-based-access-control-configure/resourcegroups_icon.png)
2. 在“资源组”边栏选项卡上，选择资源组的名称。
3. 从左侧菜单中选择“访问控制 (IAM)”。
4. “访问控制”边栏选项卡列出了授予该资源组访问权限的所有用户、组和应用程序。

    ![用户边栏选项卡 - 继承的与分配的访问权限屏幕截图](./media/role-based-access-control-configure/view-access.png)  

请注意，某些用户为**分配**的访问权限，而其他人为**继承**的访问权限。访问权限要么特定于资源组进行分配，要么从对父订阅的分配继承而来。

> [!NOTE]
> 经典订阅管理员和共同管理员被视为新 RBAC 模型中的订阅所有者。

## <a name="add-access"></a> 添加访问权限
从资源、资源组或在该角色分配范围内的订阅中授予访问权限。

1. 在“访问控制”边栏选项卡上选择“添加”。
2. 从“选择角色”边栏选项卡中选择你想要分配的角色。
3. 在你想要授予访问权限的目录中选择用户、组或应用程序。你可以通过显示名称、电子邮件地址和对象标识符搜索该目录。

    ![添加用户边栏选项卡 - 搜索屏幕截图](./media/role-based-access-control-configure/grant-access2.png)

4. 选择“确定”以创建分配。“添加用户”弹出窗口跟踪进度。
    ![添加用户进度栏 - 屏幕截图](./media/role-based-access-control-configure/addinguser_popup.png)

成功添加角色分配后，该角色分配将出现在“用户”边栏选项卡上。

## 删除访问权限

1. 在“访问控制”边栏选项卡上选择角色分配。
2. 选择“分配详细信息”边栏选项卡中的“删除”。
3. 选择“确定”以确认删除操作。
    ![用户边栏选项卡 - 从角色中删除屏幕截图](./media/role-based-access-control-configure/remove-access1.png)

不能删除继承的分配。请注意，在下图中，“删除”按钮为灰显。相反，看看“分配在”的详细信息。请转到该处列出的资源以删除角色分配。

![用户边栏选项卡 - 继承的访问权限禁用删除按钮屏幕截图](./media/role-based-access-control-configure/remove-access2.png)

## 管理访问权限的其他工具
可以使用 Azure 门户预览以外的工具中的 Azure RBAC 命令分配角色和管理访问权限。依照该链接以了解有关先决条件和 Azure RBAC 命令入门的详细信息。

- [Azure PowerShell](./role-based-access-control-manage-access-powershell.md)
- [Azure 命令行界面](./role-based-access-control-manage-access-azure-cli.md)
- [REST API](./role-based-access-control-manage-access-rest.md)

## 后续步骤
- [创建访问变更历史记录报表](./role-based-access-control-access-change-history-report.md)
- 请参阅 [RBAC 内置角色](./role-based-access-built-in-roles.md)
- 定义自己在 [Azure RBAC 中的自定义角色](./role-based-access-control-custom-roles.md)

<!---HONumber=Mooncake_1031_2016-->