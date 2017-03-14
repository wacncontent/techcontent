---
title: 处理 Azure IoT 中心设备到云的消息 (Java) | Azure
description: 如何通过通过 IoT 中心从与事件中心兼容的终结点进行读取，处理 IoT 中心设备到云的消息。创建使用 EventProcessorHost 实例的 Java 服务应用。
services: iot-hub
documentationcenter: java
author: dominicbetts
manager: timlt
editor: ''

ms.assetid: bd9af5f9-a740-4780-a2a6-8c0e2752cf48
ms.service: iot-hub
ms.devlang: java
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/12/2016
wacn.date: 02/10/2017
ms.author: dobett
---

# 处理 IoT 中心设备到云的消息 (Java)

[!INCLUDE [iot-hub-selector-process-d2c](../../includes/iot-hub-selector-process-d2c.md)]

## 介绍
Azure IoT 中心是一项完全托管的服务，可在数百万台设备和单个解决方案后端之间实现安全可靠的双向通信。其他教程（[IoT 中心入门]和[使用 IoT 中心发送云到设备的消息][lnk-c2d]）介绍了如何使用 IoT 中心的设备到云和云到设备的基本消息传递功能。

本教程以 [IoT 中心入门]教程中演示的代码为基础，介绍了如何按可缩放的方式通过消息路由处理设备到云的消息。本教程描述了如何处理需要解决方案后端立即执行操作的消息。例如，设备可能将发送一条警报消息，触发在 CRM 系统中插入票证。与此相反，数据点消息仅送入分析引擎。例如，设备中存储便于日后分析的温度遥测是数据点消息。

在本教程最后，会运行 3 个 Java 控制台应用：

* **simulated-device**（[IoT 中心入门]教程中创建的应用的修改版本）每秒发送一次设备到云的数据点消息，每 10 秒发送一次设备到云的交互式消息。此应用使用 AMQP 协议实现与 IoT 中心的通信。
* **read-d2c-messages** 显示模拟设备应用发送的遥测数据。
* **read-critical-queue** 从附加到 IoT 中心的服务总线队列中取消关键消息的排队。

> [!NOTE]
IoT 中心对许多设备平台和语言（包括 C、Java 和 JavaScript）提供 SDK 支持。若要了解如何将本教程中的模拟设备替换为物理设备，以及如何将设备连接到 IoT 中心，请参阅 [Azure IoT 开发人员中心]。
> 
> 

要完成本教程，需要具备以下先决条件：

+ [IoT 中心入门]教程的完整工作版本。

+ Java SE 8。<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Java。

+ Maven 3。<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Maven。

+ 有效的 Azure 帐户。<br/>若尚无帐户，只需几分钟即可创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。

应具备 [Azure 存储]和 [Azure 服务总线]的一些基础知识。

## 从模拟设备应用发送交互式消息
在本部分中，将修改 [IoT 中心入门]教程中创建的模拟设备应用，不定期发送需要立即处理的消息。

1. 使用文本编辑器打开 simulated-device\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。本文件包含用于 [IoT 中心入门]教程中创建的 **simulated-device** 应用的代码。
2. 使用以下代码替换 **MessageSender** 类：

    ```
    private static class MessageSender implements Runnable {
        public volatile boolean stopThread = false;

        public void run()  {
            try {
                double avgWindSpeed = 10; // m/s
                Random rand = new Random();

                while (!stopThread) {
                    double currentWindSpeed = avgWindSpeed + rand.nextDouble() * 4 - 2;
                    TelemetryDataPoint telemetryDataPoint = new TelemetryDataPoint();
                    telemetryDataPoint.deviceId = deviceId;
                    telemetryDataPoint.windSpeed = currentWindSpeed;

                    String msgStr = telemetryDataPoint.serialize();
                    if (new Random() > 0.7) {
                        Message msg = new Message("This is a critical message.");
                        msg.setProperty("level", "critical");
                    } else {
                        Message msg = new Message(msgStr);
                    }

                    System.out.println("Sending: " + msgStr);

                    Object lockobj = new Object();
                    EventCallback callback = new EventCallback();
                    client.sendEventAsync(msg, callback, lockobj);

                    synchronized (lockobj) {
                        lockobj.wait();
                    }
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                System.out.println("Finished.");
            }
        }
    }
    ```

    这会将 `"level": "critical"` 属性随机添加到模拟设备发送的消息，该设备可模拟需要解决方案后端立即执行操作的消息。应用程序将在消息属性中传递此信息（而非在消息正文中），因此 IoT 中心可将消息路由到适当的消息目标。

    > [!NOTE]
    > 可使用消息属性根据各种方案路由消息，包括冷路径处理和此处所示的热路径示例。
    > 
    > 

2. 保存并关闭 simulated-device\\src\\main\\java\\com\\mycompany\\app\\App.java 文件。

    > [!NOTE]
    > 为简单起见，本教程不实现任何重试策略。在生产代码中，应按 MSDN 文章 [Transient Fault Handling]（暂时性故障处理）中所述实施指数退让等重试策略。
    > 
    > 

3. 若要使用 Maven 构建 **simulated-device** 应用，请在 simulated-device 文件夹的命令提示符处执行以下命令：

    ```
    mvn clean package -DskipTests
    ```

## 向 IoT 中心添加一个队列并向其路由消息
在本部分中，将创建一个服务总线队列并将其连接到 IoT 中心，还会配置 IoT 中心，根据消息上的现有属性发送消息到队列。若要深入了解如何处理来自服务总线队列的消息，请参阅[队列入门][Service Bus queue]教程。

