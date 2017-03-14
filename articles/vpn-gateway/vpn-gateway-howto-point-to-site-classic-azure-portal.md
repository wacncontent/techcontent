---
title: 使用点到站点将计算机连接到 Azure 虚拟网络：Azure 门户预览：经典 | Azure
description: 使用 Azure 门户预览创建点到站点 VPN 网关连接，从而安全连接到经典 Azure 虚拟网络。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: azure-service-management

ms.assetid: 65e14579-86cf-4d29-a6ac-547ccbd743bd
ms.service: vpn-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/17/2017
wacn.date: 03/03/2017
ms.author: cherylmc
---

# 使用 Azure 门户预览配置与 VNet 的点到站点连接
> [!div class="op_single_selector"]
- [Resource Manager - Azure 门户预览](./vpn-gateway-howto-point-to-site-resource-manager-portal.md)
- [Resource Manager - PowerShell](./vpn-gateway-howto-point-to-site-rm-ps.md)
- [经典 - Azure 门户预览](./vpn-gateway-howto-point-to-site-classic-azure-portal.md)

通过点到站点 (P2S) 配置，可以创建单台客户端计算机到虚拟网络的安全连接。如果要从远程位置（例如从家里或会议室）连接到 VNet，或者只有少数几个需要连接到虚拟网络的客户端，则 P2S 连接会很有用。

点到站点连接不需要 VPN 设备或面向公众的 IP 地址即可运行。可通过从客户端计算机启动连接来建立 VPN 连接。有关点到站点连接的详细信息，请参阅本文末尾的[点到站点常见问题解答](#faq)。

本文逐步讲解如何使用 Azure 门户预览，在经典部署模型中创建具有点到站点连接的 VNet。

### P2S 连接的部署模型和方法
[!INCLUDE [部署模型](../../includes/vpn-gateway-deployment-models-include.md)]

下表显示了 P2S 配置的两种部署模型和可用的部署方法。当有配置步骤相关的文章发布时，我们会直接从此表格链接到该文章。

[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-table-point-to-site-include.md)]

## 基本工作流
![点到站点连接示意图](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/point-to-site-connection-diagram.png)  

以下部分逐步讲解如何建立与虚拟网络的安全点到站点连接。

1. 创建虚拟网络和 VPN 网关
2. 生成证书
3. 上载 .cer 文件
4. 生成 VPN 客户端配置包
5. 配置客户端计算机
6. 连接到 Azure

### 示例设置
可以使用以下示例设置：

* **名称：VNet1**
* **地址空间：192.168.0.0/16**
* **子网名称：FrontEnd**
* **子网地址范围：192.168.1.0/24**
* **订阅：**如果你有多个订阅，请确保使用正确的订阅。
* **资源组：TestRG**
* **位置：中国东部**
* **连接类型：点到站点**
* **客户端地址空间：172.16.201.0/24**。使用此点到站点连接连接到 VNet 的 VPN 客户端接收来自指定池的 IP 地址。
* **网关子网：192.168.200.0/24**。网关子网必须使用名称“GatewaySubnet”。
* **大小：**选择要使用的网关 SKU。
* **路由类型：动态**

## <a name="vnetvpn"></a>第 1 部分 - 创建虚拟网络和 VPN 网关
### <a name="createvnet"></a>第 1 部分：创建虚拟网络
如果还没有虚拟网络，请创建一个。这些屏幕截图仅供参考。请务必替换为你自己的值。若要使用 Azure 门户预览创建 VNet，请执行以下步骤：

