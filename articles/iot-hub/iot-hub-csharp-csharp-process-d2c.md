---
title: 使用路由处理 Azure IoT 中心设备到云消息 (.NET) | Azure
description: 如何使用路由将消息发送到其他后端服务，从而处理 IoT 中心设备到云消息。
services: iot-hub
documentationcenter: .net
author: dominicbetts
manager: timlt
editor: ''

ms.assetid: 5177bac9-722f-47ef-8a14-b201142ba4bc
ms.service: iot-hub
ms.devlang: csharp
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/12/2016
wacn.date: 01/13/2017
ms.author: dobett
---

# 使用路由处理 IoT 中心设备到云消息 \(.NET\)

[!INCLUDE [iot-hub-selector-process-d2c](../../includes/iot-hub-selector-process-d2c.md)]

## 介绍
Azure IoT 中心是一项完全托管的服务，可在数百万台设备和单个解决方案后端之间实现安全可靠的双向通信。其他教程（[《IoT 中心入门》]和[《使用 IoT 中心发送云到设备的消息》][lnk-c2d]）介绍了如何使用 IoT 中心的“设备到云”和“云到设备”的基本消息传递功能。

本教程以 [IoT 中心入门]教程中所示的代码为基础，说明如何以基于配置的轻松方式，使用消息路由发送设备到云消息。本教程介绍如何隔离需要解决方案后端立即执行操作以进行进一步处理的消息。例如，设备可能将发送一条警报消息，触发在 CRM 系统中插入票证。与此相反，数据点消息仅送入分析引擎。例如，设备中存储便于日后分析的温度遥测是数据点消息。

在本教程结束时，会运行 3 个 .NET 控制台应用：

* **SimulatedDevice**，[IoT 中心入门]教程中创建的应用的修改版本，每秒发送一次数据点设备到云消息，每 10 秒发送一次交互式设备到云消息。此应用使用 AMQP 协议来与 IoT 中心通信。
* **ReadDeviceToCloudMessages**，显示模拟设备应用发送的非关键遥测数据。
* **read-critical-queue**，从附加到 IoT 中心的服务总线队列中取消模拟设备应用发送的关键消息的排队。

> [!NOTE]
IoT 中心对许多设备平台和语言（包括 C、Java 和 JavaScript）提供 SDK 支持。若要了解如何将本教程中的模拟设备替换为物理设备，以及如何将设备连接到 IoT 中心，请参阅 [Azure IoT 开发人员中心]。
> 
> 

要完成本教程，需要具备以下先决条件：

