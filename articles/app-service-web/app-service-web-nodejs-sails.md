---
title: 将 Sails.js Web 应用部署到 Azure App Service | Azure
description: 学习如何将 Node.js 应用部署到 Azure App Service。本教程说明如何部署 Sails.js Web 应用。
services: app-service\web
documentationcenter: nodejs
author: cephalin
manager: wpickett
editor: ''

ms.assetid: 8877ddc8-1476-45ae-9e7f-3c75917b4564
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 12/16/2016
wacn.date: 01/03/2017
ms.author: cephalin
---

# 将 Sails.js Web 应用部署到 Azure App Service

[!INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本教程说明如何将 Sails.js 应用部署到 Azure App Service。在此过程中，可以搜集一些有关如何将 Node.js 应用配置为在应用服务中运行的一般知识。

在这里，可学习的有用技能包括：

* 配置在应用服务中运行的 Sails.js 应用。
* 从命令行将应用部署到应用服务中。
* 读取 stderr 和 stdout 日志，对任何部署问题进行故障排除。
* 在源控件以外存储环境变量。
* 从应用访问 Azure 环境变量。
* 连接到数据库 (MongoDB)。

应具备 Sails.js 的实践知识。本教程并非旨在帮助你解决运行 Sail.js 相关的一般性问题。

## 用于完成任务的 CLI 版本

可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](./app-service-web-nodejs-sails-cli-nodejs.md)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0（预览版）](./app-service-web-nodejs-sails.md)：用于资源管理部署模型的下一代 CLI

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## 先决条件
* [Node.js](https://nodejs.org/)
* [Sails.js](http://sailsjs.org/get-started)
* [Git](http://www.git-scm.com/downloads)
* [Azure CLI 2.0 预览版](https://docs.microsoft.com/cli/azure/install-az-cli2)
* 一个 Azure 帐户。如果你没有帐户，可以[注册试用版](https://www.azure.cn/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

## 步骤 1：在本地创建和配置 Sails.js 应用
首先，请执行以下步骤，在部署环境中快速创建默认的 Sails.js 应用：

1. 打开所选的命令行终端，并使用 `CD` 切换到工作目录。
2. 创建 Sails.js 应用并运行：

    ```
    sails new <app_name>
    cd <app_name>
    sails lift
    ```

    确保可以导航到 http://localhost:1377 上的默认主页。

1. 接下来，为 Azure 启用日志记录。在根目录中，创建名为 `iisnode.yml` 的文件并添加以下两行：

    ```
    loggingEnabled: true
    logDirectory: iisnode
    ```

    现在已为 Azure App Service 用于运行 Node.js 应用的 [iisnode](https://github.com/tjanczuk/iisnode) 服务器启用日志记录。有关具体操作的详细信息，请参阅[从 iisnode 获取 stdout 和 stderr 日志](./app-service-web-nodejs-get-started.md#iisnodelog)。

2. 接下来，配置 Sails.js 应用使用 Azure 环境变量。打开 config/env/production.js 来配置生产环境，并设置 `port` 和 `hookTimeout`：

    ```
    module.exports = {

        // Use process.env.port to handle web requests to the default HTTP port
        port: process.env.port,
        // Increase hooks timout to 30 seconds
        // This avoids the Sails.js error documented at https://github.com/balderdashy/sails/issues/2691
        hookTimeout: 30000,

        ...
    };
    ```

    可以在 [Sails.js 文档](http://sailsjs.org/documentation/reference/configuration/sails-config)中找到这些配置设置的文档。

4. 接下来，对想要使用的 Node.js 版本进行硬编码。在 package.json 中，添加以下 `engines` 属性，将 Node.js 设置为所需的版本。

    ```
    "engines": {
        "node": "6.9.1"
    },
    ```

5. 最后，初始化 Git 存储库并提交文件。在应用程序根目录（package.json 所在目录）中，运行以下 Git 命令：

    ```
    git init
    git add .
    git commit -m "<your commit message>"
    ```

代码已可用于部署。

## 步骤 2：创建 Azure 应用并部署 Sails.js

接下来，在 Azure 中创建应用服务资源并向其部署 Sails.js 应用。

1. 如下所示登录 Azure：

    ```
    az login
    ```

    根据提示，在浏览器中继续使用具有 Azure 订阅的 Microsoft 帐户登录。

3. 设置应用服务的部署用户。稍后会使用这些凭据部署代码。

    ```
    az appservice web deployment user set --user-name <username> --password <password>
    ```

3. 创建具有名称的[资源组](../azure-resource-manager/resource-group-overview.md)。对于本 PHP 教程，不需要实际知道它是什么。

    ```
    az group create --location "<location>" --name my-sailsjs-app-group
    ```

    若要查看可用于 `<location>` 的可能值，请使用 `az appservice list-locations` CLI 命令。

3. 创建具有名称的“免费”[应用服务计划](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md)。对于本 PHP 教程，只需知道不用为此计划中的 Web 应用付费即可。

    ```
    az appservice plan create --name my-sailsjs-appservice-plan --resource-group my-sailsjs-app-group --sku FREE
    ```

4. 使用 `<app_name>` 中的唯一名称创建新的 Web 应用。

    ```
    az appservice web create --name <app_name> --resource-group my-sailsjs-app-group --plan my-sailsjs-appservice-plan
    ```

## 步骤 3：设置并部署 Sails.js 应用

1. 使用以下命令配置新 Web 应用的本地 Git 部署：

    ```
    az appservice web source-control config-local-git --name <app_name> --resource-group my-sailsjs-app-group
    ```

    会获得类似如下的 JSON 输出，这意味着已配置远程 Git 存储库：

    ```
    {
    "url": "https://<deployment_user>@<app_name>.scm.chinacloudsites.cn/<app_name>.git"
    }
    ```

6. 将 JSON 中的 URL 作为本地存储库的 Git remote 添加（为简单起见，调用 `azure`）。

    ```
    git remote add azure https://<deployment_user>@<app_name>.scm.chinacloudsites.cn/<app_name>.git
    ```

7. 向 `azure` Git remote 部署示例代码。出现提示时，使用前面配置的部署凭据。

    ```
    git push azure master
    ```

7. 最后，只需在浏览器中启动实时 Azure 应用：

    ```
    az appservice web browse --name <app_name> --resource-group my-sailsjs-app-group
    ```

    现在，你应会看到相同的 Sails.js 主页。

    ![](./media/app-service-web-nodejs-sails/sails-in-azure.png)

## 排查部署问题
如果 Sails.js 应用在应用服务中由于某种原因而失败，请查找 stderr 日志，以帮助进行故障排除。有关详细信息，请参阅[从 iisnode 获取 stdout 和 stderr 日志](./app-service-web-nodejs-get-started.md#get-stdout-and-stderr-logs-from-iisnode)。如果应用成功启动，stdout 日志应显示熟悉的消息：

```
               .-..-.

   Sails              <|    .-..-.
   v0.12.11            |\
                      /|.\
                     / || \
                   ,'  |'  \
                .-'.-==|/_--'
                `--'-------' 
   __---___--___---___--___---___--___
 ____---___--___---___--___---___--___-__

Server lifted in `D:\home\site\wwwroot`
To see your app, visit http://localhost:\\.\pipe\c775303c-0ebc-4854-8ddd-2e280aabccac
To shut down Sails, press <CTRL> + C at any time.
```

可以控制 [config/log.js](http://sailsjs.org/#!/documentation/concepts/Logging) 文件中 stdout 日志的粒度。

## 连接到 Azure 中的数据库
若要连接到 Azure 中的数据库，可以在 Azure 中创建所选的数据库，例如 Azure SQL 数据库、MySQL、MongoDB、Azure (Redis) 缓存等，并使用相应的[数据存储适配器](https://github.com/balderdashy/sails#compatibility)连接到该数据库。本部分中的步骤说明如何使用 [Azure DocumentDB](../documentdb/documentdb-protocol-mongodb.md) 数据库（支持 MongoDB 客户端连接）连接到 MongoDB。

1. [创建具有 MongoDB 协议支持的 DocumentDB 帐户](../documentdb/documentdb-create-mongodb-account.md)。
2. [创建 DocumentDB 集合和数据库](../documentdb/documentdb-create-collection.md)。集合的名称不重要，但从 Sails.js 连接时需要数据库的名称。
3. [查找 DocumentDB 数据库的连接信息](../documentdb/documentdb-connect-mongodb-account.md#a-idgetcustomconnectiona-get-the-mongodb-connection-string-to-customize)。
2. 从命令行终端安装 MongoDB 适配器：

    ```
    npm install sails-mongo --save
    ```

3. 打开 config/connections.js 并将以下连接对象添加到列表：

    ```
    docDbMongo: {
        adapter: 'sails-mongo',
        user: process.env.dbuser,
        password: process.env.dbpassword,
        host: process.env.dbhost,
        port: process.env.dbport,
        database: process.env.dbname,
        ssl: true
    },
    ```

    > [!NOTE] 
    `ssl: true` 选项很重要，因为 [Azure DocumentDB 需要它](../documentdb/documentdb-connect-mongodb-account.md#connection-string-requirements)。
    >
    >

4. 需要在应用服务中设置每个环境变量 (`process.env.*`)。为此，请从终端运行以下命令。使用 DocumentDB 数据库的连接信息。

    ```
    az appservice web config appsettings update --settings dbuser="<database user>" --name <app_name> --resource-group my-sailsjs-app-group
    az appservice web config appsettings update --settings dbpassword="<database password>" --name <app_name> --resource-group my-sailsjs-app-group
    az appservice web config appsettings update --settings dbhost="<database hostname>" --name <app_name> --resource-group my-sailsjs-app-group
    az appservice web config appsettings update --settings dbport="<database port>" --name <app_name> --resource-group my-sailsjs-app-group
    az appservice web config appsettings update --settings dbname="<database name>" --name <app_name> --resource-group my-sailsjs-app-group
    ```

    将设置放在 Azure 应用设置中，可以使敏感数据不受源控件 (Git) 的控制。接下来，配置开发环境以便使用相同的连接信息。
5. 打开 config/local.js，并添加以下连接对象：

    ```
    connections: {
        docDbMongo: {
            user: "<database user>",
            password: "<database password>",
            host: "<database hostname>",
            database: "<database name>",
            ssl: true
        },
    },
    ```

    此配置会覆盖 config/connections.js 文件中的本地环境设置。项目中默认的 .gitignore 排除了此文件，因此该文件不会存储在 Git 中。现在，可以从 Azure Web 应用和本地开发环境中连接到 DocumentDB (MongoDB) 数据库。
6. 打开 config/env/production.js 来配置生产环境，并添加以下 `models` 对象：

    ```
    models: {
        connection: 'docDbMongo',
        migrate: 'safe'
    },
    ```
7. 打开 config/env/development.js 来配置开发环境，并添加以下 `models` 对象：

    ```
    models: {
        connection: 'docDbMongo',
        migrate: 'alter'
    },
    ```

    借助 `migrate: 'alter'`，可使用数据库迁移功能轻松创建和更新数据库集合或表。但是，在 Azure（生产）环境中使用 `migrate: 'safe'`，因为 Sails.js 不允许在生产环境中使用 `migrate: 'alter'`（请参阅 [Sails.js 文档](http://sailsjs.org/documentation/concepts/models-and-orm/model-settings)）。
8. 和往常一样在终端[生成](http://sailsjs.org/documentation/reference/command-line-interface/sails-generate) Sails.js 的[蓝图 API](http://sailsjs.org/documentation/concepts/blueprints)，然后运行 `sails lift` 以使用 Sails.js 数据库迁移功能创建数据库。例如：

    ```
     sails generate api mywidget
     sails lift
    ```

    由此命令生成的 `mywidget` 模型为空，但可以使用它来说明具有数据库连接。运行 `sails lift` 时，可为应用使用的模型创建缺失的集合和表。
9. 访问刚刚在浏览器中创建的蓝图 API。例如：

    ```
    http://localhost:1337/mywidget/create
    ```

    API 应在浏览器窗口中返回创建的条目，这表示已成功创建集合。

    ```
    {"id":1,"createdAt":"2016-09-23T13:32:00.000Z","updatedAt":"2016-09-23T13:32:00.000Z"}
    ```
10. 现在将更改推送到 Azure，并浏览到应用以确保应用仍能正常工作。

    ```
     git add .
     git commit -m "<your commit message>"
     git push azure master
     az appservice web browse --name <app_name> --resource-group my-sailsjs-app-group
    ```

11. 访问 Azure Web 应用的蓝图 API。例如：

    ```
     http://<appname>.chinacloudsites.cn/mywidget/create
    ```

     如果 API 返回另一个新条目，那么 Azure Web 应用正在和 DocumentDB (MongoDB) 数据库通信。

## 更多资源
* [在 Azure App Service 中 Node.js Web 应用入门](./app-service-web-nodejs-get-started.md)
* [将 Node.js 模块与 Azure 应用程序一起使用](../nodejs-use-node-modules-azure-apps.md)

<!---HONumber=Mooncake_1226_2016-->