1. 从浏览器导航到 [Azure 门户预览](http://portal.azure.cn)，必要时使用 Azure 帐户登录。
2. 单击“新建”。在“搜索应用商店”字段中，键入“虚拟网络”。从返回的列表中找到“虚拟网络”，单击打开“虚拟网络”边栏选项卡。

    ![搜索虚拟网络边栏选项卡](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/newvnetportal700.png)  

3. 从靠近“虚拟网络”边栏选项卡底部的“选择部署模型”列表中，选择“经典”，然后单击“创建”。

    ![选择部署模型](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/selectmodel.png)  

4. 在“创建虚拟网络”边栏选项卡上，配置 VNet 设置。在此边栏选项卡中，你将添加第一个地址空间和单个子网地址范围。完成创建 VNet 之后，可以返回并添加其他子网和地址空间。

    ![创建虚拟网络边栏选项卡](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/vnet125.png)  

5. 确认“订阅”正确无误。可以使用下拉列表更改订阅。
6. 单击“资源组”并选择现有的资源组，或键入新的资源组名称创建新资源组。如果要创建新组，请根据计划的配置值来命名资源组。有关资源组的详细信息，请访问 [Azure 资源管理器概述](../azure-resource-manager/resource-group-overview.md#resource-groups)。
7. 接下来，选择 VNet 的“位置”设置。该位置决定了要部署到此 VNet 中的资源所在的位置。
8. 如果想要在仪表板上轻松找到 VNet，请选择“固定到仪表板”，然后单击“创建”。

    ![固定到仪表板](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/pintodashboard150.png)  

9. 单击“创建”后，你将看到仪表板上有一个磁贴反映了 VNet 的进度。创建 VNet 时，该磁贴会更改。

    ![创建虚拟网络磁贴](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/deploying150.png)  

10. 创建虚拟网络后，可以添加 DNS 服务器的 IP 地址来处理名称解析。打开虚拟网络的设置，单击 DNS 服务器，然后添加要使用的 DNS 服务器 IP 地址。此设置不会创建新的 DNS 服务器。请务必添加可与资源通信的 DNS 服务器。

创建虚拟网络后，可以在 Azure 经典管理门户中的“网络”页上，看到“状态”下面列出了“已创建”。

### <a name="gateway"></a>第 2 部分：创建网关子网和动态路由网关
本步骤创建网关子网和动态路由网关。在经典部署模型的 Azure 门户预览中，可以通过相同的配置边栏选项卡创建网关子网和网关。

1. 在门户中，导航到要为其创建网关的虚拟网络。
2. 在虚拟网络“概述”边栏选项卡上的“VPN 连接”部分中，单击“网关”。

    ![单击创建网关](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/beforegw125.png)  

3. 在“新建 VPN 连接”边栏选项卡中，选择“点到站点”。

    ![点到站点连接类型](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/newvpnconnect.png)  

4. 对于“客户端地址空间”，请添加 IP 地址范围。这是 VPN 客户端在连接时要从中接收 IP 地址的范围。删除自动填充的范围，添加自己的范围。

    ![客户端地址空间](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clientaddress.png)  

5. 选中“立即创建网关”复选框。

    ![立即创建网关](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/creategwimm.png)  

6. 单击“可选网关配置”打开“网关配置”边栏选项卡。

    ![单击可选网关配置](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/optsubnet125.png)  

7. 单击“子网配置所需的设置”添加**网关子网**。尽管创建的网关子网最小可为 /29，但建议至少选择 /28 或 /27，创建包含更多地址的更大子网。这样便可以留出足够多的地址，满足将来可能需要使用的其他配置。

    > [!IMPORTANT]
    处理网关子网时，请避免将网络安全组 (NSG) 关联到网关子网。将网络安全组与此子网关联可能会导致 VPN 网关停止按预期方式工作。有关网络安全组的详细信息，请参阅 [What is a network security group?](../virtual-network/virtual-networks-nsg.md)（什么是网络安全组？）
    >
    >

    ![添加网关子网](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/gwsubnet125.png)  

8. 选择网关**大小**。这是用于创建虚拟网络网关的网关 SKU。在门户中，默认 SKU 为“基本”。有关网关 SKU 的详细信息，请参阅 [About VPN Gateway Settings](./vpn-gateway-about-vpn-gateway-settings.md#gwsku)（关于 VPN 网关设置）。

    ![网关大小](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/gwsize125.png)  

9. 选择网关的**路由类型**。P2S 配置需要**动态**路由类型。在此边栏选项卡中完成配置后，请单击“确定”。

    ![配置路由类型](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/routingtype125.png)  

10. 在“新建 VPN 连接”边栏选项卡中，单击底部的“确定”开始创建虚拟网络网关。此步骤最多需要 45 分钟才能完成。

## <a name="generatecerts"></a>第 2 节 - 生成证书
Azure 使用证书对点到站点 VPN 的 VPN 客户端进行身份验证。从企业证书解决方案生成的根证书或自签名根证书将公共证书数据（不是私钥）导出为 Base-64 编码 X.509 .cer 文件。然后从根证书将公共证书数据导入到 Azure。此外，需要从客户端的根证书生成客户端证书。每个要使用 P2S 连接连接到虚拟网络的客户端都必须已安装从根证书生成的客户端证书。

### <a name="cer"></a>第 1 部分：获取根证书的 .cer 文件
如果使用的是企业解决方案，可以使用现有的证书链。如果使用的不是企业 CA 解决方案，可以创建自签名根证书。建议使用 [makecert](./vpn-gateway-certificates-point-to-site.md) 方法创建用于 P2S 连接的自签名证书。虽然可以使用 PowerShell 创建自签名证书，但使用 PowerShell 生成的证书不包含 P2S 连接所需的字段。

* 如果使用企业证书解决方案，请获取要使用的根证书的 .cer 文件。
* 如果使用的不是企业证书解决方案，则需要[使用 makecert 生成自签名根证书](./vpn-gateway-certificates-point-to-site.md)。

1. 若要从证书中获取 .cer 文件，请打开 **certmgr.msc** 并找到根证书。右键单击自签名根证书，单击“所有任务”，然后单击“导出”。此操作将打开“证书导出向导”。
2. 在向导中，单击“下一步”，选择“否，不导出私钥”，然后单击“下一步”。
3. 在“导出文件格式”页上，选择“Base-64 编码的 X.509 (.CER)”。 然后，单击“下一步”。
4. 在“要导出的文件”中，单击“浏览”并选择要导出证书的位置。在“文件名”中，为证书文件命名。然后单击“下一步”。
5. 单击“完成”以导出证书。

### <a name="genclientcert"></a>第 2 部分：生成客户端证书
可以为每个要连接的客户端生成唯一证书，也可以在多个客户端上使用相同的证书。生成唯一客户端证书的优势是能够根据需要吊销单个证书。否则，如果每个人都使用相同的客户端证书，在需要吊销某个客户端的证书时，必须为所有使用该证书进行身份验证的客户端生成并安装新证书。

####企业证书
- 如果使用的是企业证书解决方案，请使用通用名称值格式“name@yourdomain.com”生成客户端证书，而不要使用“域名\\用户名”格式。
- 请确保颁发的客户端证书基于“用户”证书模板，该模板使用“客户端身份验证”作为使用列表中的第一项，而不是智能卡登录等。可以通过双击客户端证书，并查看“详细信息”>“增强型密钥使用”来检查证书。

####自签名证书 
如果使用自签名的证书解决方案，请参阅 [Working with self-signed root certificates for Point-to-Site configurations](./vpn-gateway-certificates-point-to-site.md)（为点到站点配置使用自签名根证书）生成客户端证书。

### <a name="exportclientcert"></a>第 3 部分：导出客户端证书
在要连接到虚拟网络的每台计算机上安装客户端证书。身份验证时需要客户端证书。可以自动安装客户端证书，也可以手动安装。以下步骤指导如何手动导出和安装客户端证书。

1. 若要导出客户端证书，可以使用 *certmgr.msc*。右键单击要导出的客户端证书，单击“所有任务”，然后单击“导出”。
2. 导出包含私钥的客户端证书。这是一个 *.pfx* 文件。请确保记录或记住为此证书设置的密码（密钥）。

## <a name="upload"></a>第 3 节 - 上载根证书 .cer 文件
创建网关后，可以将受信任根证书的 .cer 文件上载到 Azure。最多可以上载 20 个根证书的文件。不要将根证书的私钥上载到 Azure。上载 .Cer 文件后，Azure 将使用它来对连接到虚拟网络的客户端进行身份验证。

1. 在 VNet 边栏选项卡的“VPN 连接”部分中，单击“客户端”图形打开“点到站点 VPN 连接”边栏选项卡。

    ![客户端](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clients125.png)  

2. 在“点到站点连接”边栏选项卡中，单击“管理证书”打开“证书”边栏选项卡。<br>

    ![证书边栏选项卡](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/ptsmanage.png)<br><br>
3. 在“证书”边栏选项卡中，单击“上载”打开“上载证书”边栏选项卡。<br>

    ![上载证书边栏选项卡](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/uploadcerts.png)
4. 单击文件夹图形浏览 .cer 文件。选择该文件，然后单击“确定”。刷新页面，在“证书”边栏选项卡中查看上载的证书。

    ![上载证书](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/upload.png)

## <a name="vpnclientconfig"></a>第 4 节 - 生成 VPN 客户端配置包
若要连接到虚拟网络，还需配置 VPN 客户端。客户端计算机要求提供客户端证书和正确的 VPN 客户端配置包来建立连接。

VPN 客户端包中含有用于配置 Windows 内置 VPN 客户端软件的配置信息。该程序包不安装额外的软件。这些设置特定于要连接到的虚拟网络。有关支持的客户端操作系统列表，请参阅本文末尾的[点到站点连接常见问题解答](#faq)。

### 生成 VPN 客户端配置包
1. 在 Azure 门户预览中，在 VNet“概述”边栏选项卡上的“VPN 连接”中，单击客户端图形打开“点到站点 VPN 连接”边栏选项卡。
2. 在“点到站点 VPN 连接”边栏选项卡顶部，选择与要在其中进行安装的客户端操作系统对应的下载包：

    * 对于 64 位客户端，请选择“VPN 客户端(64 位)”。
    * 对于 32 位客户端，请选择“VPN 客户端(32 位)”。

     ![下载 VPN 客户端配置包](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/dlclient.png)
3. 此时将显示一条消息，指出 Azure 正在为虚拟网络生成 VPN 客户端配置包。几分钟后，将会生成包，同时本地计算机上会显示一条消息，指出已下载包。保存配置包文件。需要在使用 P2S 连接到虚拟网络的每台客户端计算机上安装此文件。

## <a name="clientconfiguration"></a>第 5 节 - 配置客户端计算机
### 第 1 部分：安装客户端证书
每个客户端计算机必须有一个客户端证书才能执行身份验证。安装客户端证书时，需要使用导出客户端证书时创建的密码。

1. 将 .pfx 文件复制到客户端计算机。
2. 双击 .pfx 文件将它安装。请勿修改安装位置。

### 第 2 部分：安装 VPN 客户端配置包
如果版本与客户端的体系结构匹配，可以在每台客户端计算机上使用相同的 VPN 客户端配置包。

1. 将配置文件通过本地方式复制到需要连接到虚拟网络的计算机上，然后双击 .exe 文件。
2. 安装完程序包后，即可启动 VPN 连接。Microsoft 没有对配置包进行签名。可能需要使用组织的签名服务对程序包进行签名，也可以自行使用 [SignTool](http://go.microsoft.com/fwlink/p/?LinkId=699327) 对其签名。可以使用不签名的程序包。但如果程序包未签名，则在安装该程序包时，会显示一条警告。
3. 在客户端计算机上，导航到“网络设置”，然后单击“VPN”。此时将会列出连接。其中显示了要连接到的虚拟网络的名称，如下所示：

    ![VPN 客户端](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/vpn.png)  

## <a name="connect"></a>第 6 节 - 连接到 Azure
### 连接到 VNet
1. 若要连接到 VNet，请在客户端计算机上导航到 VPN 连接，找到创建的 VPN 连接。其名称与虚拟网络的名称相同。单击“连接”。可能会出现与使用证书相关的弹出消息。如果出现此消息，请单击“继续”以使用提升的权限。
2. 在“连接”状态页上，单击“连接”开始连接。如果你看到“选择证书”屏幕，请确保所显示的客户端证书就是你要用来连接的证书。如果不是，请使用下拉箭头选择正确的证书，然后单击“确定”。

    ![VPN 客户端连接](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clientconnect.png)  

3. 现在应已建立连接。

    ![已建立连接](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/connected.png)  

> [!NOTE]
如果使用的是通过企业 CA 解决方案颁发的证书，并且无法进行身份验证，请检查客户端证书上的身份验证顺序。可以通过双击客户端证书，并转到“详细信息”>“增强型密钥使用”来检查身份验证列表顺序。请确保此列表显示的第一项是“客户端身份验证”。如果不是，则需要基于将“客户端身份验证”作为列表中第一项的用户模板颁发客户端证书。
>
>

### 验证 VPN 连接
1. 若要验证你的 VPN 连接是否处于活动状态，请打开提升的命令提示符，然后运行 *ipconfig/all*。
2. 查看结果。请注意，你收到的 IP 地址是点到站点连接地址范围中的一个地址，该范围是你在创建 VNet 时指定的。结果应大致如下所示：

示例：

```
PPP adapter VNet1:
    Connection-specific DNS Suffix .:
    Description.....................: VNet1
    Physical Address................:
    DHCP Enabled....................: No
    Autoconfiguration Enabled.......: Yes
    IPv4 Address....................: 192.168.130.2(Preferred)
    Subnet Mask.....................: 255.255.255.255
    Default Gateway.................:
    NetBIOS over Tcpip..............: Enabled
```

## <a name="faq"></a>点到站点常见问题解答

[!INCLUDE [点到站点常见问题解答](../../includes/vpn-gateway-point-to-site-faq-include.md)]

## 后续步骤
连接完成后，即可将虚拟机添加到虚拟网络。有关详细信息，请参阅[虚拟机](../virtual-machines/index.md)。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: wording update and add FAQ-->