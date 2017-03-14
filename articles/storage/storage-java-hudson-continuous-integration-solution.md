---
title: 如何将 Hudson 与 Blob 存储一起使用 | Azure
description: 介绍如何将 Hudson 与 Azure Blob 存储一起用作生成项目的存储库。
services: storage
documentationcenter: java
author: dineshmurthy
manager: jahogg
editor: tysonn

ms.assetid: 119becdd-72c4-4ade-a439-070233c1e1ac
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 10/18/2016
wacn.date: 12/26/2016
ms.author: dinesh
---

# 将 Azure 存储用于 Hudson 持续集成解决方案
## 概述
下列信息演示了如何将 Blob 存储用作 Hudson 持续集成 (CI) 解决方案创建的生成项目的存储库，或者用作要在生成过程中使用的可下载文件的源。在以下情况中你将会发现这一做法很有用：你在敏捷开发环境进行编码（使用 Java 或其他语言），生成基于持续集成运行并且你需要一个适用于生成项目的存储库，以便（举例来说）你能与其他组织成员、你的客户共享生成项目或维护存档。另一种情况是，当你的生成作业本身需要其他文件时，例如需要下载依赖项作为生成输入的一部分时。

在本教程中，你将使用 Microsoft 提供的适用于 Hudson CI 的 Azure 存储插件。

## Hudson 简介
Hudson 通过允许开发人员轻松地集成其代码更改以及自动和频繁地生成版本，实现了软件项目的持续集成，因此提高了开发人员的工作效率。生成通过版本控制，并且可将生成项目上传到不同存储库。本文将演示如何将 Azure Blob 存储用作生成项目的存储库。它还将演示如何从 Azure Blob 存储下载依赖项。