1. 按[队列入门][Service Bus queue]中所述，创建服务总线队列。记下命名空间和队列名称。

2. 在 Azure 门户预览中，打开 IoT 中心并单击“终结点”。

    ![IoT 中心的终结点][30]  

3. 在“终结点”边栏选项卡中，单击顶部的“添加”，将队列添加到 IoT 中心。将终结点命名为“CriticalQueue”，并使用下拉列表选择“服务总线队列”、队列所在的服务总线命名空间和队列名称。完成后，单击底部的“保存”。

    ![添加终结点][31]  

4. 现在单击 IoT 中心的“路由”。单击边栏选项卡顶部的“添加”，创建将消息路由到刚添加的队列的规则。选择“DeviceTelemetry”作为数据源。输入 `level="critical"` 作为条件，然后选择刚添加为终结点的队列作为路由终结点。完成后，单击底部的“保存”。

    ![添加路由][32]  

    请确保回退路由设为“开”。这是 IoT 中心的默认配置。

    ![回退路由][33]  

## （可选）从队列终结点读取
可按照[队列入门][lnk-sb-queues-java]中的说明，选择性地从队列终结点读取消息。将应用命名为 **read-critical-queue**。

## 运行应用程序
现在即可运行 3 个应用程序。

1. 若要运行 **read-d2c-messages** 应用程序，请在命令提示符或外壳处导航到 read-d2c 文件夹并执行以下命令：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![运行 read-d2c-messages][readd2c]  

2. 若要运行 **read-critical-queue** 应用程序，请在命令提示符或外壳处导航到 read-critical-queue 文件夹并执行以下命令：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![运行 read-critical-messages][readqueue]  

3. 若要运行 **simulated-device** 应用，请在命令提示符或 shell 处导航到 simulated-device 文件夹并执行以下命令：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![运行 simulated-device][simulateddevice]  

## 后续步骤
在本教程中，介绍了如何使用 IoT 中心的消息路由功能可靠地分派设备到云的消息。

通过[如何使用 IoT 中心发送云到设备的消息][lnk-c2d]，了解如何将消息从解决方案后端发送到设备。

若要查看使用 IoT 中心完成端到端解决方案的示例，请参阅 [Azure IoT 套件][lnk-suite]。

若要深入了解如何使用 IoT 中心开发解决方案，请参阅 [IoT 中心开发人员指南]。

若要详细了解 IoT 中心的消息路由，请参阅[使用 IoT 中心发送和接收消息][lnk-devguide-messaging]。

<!-- Images. -->

<!-- TODO: UPDATE PICTURES -->
[simulateddevice]: ./media/iot-hub-java-java-process-d2c/runsimulateddevice.png
[readd2c]: ./media/iot-hub-java-java-process-d2c/runprocessinteractive.png
[readqueue]: ./media/iot-hub-java-java-process-d2c/runprocessd2c.png

[30]: ./media/iot-hub-java-java-process-d2c/click-endpoints.png
[31]: ./media/iot-hub-java-java-process-d2c/endpoint-creation.png
[32]: ./media/iot-hub-java-java-process-d2c/route-creation.png
[33]: ./media/iot-hub-java-java-process-d2c/fallback-route.png

<!-- Links -->

[Azure Blob 存储]: ../storage/storage-dotnet-how-to-use-blobs.md
[Azure 数据工厂]: ../data-factory/index.md
[HDInsight (Hadoop)]: ../hdinsight/index.md
[Service Bus queue]: ../service-bus-messaging/service-bus-java-how-to-use-queues.md
[lnk-sb-queues-java]: ../service-bus-messaging/service-bus-java-how-to-use-queues.md

[Azure IoT 中心开发人员指南 - 设备到云]: ./iot-hub-devguide-messaging.md

[Azure 存储]: ../storage/index.md
[Azure 服务总线]: ../service-bus/index.md

[IoT 中心开发人员指南]: ./iot-hub-devguide.md
[lnk-devguide-messaging]: ./iot-hub-devguide-messaging.md
[IoT 中心入门]: ./iot-hub-java-java-getstarted.md
[Azure IoT 开发人员中心]: /develop/iot
[lnk-service-fabric]: ../service-fabric/index.md
[lnk-stream-analytics]: ../stream-analytics/index.md
[lnk-event-hubs]: ../event-hubs/index.md
[Transient Fault Handling]: https://msdn.microsoft.com/en-us/library/hh675232.aspx

<!-- Links -->
[关于 Azure 存储]: ../storage/storage-create-storage-account.md#create-a-storage-account
[事件中心入门]: ../event-hubs/event-hubs-java-ephjava-getstarted.md
[Azure 存储可缩放性指导原则]: ../storage/storage-scalability-targets.md
[Azure Block Blobs]: https://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx
[事件中心]: ../event-hubs/event-hubs-overview.md
[Event Hubs Programming Guide]: https://github.com/Azure/azure-event-hubs/blob/master/java/readme.md
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

[lnk-classic-portal]: https://manage.windowsazure.cn
[lnk-c2d]: ./iot-hub-java-java-process-d2c.md
[lnk-suite]: ../iot-suite/index.md

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdk-java
[lnk-create-an-iot-hub]: ./iot-hub-java-java-getstarted.md#create-an-iot-hub

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description:update wording and code-->