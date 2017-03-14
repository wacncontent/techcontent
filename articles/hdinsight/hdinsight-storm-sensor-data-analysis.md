---
title: 使用 Apache Storm 和 HBase 分析传感器数据 | Azure
description: 了解如何使用虚拟网络连接到 Apache Storm。了解如何使用 Storm 和 HBase 处理来自 Azure 事件中心的传感器数据，然后使用 D3.js 来可视化这些数据。
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun

ms.assetid: a9a1ac8e-5708-4833-b965-e453815e671f
ms.service: hdinsight
ms.devlang: java
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/12/2017
wacn.date: 03/10/2017
ms.author: larryfr
---

# 使用 Apache Storm、事件中心和 HDInsight 中的 HBase (Hadoop) 分析传感器数据
了解如何使用 Apache Storm on HDInsight 处理来自 Azure 事件中心的传感器数据，将这些数据存储到 Apache HBase on HDInsight 中，然后使用作为 Azure Web 应用运行的 D3.js 来可视化这些数据。

在本文档中使用的 Azure Resource Manager 模板演示了如何在资源组中创建多个 Azure 资源。具体说来，它将创建一个 Azure 虚拟网络、两个 HDInsight 群集（Storm 和 HBase）以及一个 Azure Web 应用。node.js 所实现的实时 Web 仪表板将自动部署到 Web 应用。