有关 Hudson 的详细信息，请访问[了解 Hudson](http://wiki.eclipse.org/Hudson-ci/Meet_Hudson)。

## 使用 Blob 服务的好处
使用 Blob 服务承载敏捷开发生成项目的好处包括：

* 生成项目和/或可下载依赖项的高可用性。
* Hudson CI 解决方案上传生成项目时的性能。
* 客户和合作伙伴下载生成项目时的性能。
* 通过选择匿名访问、基于过期的共享访问、签名访问、专用访问等控制用户访问策略。

## 先决条件
你将需要以下内容才能将 Blob 服务用于 Hudson CI 解决方案：

* 一个 Hudson 持续集成解决方案。

    如果你当前没有 Hudson CI 解决方案，可以使用以下技术运行一个 Hudson CI 解决方案：

  1. 在已启用 Java 的计算机上，从 <http://hudson-ci.org/> 下载 Hudson WAR。
  2. 在打开到包含 Hudson WAR 的文件夹的命令提示符下，运行 Hudson WAR。例如，如果你下载了版本 3.1.2：

      `java -jar hudson-3.1.2.war`

  3. 在浏览器中，打开 `http://localhost:8080/`。这将打开 Hudson 仪表板。
  4. 首次使用 Hudson 时，在 `http://localhost:8080/` 上完成初始设置。
  5. 在完成初始设置后，取消 Hudson WAR 的正在运行的实例、再次启动 Hudson WAR，然后重新打开 Hudson 仪表板 (`http://localhost:8080/`)，你将使用它安装和配置 Azure 存储插件。

      虽然典型 Hudson CI 解决方案将设置为作为一个服务运行，但在本教程中，通过命令行运行 Hudson WAR 就足够了。
* 一个 Azure 帐户。注册 Azure 帐户的位置位于 <https://www.azure.cn>。
* 一个 Azure 存储帐户。如果你还没有存储帐户，则可使用[创建存储帐户](./storage-create-storage-account.md#create-a-storage-account)中的步骤创建一个。
* 建议熟悉 Hudson CI 解决方案（但不是必需的），因为以下内容将使用一个基本示例向你演示使用 Blob 服务作为 Hudson CI 生成项目的存储库时所需的步骤。

## 如何将 Blob 服务用于 Hudson CI
若要将 Blob 服务用于 Hudson，你将需要安装 Azure 存储插件，并对该插件进行配置以使用你的存储帐户，然后创建一个将生成项目上传到你的存储帐户的生成后操作。以下部分介绍了这些步骤。

## 如何安装 Azure 存储插件
1. 在 Hudson 仪表板中，单击“管理 Hudson”。
2. 在“管理 Hudson”页中，单击“管理插件”。
3. 单击“可用”选项卡。
4. 单击“其他”。
5. 在“项目上载程序”部分中，选择“Azure 存储插件”。
6. 单击“安装”。
7. 安装完毕后，重新启动 Hudson。

## 如何配置 Azure 存储插件以使用你的存储帐户
1. 在 Hudson 仪表板中，单击“管理 Hudson”。
2. 在“管理 Hudson”页中，单击“配置系统”。
3. 在“Azure 存储帐户配置”部分：

    a.输入你的存储帐户名称，可以从 [Azure 门户预览](https://portal.azure.cn)获取该帐户名称。

    b.输入你的存储帐户密钥，同样可以从 [Azure 门户预览](https://portal.azure.cn)获取该密钥。

    c.如果你在使用公共 Azure 云，对于“Blob 服务终结点 URL”，请使用默认值。如果你在使用其他 Azure 云，则使用在 [Azure 门户预览](https://portal.azure.cn)中为你的存储帐户指定的终结点。

    d.单击“验证存储凭据”以验证你的存储帐户。

    e. [可选] 如果你有其他存储帐户并且希望其可供 Hudson CI 使用，请单击“添加更多存储帐户”。

    f.单击“保存”以保存你的设置。

## 如何创建将你的生成项目上传到存储帐户的生成后操作
为了进行说明，首先我们将需要创建一个将创建若干文件的作业，然后添加在生成后操作中以将文件上传到存储帐户。

1. 在 Hudson 仪表板中，单击“新建作业”。
2. 将此作业命名为“MyJob”，单击“生成自由格式的软件作业”，然后单击“确定”。
3. 在作业配置的“生成”部分，单击“添加生成步骤”并选择“执行 Windows 批处理命令”。
4. 在“命令”中，使用下列命令：

    ```
    md text
    cd text
    echo Hello Azure Storage from Hudson > hello.txt
    date /t > date.txt
    time /t >> date.txt
    ```

5. 在作业配置的“后期生成操作”部分，单击“将项目上载到 Azure Blob 存储”。
6. 对于“存储帐户名称”，选择要使用的存储帐户。
7. 对于“容器名称”，请指定容器名称。（如果上传生成项目时不存在该容器，则将创建该容器。） 你可使用环境变量，因此在此示例中，请输入 **${JOB\_NAME}** 作为容器名称。

    **提示**

    在你为“执行 Windows 批处理命令”输入脚本的“命令”部分下方，有一个指向 Hudson 所识别环境变量的链接。单击此链接可了解环境变量名称和说明。请注意，不允许将包含特殊字符的环境变量（如 **BUILD\_URL** 环境变量）用作容器名称或通用虚拟路径。
8. 对于此示例，请单击“默认将新容器设为公开的”。（如果要使用私有容器，你将需要创建共享访问签名以允许访问。这超出了本文的范围。可以在[创建共享访问签名 (SAS)](./storage-dotnet-shared-access-signature-part-1.md)中了解有关共享访问签名的详细信息。
9. [可选]如果你希望在上载生成项目之前清除容器的内容，请单击“在上载前清除容器”（如果你不希望清除容器的内容，则使该复选框保持未选中状态）。
10. 对于“要上传的项目列表”，请输入 **text/*.txt**。
11. 对于“已上载项目的通用虚拟路径”，输入 **${BUILD\_ID}/${BUILD\_NUMBER}**。
12. 单击“保存”以保存你的设置。
13. 在 Hudson 仪表板中，单击“立即生成”以运行 **MyJob**。检查控制台输出中的状态。当生成后操作开始上传生成项目时，Azure 存储的状态消息将包括在控制台输出中。
14. 成功完成此作业后，你可通过打开公共 Blob 检查生成项目。

    a.登录到 [Azure 门户预览](https://portal.azure.cn)。

    b.单击“存储”。

    c.单击你用于 Hudson 的存储帐户名称。

    d.单击“容器”。

    e.单击名为 **myjob** 的容器，该名称是你创建 Hudson 作业时分配的作业名称的小写形式。在 Azure 存储中，容器名称和 Blob 名称都采用小写形式（并且区分大小写）。在名为 **myjob** 的容器的 Blob 列表中，你应该能看到 **hello.txt** 和 **date.txt**。复制这两项中任一项的 URL 并在浏览器中打开。你将看到已作为生成项目上传的文本文件。

每个作业只能创建一个用于将项目上传到 Azure Blob 存储的生成后操作。请注意，用来将项目上传到 Azure Blob 存储的单个生成后操作可以在“要上传的项目列表”中使用分号作为分隔符指定不同的文件（包括通配符）和文件路径。例如，如果你的 Hudson 生成在你的工作区的 **build** 文件夹中生成了 JAR 文件和 TXT 文件，并且你希望将这两者都上传到 Azure Blob 存储，请使用以下项作为“要上传的项目列表”值：**build/*.jar;build/*.txt**。你还可以使用双冒号语法指定要在 Blob 名称内使用的路径。例如，如果你希望在 Blob 路径中使用 **binaries** 以上载 JAR 并在 Blob 路径中使用 **notices** 以上载 TXT 文件，请使用以下项作为“要上载的项目列表”值：**build/*.jar::binaries;build/*.txt::notices**。

## 如何创建从 Azure Blob 存储进行下载的生成步骤
以下步骤演示了如何配置从 Azure Blob 存储下载项目的生成步骤。如果你希望在你的生成中包括某些项（例如你保存在 Azure Blob 存储中的 JAR），这将非常有用。

1. 在作业配置的“生成”部分中，单击“添加生成步骤”并选择“从 Azure Blob 存储下载”。
2. 对于“存储帐户名称”，选择要使用的存储帐户。
3. 对于“容器名称”，指定包含你要下载的 Blob 的容器的名称。你可以使用环境变量。
4. 对于“Blob 名称”，指定 Blob 名称。你可以使用环境变量。另外，在指定 Blob 名称的初始字母后，你可以使用星号作为通配符。例如，**project*** 将指定其名称以 **project** 开头的所有 Blob。
5. [可选] 对于“下载路径”，指定 Hudson 计算机上你希望将文件从 Azure Blob 存储下载到的路径。也可以使用环境变量。（如果你没有为“下载路径”提供值，则 Azure Blob 存储中的文件将被下载到作业的工作区中。）

如果你还希望从 Azure Blob 存储下载其他项，可以创建其他生成步骤。

在运行生成后，你可以检查生成历史记录控制台输出或查看你的下载位置，看看是否成功下载了所需的 Blob。

## Blob 服务使用的组件
以下信息概述了 Blob 服务组件。

* **存储帐户**：对 Azure 存储的所有访问都要通过存储帐户来完成。存储帐户是访问 blob 的最高级别的命名空间。一个帐户可以包含无限个容器，只要这些容器的总大小不超过 100 TB 即可。
* **容器**：一个容器包含一组 blob 集。所有 blob 必须位于相应的容器中。一个帐户可以包含无限个容器。一个容器可以存储无限个 Blob。
* **Blob**：任何类型和大小的文件。可将两类 Blob 存储到 Azure 存储中：块 Blob 和页 Blob。大部分文件都是块 blob。单个块 Blob 最大可以为 200 GB。本教程使用块 Blob。另一种 Blob 类型为页 Blob，其大小可以达 1 TB，在对文件中的一系列字节进行频繁修改时，这种 Blob 类型更加高效。有关 Blob 的更多信息，请参阅[了解块 Blob、追加 Blob 和页 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx)。
* **URL 格式**：可使用以下 URL 格式对 Blob 寻址：

    `http://storageaccount.blob.core.chinacloudapi.cn/container_name/blob_name`

    （以上格式适用于公共 Azure 云。如果你在使用其他 Azure 云，请使用 [Azure 门户预览](https://portal.azure.cn)中的终结点来确定你的 URL 终结点。）

    在以上格式中，`storageaccount` 表示存储帐户的名称，`container_name` 表示容器的名称，而 `blob_name` 表示 Blob 的名称。在容器名称中，你可具有多个由正斜杠 ** /** 分隔的路径。本教程的示例容器名称为 **MyJob**，**${BUILD\_ID}/${BUILD\_NUMBER}** 用于通用虚拟路径，从而导致 Blob 具有以下格式的 URL：

    `http://example.blob.core.chinacloudapi.cn/myjob/2014-05-01_11-56-22/1/hello.txt`

## 后续步骤
- [Hudson 简介](http://wiki.eclipse.org/Hudson-ci/Meet_Hudson)
- [Azure Storage SDK for Java](https://github.com/azure/azure-storage-java)
- [Azure 存储客户端 SDK 参考](http://azure.github.io/azure-storage-java/)
- [Azure 存储服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx)
- [Azure 存储团队博客](http://blogs.msdn.com/b/windowsazurestorage/)

有关详细信息，请参阅 [Java 开发人员中心](/develop/java/)。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->