* Microsoft Visual Studio 2015。
* 有效的 Azure 帐户。<br/>如果没有帐户，只需花费几分钟就能创建一个[帐户](https://www.azure.cn/pricing/1rmb-trial/)。

应具备 [Azure 存储]和 [Azure 服务总线]的一些基础知识。

## 从模拟设备应用发送交互式消息
在本部分中，会修改 [IoT 中心入门]教程中创建的模拟设备应用，以便不定期地发送需要立即处理的消息。

1. 在 Visual Studio 的 **SimulatedDevice** 项目中，将 `SendDeviceToCloudMessagesAsync` 方法替换为以下代码。

    ```
    private static async void SendDeviceToCloudMessagesAsync()
    {
        double avgWindSpeed = 10; // m/s
        Random rand = new Random();

        while (true)
        {
            double currentWindSpeed = avgWindSpeed + rand.NextDouble() * 4 - 2;

            var telemetryDataPoint = new
            {
                deviceId = "myFirstDevice",
                windSpeed = currentWindSpeed
            };
            var messageString = JsonConvert.SerializeObject(telemetryDataPoint);
            string levelValue;

            if (rand.NextDouble() > 0.7)
            {
                messageString = "This is a critical message";
                levelValue = "critical";
            }
            else
            {
                levelValue = "normal";
            }

            var message = new Message(Encoding.ASCII.GetBytes(messageString));
            message.Properties.Add("level", levelValue);

            await deviceClient.SendEventAsync(message);
            Console.WriteLine("{0} > Sent message: {1}", DateTime.Now, messageString);

            await Task.Delay(1000);
        }
    }
    ```

     这会将 `"level": "critical"` 属性随机添加到设备发送的消息，可模拟需要解决方案后端立即执行操作的消息。设备应用会在消息属性中（而非在消息正文中）传递此信息，以便 IoT 中心能够将消息路由到适当的消息目标。

    > [!NOTE]
    > 可使用消息属性根据各种方案路由消息，包括冷路径处理和此处所示的热路径示例。
    > 
    > 

    > [!NOTE]
    > 为简单起见，本教程不实现任何重试策略。在生产代码中，应按 MSDN 文章 [Transient Fault Handling]（暂时性故障处理）中所述实施指数退让等重试策略。
    > 
    > 

## 向 IoT 中心添加一个队列并向其路由消息
在本部分中，将创建一个服务总线队列并将其连接到 IoT 中心，还会配置 IoT 中心，根据消息上的现有属性发送消息到队列。若要深入了解如何处理来自服务总线队列的消息，请参阅[队列入门][Service Bus queue]教程。

1. 按[队列入门][Service Bus queue]中所述，创建服务总线队列。队列必须与 IoT 中心位于同一订阅和区域中。记下命名空间和队列名称。

2. 在 Azure 门户中，打开 IoT 中心并单击“终结点”。

    ![IoT 中心的终结点][30]  

3. 在“终结点”边栏选项卡中，单击顶部的“添加”，将队列添加到 IoT 中心。将终结点命名为“CriticalQueue”，并使用下拉列表选择“服务总线队列”、队列所在的服务总线命名空间和队列名称。完成后，单击底部的“保存”。

    ![添加终结点][31]  

4. 现在单击 IoT 中心的“路由”。单击边栏选项卡顶部的“添加”，创建将消息路由到刚添加的队列的规则。选择“DeviceTelemetry”作为数据源。输入 `level="critical"` 作为条件，然后选择刚添加为终结点的队列作为路由终结点。完成后，单击底部的“保存”。

    ![添加路由][32]  

    请确保回退路由设为“开”。这是 IoT 中心的默认配置。

    ![回退路由][33]  

## 从队列终结点读取
在本部分中，会从队列终结点读取消息。

1. 在当前的 Visual Studio 解决方案中，使用“控制台应用程序”项目模板创建 Visual C\# Windows 项目。将项目命名为 **ReadCriticalQueue**。

2. 在“解决方案资源管理器”中，右键单击 **ReadCriticalQueue** 项目，然后单击“管理 NuGet 包”。此操作会显示“NuGet 包管理器”窗口。

3. 搜索 **WindowsAzure.ServiceBus**，单击“安装”并接受使用条款。此操作会下载和安装 Azure 服务总线及其所有依赖项，并添加对它的引用。

4. 在 **Program.cs** 文件的顶部添加以下 **using** 语句：

    ```
    using System.IO;
    using Microsoft.ServiceBus.Messaging;
    ```

5. 最后，在 **Main** 方法中添加以下行。将连接字符串替换为队列的 **Listen** 权限：

    ```
    Console.WriteLine("Receive critical messages. Ctrl-C to exit.\n");
    var connectionString = "{service bus listen string}";
    var queueName = "{queue name}";

    var client = QueueClient.CreateFromConnectionString(connectionString, queueName);

    client.OnMessage(message =>
        {
            Stream stream = message.GetBody<Stream>();
            StreamReader reader = new StreamReader(stream, Encoding.ASCII);
            string s = reader.ReadToEnd();
            Console.WriteLine(String.Format("Message body: {0}", s));
        });

    Console.ReadLine();
    ```

## 运行应用程序
现在，你已准备就绪，可以运行应用程序了。

1. 在 Visual Studio 的解决方案资源管理器中，右键单击你的解决方案并选择“设置启动项目”。选择“多个启动项目”，然后为 **ReadDeviceToCloudMessages**、**SimulatedDevice** 和 **ReadCriticalQueue** 项目选择“启动”作为操作。
2. 按 **F5** 启动 3 个控制台应用。**ReadDeviceToCloudMessages** 应用仅拥有 **SimulatedDevice** 应用程序发送的非关键消息，而 **ReadCriticalQueue** 应用仅拥有关键消息。

    ![3 个控制台应用][50]  

## 后续步骤
在本教程中，介绍了如何使用 IoT 中心的消息路由功能可靠地分派设备到云的消息。

通过[如何使用 IoT 中心发送云到设备的消息][lnk-c2d]，了解如何将消息从解决方案后端发送到设备。

若要查看使用 IoT 中心完成端到端解决方案的示例，请参阅 [Azure IoT 套件][lnk-suite]。

若要深入了解如何使用 IoT 中心开发解决方案，请参阅 [IoT 中心开发人员指南]。

若要详细了解 IoT 中心的消息路由，请参阅[使用 IoT 中心发送和接收消息][lnk-devguide-messaging]。

<!-- Images. -->

[50]: ./media/iot-hub-csharp-csharp-process-d2c/run1.png
[10]: ./media/iot-hub-csharp-csharp-process-d2c/create-identity-csharp1.png

[30]: ./media/iot-hub-csharp-csharp-process-d2c/click-endpoints.png
[31]: ./media/iot-hub-csharp-csharp-process-d2c/endpoint-creation.png
[32]: ./media/iot-hub-csharp-csharp-process-d2c/route-creation.png
[33]: ./media/iot-hub-csharp-csharp-process-d2c/fallback-route.png

<!-- Links -->

[Azure Blob storage]: ../storage/storage-dotnet-how-to-use-blobs.md
[Azure Data Factory]: ../data-factory/index.md
[HDInsight (Hadoop)]: ../hdinsight/index.md
[Service Bus Queue]: ../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md

[Azure IoT Hub developer guide - Device to cloud]: ./iot-hub-devguide-messaging.md

[Azure 存储]: ../storage/index.md
[Azure 服务总线]: ../service-bus/index.md

[IoT 中心开发人员指南]: ./iot-hub-devguide.md
[IoT 中心入门]: ./iot-hub-csharp-csharp-getstarted.md
[《IoT 中心入门》]: ./iot-hub-csharp-csharp-getstarted.md
[lnk-devguide-messaging]: ./iot-hub-devguide-messaging.md
[Azure IoT 开发人员中心]: /develop/iot
[lnk-service-fabric]: ../service-fabric/index.md
[lnk-stream-analytics]: ../stream-analytics/index.md
[lnk-event-hubs]: ../event-hubs/index.md

<!-- Links -->

[About Azure Storage]: ../storage/storage-create-storage-account.md#create-a-storage-account
[Get Started with Event Hubs]: ../event-hubs/event-hubs-csharp-ephcs-getstarted.md
[Azure Storage scalability Guidelines]: ../storage/storage-scalability-targets.md
[Azure Block Blobs]: https://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx
[Event Hubs]: ../event-hubs/event-hubs-overview.md
[EventProcessorHost]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.eventprocessorhost(v=azure.95).aspx
[Event Hubs Programming Guide]: ../event-hubs/event-hubs-programming-guide.md
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[Build multi-tier applications with Service Bus]: ../service-bus-messaging/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md

[lnk-classic-portal]: https://manage.windowsazure.cn
[lnk-c2d]: ./iot-hub-csharp-csharp-process-d2c.md
[lnk-suite]: ../iot-suite/index.md

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording and code-->