[!INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [!NOTE]
本文档中的信息以及提供的示例已使用基于 Linux 的 HDInsight 3.3 和 3.4 群集版本测试过。
><p>
> Linux 是在 HDInsight 3.4 或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](./hdinsight-component-versioning.md#hdi-version-32-and-33-nearing-deprecation-date)。

## 先决条件
* Azure 订阅。请参阅[获取 Azure 试用版](https://www.azure.cn/pricing/1rmb-trial/)。

    > [!IMPORTANT]
    不需要现有的 HDInsight 群集；本文档中的步骤将创建以下资源：
    > <p> 
    ><p>* Azure 虚拟网络
    <p> * Storm on HDInsight 群集（基于 Linux 的 2 个工作节点）
    <p> * HBase on HDInsight 群集（基于 Linux 的 2 个工作节点）
    <p> *托管 Web 仪表板的 Azure Web 应用
    > 
    > 
* [Node.js](http://nodejs.org/)：用于在开发环境中以本地方式预览 Web 仪表板。
* [Java 和 JDK 1.7](http://www.oracle.com/technetwork/java/javase/downloads/index.html)：用于开发 Storm 拓扑。
* [Maven](http://maven.apache.org/what-is-maven.html)：用于生成和编译项目。
* [Git](http://git-scm.com/)：用于从 GitHub 下载项目。
* **SSH** 客户端：用于连接到基于 Linux 的 HDInsight 群集。有关如何将 SSH 与 HDInsight 配合使用的详细信息，请参阅以下文档。

    * [Use SSH with HDInsight from Windows clients（在 Windows 客户端中将 SSH 与 HDInsight 配合使用）](./hdinsight-hadoop-linux-use-ssh-windows.md)
    * [在 Linux、Unix 或 Mac 客户端中将 SSH 与 HDInsight 配合使用](./hdinsight-hadoop-linux-use-ssh-unix.md)

        > [!NOTE]
        用户还必须有权访问 `scp` 命令，该命令用于通过 SSH 在本地开发环境和 HDInsight 群集之间复制文件。
        > 
        > 

## 体系结构
![体系结构示意图](./media/hdinsight-storm-sensor-data-analysis/devicesarchitecture.png)  

本示例包括以下组成部分：

* **Azure 事件中心**：包含从传感器收集的数据。对于本示例，将提供用于生成数据的应用程序。
* **Storm on HDInsight**：用于实时处理来自事件中心的数据。
* **HBase on HDInsight**：由 Storm 处理数据后为数据提供持久性 NoSQL 数据存储。
* **Azure 虚拟网络服务**：在 Storm on HDInsight 和 HBase on HDInsight 群集之间启用安全通信。

    > [!NOTE]
    若要使用 Java HBase 客户端 API，必须使用虚拟网络，因为该 API 不是通过 HBase 群集的公共网关公开的。将 HBase 和 Storm 群集安装到同一虚拟网络以后，Storm 群集（或虚拟网络上的任何其他系统）即可使用客户端 API 直接访问 HBase。
    > 
    > 
* **仪表板网站**：实时绘制数据图表的示例仪表板。

    * 该网站在 Node.js 中实现，因此它可以在用于测试的任何客户端操作系统上运行，或者可以部署到 Azure 网站。
    * [Socket.io](http://socket.io/) 用于 Storm 拓扑和网站之间的实时通信。

        > [!NOTE]
        这是实现详细信息。你可以使用任何通信框架，例如原始 WebSockets 或 SignalR。
        > 
        > 
    * [D3.js](http://d3js.org/) 用于绘制发送到网站的数据的图表。

> [!IMPORTANT]
需要两个群集，因为没有方法可以创建一个同时适用于 Storm 和 HBase 的 HDInsight 群集。
> 
> 

拓扑使用 [org.apache.storm.eventhubs.spout.EventHubSpout](http://storm.apache.org/releases/0.10.1/javadocs/org/apache/storm/eventhubs/spout/class-use/EventHubSpout.html) 类从事件中心读取数据，使用 [org.apache.storm.hbase.bolt.HBaseBolt](https://storm.apache.org/javadoc/apidocs/org/apache/storm/hbase/bolt/class-use/HBaseBolt.html) 类将数据写入 HBase 中。与网站的通信可通过使用 [socket.io client.java](https://github.com/nkzawa/socket.io-client.java) 来实现。

下面是拓扑图。

![拓扑图示意图](./media/hdinsight-storm-sensor-data-analysis/sensoranalysis.png)  

> [!NOTE]
这是一个非常简单的拓扑视图。在运行时，每个组件的实例为每个分区创建事件中心所读取。这些实例分布在群集中，节点和数据在它们之间路由，如下所示：
><p> 
><p> * 从 spout 到分析器的数据已经过负载均衡。
<p> * 从分析器到仪表板和 HBase 的数据已按设备 ID 分组，因此来自同一设备的消息始终流向同一组件。
> 
> 

### 拓扑组件
* **EventHub Spout**：此 Spout 作为 Apache Storm 0.10.0 及更高版本的一部分提供。

    > [!NOTE]
    此示例中使用的事件中心 Spout 需要 Storm on HDInsight 群集版本 3.3 或 3.4。有关如何将事件中心与旧版 Storm on HDInsight 配合使用的信息，请参阅[使用 Storm on HDInsight 从 Azure 事件中心处理事件](./hdinsight-storm-develop-java-event-hub-topology.md)。
    > 
    > 
* **ParserBolt.java**：spout 发出的数据是原始的 JSON，有时每次会发出多个事件。此 bolt 演示如何读取 spout 发出的数据，并将它作为包含多个字段的元组形式发送到新流。
* **DashboardBolt.java**：演示如何使用 Java 的 Socket.io 客户端库将数据实时发送到 Web 仪表板。

此示例使用 [Flux](https://storm.apache.org/releases/0.10.0/flux.html) 框架，因此 YAML 文件中包含拓扑定义。有两个文件：

* **no-hbase.yaml** - 在开发环境中测试拓扑时使用此文件。它不使用 HBase 组件，因为无法从群集所在的虚拟网络外部访问 HBase Java API。
* **with-hbase.yaml** - 将拓扑部署到 Storm 群集时使用此文件。它使用 HBase 组件，因为它在与 HBase 群集相同的虚拟网络中运行。

## 准备环境
在使用本示例之前，必须创建由 Storm 拓扑读取的 Azure 事件中心。

### 配置事件中心
事件中心是此示例的数据源。按照下列步骤创建一个新的事件中心。

1. 在 [Azure 门户预览](https://portal.azure.cn)中，选择“+ 新建”->“物联网”->“事件中心”。
2. 在“创建命名空间”边栏选项卡上，执行以下任务：

    1. 输入该命名空间的“名称”。
    2. 选择定价层。“基本”对于本示例来说已足够。
    3. 选择要使用的 Azure“订阅”。
    4. 选择现有的资源组或创建新资源组。
    5. 选择事件中心的“位置”。
    6. 选择“固定到仪表板”，然后单击“创建”。
3. 创建过程完成后，将显示命名空间的“事件中心”边栏选项卡。在此处选择“+ 添加事件中心”。在“创建事件中心”边栏选项卡上，输入名称“sensordata”，然后选择“创建”。将其他字段保留默认值。
4. 在命名空间的“事件中心”边栏选项卡中，选择“事件中心”。选择“sensordata”条目。
5. 在 sensordata 事件中心的边栏选项卡中，选择“共享访问策略”。使用“+ 添加”链接添加以下策略：

    | 策略名称 | 声明 |
    | ----- | ----- |
    | devices | 发送 |
    | storm | 侦听 |

1. 选择这两个策略，记下“PRIMARY KEY”值。在将来的步骤中需要这两个策略的值。

## 下载并配置项目
使用以下命令从 GitHub 中下载项目。

```
git clone https://github.com/Blackmist/hdinsight-eventhub-example
```

命令执行完毕后，你将得到以下目录结构：

```
hdinsight-eventhub-example/
    TemperatureMonitor/ - this contains the topology
        resources/
            log4j2.xml - set logging to minimal
            no-hbase.yaml - topology definition for local testing
            with-hbase.yaml - topology definition that uses HBase in a virutal network
        src/ - the Java bolts
        dev.properties - contains configuration values for your environment
    dashboard/nodejs/ - this is the node.js web dashboard
    SendEvents/ - utilities to send fake sensor data
```

> [!NOTE]
本文档不会深入介绍本示例中包含的代码；但是，代码带有全面的注释。
> 
> 

打开 **hdinsight-eventhub-example/TemperatureMonitor/dev.properties** 文件，将事件中心信息添加到以下行：

```
eventhub.read.policy.name: storm
eventhub.read.policy.key: KeyForTheStormPolicy
eventhub.namespace: YourNamespace
eventhub.name: sensordata
```

> [!NOTE]
此示例假定使用 **storm** 作为具有 **Listen** 声明的策略的名称，且事件中心的名称为 **sensordata**。
> 
> 

在添加此信息后，请保存该文件。

## 编译并在本地测试
测试之前，必须启动仪表板以查看拓扑的输出，并生成要在事件中心中存储的数据。

> [!IMPORTANT]
在本地进行测试时，此拓扑的 HBase 组件不处于活动状态，因为 HBase 群集的 Java API 无法从包含群集的 Azure 虚拟网络外部访问。
> 
> 

### 启动 Web 应用程序
1. 打开新的命令提示符或终端，并将目录更改为 将目录更改为 **hdinsight-eventhub-example/dashboard**，并使用以下命令安装 Web 应用程序所需的依赖项：

    ```
    npm install
    ```
2. 使用以下命令启动 Web 应用程序：

    ```
    node server.js
    ```

    你应看到类似于下面的消息：

    ```
    Server listening at port 3000
    ```
3. 打开 Web 浏览器，并输入 http://localhost:3000/** 作为地址。你应看到类似于下面的页面：

    ![Web 仪表板](./media/hdinsight-storm-sensor-data-analysis/emptydashboard.png)  

    将此命令提示符或终端保持打开状态。测试完成后，使用 Ctrl-C 停止 Web 服务器。

### 开始生成数据
> [!NOTE]
此部分中的步骤使用 Node.js，以便它们可以在任何平台上使用。对于其他语言示例，请参阅 **SendEvents** 目录。
> 
> 

1. 打开新的命令提示符、shell 或终端，将目录更改为 **hdinsight-eventhub-example/SendEvents/nodejs**，然后使用以下命令安装应用程序所需的依赖项：

    ```
    npm install
    ```
2. 在文本编辑器中打开 **app.js** 文件，并添加你之前获取的事件中心信息：

    ```
    // ServiceBus Namespace
    var namespace = 'YourNamespace';
    // Event Hub Name
    var hubname ='sensordata';
    // Shared access Policy name and key (from Event Hub configuration)
    var my_key_name = 'devices';
    var my_key = 'YourKey';
    ```

    > [!NOTE]
    此示例假定已使用 **sensordata** 作为事件中心的名称并已使用**devices** 作为具有 **Send** 声明的策略的名称。

3. 使用以下命令在事件中心插入新条目：

    ```
    node app.js
    ```

    你应会看到包含发送到事件中心的数据的多个输出行。这些信息如下所示：

    ```
    {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"0","Temperature":7}
    {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"1","Temperature":39}
    {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"2","Temperature":86}
    {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"3","Temperature":29}
    {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"4","Temperature":30}
    {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"5","Temperature":5}
    {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"6","Temperature":24}
    {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"7","Temperature":40}
    {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"8","Temperature":43}
    {"TimeStamp":"2015-02-10T14:43.05.00320Z","DeviceId":"9","Temperature":84}
    ```

### 启动拓扑
1. 打开新的命令提示符、shell 或终端，将目录更改为 **hdinsight-eventhub-example/TemperatureMonitor**，然后使用以下命令启动拓扑：

    ```
    mvn compile exec:java -Dexec.args="--local -R /no-hbase.yaml --filter dev.properties"
    ```

    如果使用的是 PowerShell，请使用以下命令：

    ```
    mvn compile exec:java "-Dexec.args=--local -R /no-hbase.yaml --filter dev.properties"
    ```

    > [!NOTE]
    如果在 Linux/Unix/OS X 系统上，并且[已在开发环境中安装 Storm](http://storm.apache.org/releases/0.10.0/Setting-up-development-environment.html)，则可以使用以下命令：
    ><p> 
    ><p> `mvn compile package` <p> `storm jar target/WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local -R /no-hbase.yaml`
    > 
    > 

    这会在本地模式下启动 **no-hbase.yaml** 文件中定义的拓扑。**dev.properties** 文件中包含的值提供事件中心的连接信息。启动后，拓扑会从事件中心读取条目，然后将它们发送到在本地计算机上运行的仪表板。你应看到各行显示在 Web 仪表板中，如下所示：

    ![包含数据的仪表板](./media/hdinsight-storm-sensor-data-analysis/datadashboard.png)  

2. 当仪表板正在运行时，使用前面步骤中的 `node app.js` 命令将新数据发送到事件中心。由于温度值是随机生成的，因此图表应进行更新，以显示温度的较大变化。

    > [!NOTE]
    使用 `node app.js` 命令时，必须位于 **hdinsight-eventhub-example/SendEvents/Nodejs** 目录中。
    > 
    > 
3. 验证其运行情况后，使用 Ctrl+C 停止拓扑。也可以使用 Ctrl+C 停止本地 Web 服务器。

## 创建 Storm 和 HBase 群集
为了在 HDInsight 上运行拓扑并启用 HBase Bolt，必须创建新的 Storm 群集和 HBase 群集。此部分的步骤使用 [Azure Resource Manager 模板](../azure-resource-manager/resource-group-template-deploy.md)创建新的 Azure 虚拟网络以及该虚拟网络上的 Storm 和 HBase 群集。该模板还创建 Azure Web 应用并将仪表板的副本部署到其中。

> [!NOTE]
使用虚拟网络，以便运行在 Storm 群集上的拓扑能够直接使用 HBase Java API 与 HBase 群集通信。
> 
> 

本文档所使用的 Resource Manager 模板位于 **https://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-hbase-storm-cluster-in-vnet.json** 的公共 Blob 容器中。

1. 单击以下按钮登录到 Azure，然后在 Azure 门户预览中打开 Resource Manager 模板。

    <a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Farmtemplates%2Fcreate-linux-based-hbase-storm-cluster-in-vnet.json" target="_blank"><img src="./media/hdinsight-storm-sensor-data-analysis/deploy-to-azure.png" alt="Deploy to Azure"></a>

    >[!NOTE]
    > 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，将一些终结点 -“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”；将允许的位置更改为“中国北部”和“中国东部”；将 HDInsight Linux 版本更改为 Azure 中国区支持的版本 3.5。

2. 在“参数”边栏选项卡中，输入以下内容：

    ![HDInsight 参数](./media/hdinsight-storm-sensor-data-analysis/parameters.png)  

    * **BASECLUSTERNAME**：此值将用作 Storm 和 HBase 群集的基名称。例如，输入 **hdi** 会创建名为 **storm-hdi** 的 Storm 群集，以及名为 **hbase-hdi** 的 HBase 群集。
    * **CLUSTERLOGINUSERNAME**：Storm 和 HBase 群集的管理员用户名。
    * **CLUSTERLOGINPASSWORD**：Storm 和 HBase 群集的管理员用户名。
    * **SSHUSERNAME**：要为 Storm 和 HBase 群集创建的 SSH 用户。
    * **SSHPASSWORD**：Storm 和 HBase 群集的 SSH 用户的密码。
    * **LOCATION**：要在其中创建群集的区域。

        单击**确定**以保存参数。
3. 使用“资源组”部分创建新资源组或选择现有的资源组。
4. 在“资源组位置”下拉菜单中选择一个位置，该位置与为 **LOCATION** 参数选择的位置相同。
5. 选择“法律条款”，然后选择“创建”。
6. 最后，选中“固定到仪表板”并选择“创建”。创建群集大约会需要 20 分钟时间。

创建资源后，将重定向到包含群集和 Web 仪表板的资源组的边栏选项卡。

![VNet 和群集的资源组边栏选项卡](./media/hdinsight-storm-sensor-data-analysis/groupblade.png)  

> [!IMPORTANT]
请注意，HDInsight 群集的名称为 **storm-BASENAME** 和 **hbase-BASENAME**，其中，BASENAME 是为模板提供的名称。在后续步骤中连接到群集时，将用到这些名称。另请注意，仪表板站点的名称是 **basename-dashboard**。以后在查看仪表板时会用到该名称。
> 
> 

## 配置仪表板 Bolt
若要将数据发送到部署为 Web 应用的仪表板，必须修改 **dev.properties** 文件中的以下行：

```
dashboard.uri: http://localhost:3000
```

将 `http://localhost:3000` 更改为 `http://BASENAME-dashboard.chinacloudsites.cn`，然后保存文件。将 **BASENAME** 替换为在上一步提供的基名称。还可以通过以前创建的资源组选择仪表板并查看 URL。

## 创建 HBase 表
若要将数据存储在 HBase 中，必须先创建表。通常需预先创建供 Storm 将内容写入到其中的资源，因为如果尝试从 Storm 拓扑内部创建资源，则代码的多个分布式副本会尝试创建同一资源。在拓扑外部创建资源，只使用 Storm 进行读取/写入和分析。

1. 使用你在创建群集期间提供给模板的 SSH 用户和密码，通过 SSH 连接到 HBase 群集。例如，如果使用 `ssh` 命令进行连接，将使用以下语法：

    ```
    ssh USERNAME@hbase-BASENAME-ssh.azurehdinsight.cn
    ```

    在该命令中，将 **USERNAME** 替换为创建群集时提供的 SSH 用户名，将 **BASENAME** 替换为所提供的基名称。出现提示时，请输入 SSH 用户的密码。
2. 从 SSH 会话中启动 HBase Shell。

    ```
    hbase shell
    ```

    在 Shell 加载后，将会显示 `hbase(main):001:0>` 提示符。
3. 从 HBase Shell 中，输入以下命令以创建存储传感器数据的表。

    ```
    create 'SensorData', 'cf'
    ```
4. 使用以下命令验证是否已创建该表：

    ```
    scan 'SensorData'
    ```

    此时会返回类似于以下示例的信息，指示表中有 0 行。

    ```
    ROW                   COLUMN+CELL                                       0 row(s) in 0.1900 seconds
    ```
5. 输入以下命令，退出 HBase Shell：

    ```
    exit
    ```

## 配置 HBase Bolt

若要将内容从 Storm 群集写入 HBase，必须为 HBase Bolt 提供 HBase 群集的配置详细信息。若要执行此操作，最简单的方法是从群集下载 **hbase-site.xml** 并将其包括在项目中。

> [!IMPORTANT]
还必须下载在 Storm on HDInsight 3.3 或 3.4 群集上提供的 storm-hbase.jar 文件；此版本经编译后兼容 HBase 1.1.x，后者可用于 HBase on HDInsight 3.3 和 3.4 群集。如果从其他位置使用 storm-hbase 组件，则可针对旧版 HBase 对其进行编译。

### 下载 hbase-site.xml
在命令提示符处，使用 SCP 从群集下载 **hbase-site.xml** 文件。在以下示例中，将 **USERNAME** 替换为创建群集时提供的 SSH 用户，将 **BASENAME** 替换为此前提供的基名称。出现提示时，请输入 SSH 用户的密码。将 `/path/to/TemperatureMonitor/resources/hbase-site.xml` 替换为此文件在 TemperatureMonitor 项目中的路径。

```
scp USERNAME@hbase-BASENAME-ssh.azurehdinsight.cn:/etc/hbase/conf/hbase-site.xml /path/to/TemperatureMonitor/resources/hbase-site.xml
```

此时会将 **hbase-site.xml** 下载到指定路径。

### 下载并安装 storm-hbase 组件
1. 在命令提示符处，使用 SCP 从 Storm 群集下载 **storm-hbase.jar** 文件。在以下示例中，将 **USERNAME** 替换为创建群集时提供的 SSH 用户，将 **BASENAME** 替换为此前提供的基名称。出现提示时，请输入 SSH 用户的密码。

    ```
    scp USERNAME@storm-BASENAME-ssh.azurehdinsight.cn:/usr/hdp/current/storm-client/contrib/storm-hbase/storm-hbase*.jar .
    ```

    此时会下载名为 `storm-hbase-####.jar` 的文件，其中 #### 是此群集的 Storm 的版本号。记下该版本号，因为此后会用到它。
2. 使用以下命令，将该组件安装到开发环境中的本地 Maven 存储库。这样 Maven 就能在编译项目时找到包。将 **####** 替换为文件名中包含的版本号。

    ```
    mvn install:install-file -Dfile=storm-hbase-####.jar -DgroupId=org.apache.storm -DartifactId=storm-hbase -Dversion=#### -Dpackaging=jar
    ```

    如果使用的是 PowerShell，请使用以下命令：

    ```
    mvn install:install-file "-Dfile=storm-hbase-####.jar" "-DgroupId=org.apache.storm" "-DartifactId=storm-hbase" "-Dversion=####" "-Dpackaging=jar"
    ```

3. 在 __pom.xml__ 文件中，查找 __storm-hbase__ 的依赖项节。通过删除围绕依赖项的 `<!--` 和 `-->`，取消注释依赖项。此外，更改 `<version></version>` 条目以匹配先前步骤中使用的 ####。该条目将如以下示例中所示：

    ```
    <dependency>
        <groupId>org.apache.storm</groupId>
        <artifactId>storm-hbase</artifactId>
        <version>0.10.0.2.4.2.4-5</version>
    </dependency>
    ```

    进行更改之后，保存该文件。

## 生成解决方案，然后将其打包并部署到 HDInsight

在开发环境中，按以下步骤将 Storm 拓扑部署到 Storm 群集。

1. 在 **TemperatureMonitor** 目录中，使用以下命令在项目中执行新的生成操作并创建 JAR 包：

    ```
    mvn clean compile package
    ```

    此操作将在项目的 **target** 目录中创建一个名为 **TemperatureMonitor-1.0-SNAPSHOT.jar** 的文件。
2. 使用 scp 将 **TemperatureMonitor-1.0-SNAPSHOT.jar** 文件上载到 Storm 群集。在以下示例中，将 **USERNAME** 替换为创建群集时提供的 SSH 用户，将 **BASENAME** 替换为此前提供的基名称。出现提示时，请输入 SSH 用户的密码。

    ```
    scp target/TemperatureMonitor-1.0-SNAPSHOT.jar USERNAME@storm-BASENAME-ssh.azurehdinsight.cn:TemperatureMonitor-1.0-SNAPSHOT.jar
    ```

    > [!NOTE]
    上载文件可能需要几分钟的时间，因为文件大小有数 MB。
    > 
    > 

    使用 scp 上载 **dev.properties** 文件，因为其中包含用于连接到事件中心和仪表板的信息。

    ```
    scp dev.properties USERNAME@storm-BASENAME-ssh.azurehdinsight.cn:dev.properties
    ```
3. 上载文件以后，使用 SSH 连接到群集。

    ```
    ssh USERNAME@storm-BASENAME-ssh.azurehdinsight.cn
    ```
4. 在 SSH 会话中，使用以下命令启动拓扑。

    ```
    storm jar TemperatureMonitor-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /with-hbase.yaml --filter dev.properties
    ```

    这将使用 **with-hbase.yaml** 文件中的拓扑定义以及 **dev.properties** 文件中的配置值启动拓扑。
5. 启动拓扑后，打开浏览器到 Azure 发布的网站，然后使用 `node app.js` 命令将数据发送到事件中心。你应该看到 Web 仪表板更新以显示信息。

    ![仪表板](./media/hdinsight-storm-sensor-data-analysis/datadashboard.png)  

## 查看 HBase 数据
通过 `node app.js` 将数据提交到拓扑以后，请执行以下步骤，以便连接到 HBase 并确保数据已写入此前创建的表中。

1. 使用 SSH 连接到 HBase 群集。

    ```
    ssh USERNAME@hbase-BASENAME-ssh.azurehdinsight.cn
    ```
2. 从 SSH 会话中启动 HBase Shell。

    ```
    hbase shell
    ```

    在 Shell 加载后，将会显示 `hbase(main):001:0>` 提示符。
3. 查看表中的行：

    ```
    scan 'SensorData'
    ```

    此时会返回类似于以下内容的信息，指示表中有 0 行。

    ```
    hbase(main):002:0> scan 'SensorData'
    ROW                             COLUMN+CELL
    \x00\x00\x00\x00               column=cf:temperature, timestamp=1467290788277, value=\x00\x00\x00\x04
    \x00\x00\x00\x00               column=cf:timestamp, timestamp=1467290788277, value=2015-02-10T14:43.05.00320Z
    \x00\x00\x00\x01               column=cf:temperature, timestamp=1467290788348, value=\x00\x00\x00M
    \x00\x00\x00\x01               column=cf:timestamp, timestamp=1467290788348, value=2015-02-10T14:43.05.00320Z
    \x00\x00\x00\x02               column=cf:temperature, timestamp=1467290788268, value=\x00\x00\x00R
    \x00\x00\x00\x02               column=cf:timestamp, timestamp=1467290788268, value=2015-02-10T14:43.05.00320Z
    \x00\x00\x00\x03               column=cf:temperature, timestamp=1467290788269, value=\x00\x00\x00#
    \x00\x00\x00\x03               column=cf:timestamp, timestamp=1467290788269, value=2015-02-10T14:43.05.00320Z
    \x00\x00\x00\x04               column=cf:temperature, timestamp=1467290788356, value=\x00\x00\x00>
    \x00\x00\x00\x04               column=cf:timestamp, timestamp=1467290788356, value=2015-02-10T14:43.05.00320Z
    \x00\x00\x00\x05               column=cf:temperature, timestamp=1467290788326, value=\x00\x00\x00\x0D
    \x00\x00\x00\x05               column=cf:timestamp, timestamp=1467290788326, value=2015-02-10T14:43.05.00320Z
    \x00\x00\x00\x06               column=cf:temperature, timestamp=1467290788253, value=\x00\x00\x009
    \x00\x00\x00\x06               column=cf:timestamp, timestamp=1467290788253, value=2015-02-10T14:43.05.00320Z
    \x00\x00\x00\x07               column=cf:temperature, timestamp=1467290788229, value=\x00\x00\x00\x12
    \x00\x00\x00\x07               column=cf:timestamp, timestamp=1467290788229, value=2015-02-10T14:43.05.00320Z
    \x00\x00\x00\x08               column=cf:temperature, timestamp=1467290788336, value=\x00\x00\x00\x16
    \x00\x00\x00\x08               column=cf:timestamp, timestamp=1467290788336, value=2015-02-10T14:43.05.00320Z
    \x00\x00\x00\x09               column=cf:temperature, timestamp=1467290788246, value=\x00\x00\x001
    \x00\x00\x00\x09               column=cf:timestamp, timestamp=1467290788246, value=2015-02-10T14:43.05.00320Z
    10 row(s) in 0.1800 seconds
    ```

    > [!NOTE]
    此扫描操作将仅返回表中的行，最多 10 行。
    > 
    > 

## 删除群集
[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

若要同时删除群集、存储和 Web 应用，请删除包含它们的资源组。

## 后续步骤
你现在已了解如何使用 Storm 从事件中心读取数据、如何将数据存储到 HBase 中，以及如何使用 Socket.io 和 D3.js 将信息显示在外部仪表板上。

* 有关 HDinsight Storm 拓扑的更多示例，请参阅：

    * [Storm on HDInsight 的示例拓扑](./hdinsight-storm-example-topology.md)
* 有关 Apache Storm 的详细信息，请参阅 [Apache Storm](https://storm.incubator.apache.org/) 站点。
* 有关 HBase on HDInsight 的详细信息，请参阅 [HDInsight 上的 HBase 概述](./hdinsight-hbase-overview.md)。
* 有关 Socket.io 的详细信息，请参阅 [socket.io](http://socket.io/) 站点。
* 有关 D3.js 的详细信息，请参阅 [D3.js - 数据驱动的文档](http://d3js.org/)。
* 有关以 Java 创建拓扑的信息，请参阅[为 Apache Storm on HDInsight 开发 Java 拓扑](./hdinsight-storm-develop-java-topology.md)。
* 有关以 .NET 创建拓扑的信息，请参阅[使用 Visual Studio 为 Apache Storm on HDInsight 开发 C# 拓扑](./hdinsight-storm-develop-csharp-visual-studio-topology.md)。

[azure-portal]: https://portal.azure.cn

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned-->