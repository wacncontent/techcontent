---
title: 如何通过 Java 使用服务总线队列 | Azure
description: 了解如何在 Azure 中使用服务总线队列。用 Java 编写的代码示例。
services: service-bus
documentationCenter: java
authors: sethmanheim
manager: timlt

ms.service: service-bus
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 01/11/2017
ms.author: sethm
wacn.date: 02/20/2017
---

# 如何使用服务总线队列

[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

本文介绍了如何使用服务总线队列。相关示例用 Java 编写且使用 [Azure SDK for Java][Azure SDK for Java]。涉及的应用场景包括**创建队列**、**发送和接收消息**以及**删除队列**。

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]

## 创建服务命名空间

若要开始在 Azure 中使用服务总线队列，必须先创建一个命名空间。命名空间提供了用于对应用程序中的服务总线资源进行寻址的范围容器。

创建命名空间：

1. 打开 Azure Powershell 控制台窗口。

2. 键入以下命令以创建命名空间。提供你自己的命名空间值，并指定与应用程序相同的区域。

    ```
    New-AzureSBNamespace -Name 'yourexamplenamespace' -Location 'China East' -NamespaceType 'Messaging' -CreateACSNamespace $true
    ```

    ![创建命名空间](./media/service-bus-ruby-how-to-use-topics-subscriptions/showcmdcreate.png)  

## 获取命名空间的默认管理凭据

若要在新命名空间上执行管理操作（如创建队列），则必须获取该命名空间的管理凭据。

你运行的用于创建服务总线命名空间的 PowerShell cmdlet 将显示可用于管理命名空间的密钥。复制 **DefaultKey** 值。你将在本教程稍后的代码中使用此值。

![复制密钥](./media/service-bus-ruby-how-to-use-topics-subscriptions/defaultkey.png)  

记下主密钥，或将其复制到剪贴板。

## 配置应用程序以使用服务总线
在生成本示例之前，请确保已安装 [Azure SDK for Java][Azure SDK for Java]。如果使用了 Eclipse，则可以安装包含 Azure SDK for Java 的 [Azure Toolkit for Eclipse][Azure Toolkit for Eclipse]。然后，你可以将 **Microsoft Azure Libraries for Java** 添加到你的项目：

![](./media/service-bus-java-how-to-use-queues/eclipselibs.png)  

将以下 `import` 语句添加到 Java 文件顶部：

```
    // Include the following imports to use Service Bus APIs
    import com.microsoft.windowsazure.services.servicebus.*;
    import com.microsoft.windowsazure.services.servicebus.models.*;
    import com.microsoft.windowsazure.core.*;
    import javax.xml.datatype.*;
```

## 创建队列

服务总线队列的管理操作可通过 **ServiceBusContract** 类执行。**ServiceBusContract** 对象是使用封装了 SAS 令牌及用于管理它的权限的适当配置构造的，而 **ServiceBusContract** 类是与 Azure 进行通信的单一点。

**ServiceBusService** 类提供了创建、枚举和删除队列的方法。以下示例演示了如何通过名为“HowToSample”的命名空间，使用 **ServiceBusService** 对象创建名为“TestQueue”的队列：

```
    Configuration config =
        ServiceBusConfiguration.configureWithSASAuthentication(
                "HowToSample",
                "RootManageSharedAccessKey",
                "SAS_key_value",
                ".servicebus.chinacloudapi.cn"
                );

ServiceBusContract service = ServiceBusService.create(config);
QueueInfo queueInfo = new QueueInfo("TestQueue");
try
{
    CreateQueueResult result = service.createQueue(queueInfo);
}
catch (ServiceException e)
{
    System.out.print("ServiceException encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
}
```

可对 **QueueInfo** 执行某些方法，以调整队列的属性（例如，将默认的生存时间 (TTL) 值设置为应用于发送到队列的消息）。以下示例演示了如何创建最大大小为 5GB 且名为 `TestQueue` 的队列：

```
long maxSizeInMegabytes = 5120;
QueueInfo queueInfo = new QueueInfo("TestQueue");
queueInfo.setMaxSizeInMegabytes(maxSizeInMegabytes);
CreateQueueResult result = service.createQueue(queueInfo);
```

注意：你可以对 **ServiceBusContract** 对象使用 **listQueues** 方法来检查具有指定名称的队列在某个服务命名空间中是否已存在。

## 向队列发送消息

若要将消息发送到服务总线队列，你的应用程序将获得 **ServiceBusContract** 对象。以下代码演示了如何将消息发送到先前在 `HowToSample` 命名空间中创建的 `TestQueue` 队列。

```
try
{
    BrokeredMessage message = new BrokeredMessage("MyMessage");
    service.sendQueueMessage("TestQueue", message);
}
catch (ServiceException e) 
{
    System.out.print("ServiceException encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
}
```

在服务总线队列中发送和接收的消息是 [BrokeredMessage][] 类的实例。[BrokeredMessage][] 对象包含一组标准属性（如 [Label](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.label.aspx) 和 [TimeToLive](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx)）、一个用来保存自定义应用程序特定属性的词典以及大量随机应用程序数据。应用程序可通过将任何可序列化对象传入到 [BrokeredMessage][] 的构造函数中来设置消息的正文，然后将使用适当的序列化程序来序列化对象。或者，你可以提供 **java.IO.InputStream** 对象。

以下示例演示了如何将五条测试消息发送到在前面的代码段中获取的 `TestQueue` **MessageSender**：

```
for (int i=0; i<5; i++)
{
     // Create message, passing a string message for the body.
     BrokeredMessage message = new BrokeredMessage("Test message " + i);
     // Set an additional app-specific property.
     message.setProperty("MyProperty", i); 
     // Send message to the queue
     service.sendQueueMessage("TestQueue", message);
}
```

服务总线队列在标准层中支持的最大消息大小为 256 KB。标头最大为 64 KB，其中包括标准和自定义应用程序属性。一个队列可包含的消息数不受限制，但消息的总大小受限。此队列大小是在创建时定义的，上限为 5 GB。

## 从队列接收消息

从队列接收消息的主要方法是使用 **ServiceBusContract** 对象。收到的消息可在两种不同模式下工作：**ReceiveAndDelete** 和 **PeekLock**。

当使用 **ReceiveAndDelete** 模式时，接收是一项单次操作，即，当服务总线接收到队列中某条消息的读取请求时，它会将该消息标记为“已使用”并将其返回给应用程序。**ReceiveAndDelete** 模式（默认模式）是最简单的模式，最适合应用程序允许出现故障时不处理消息的方案。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生崩溃。由于服务总线会将消息标记为“已使用”，因此当应用程序重启并重新开始使用消息时，它会遗漏在发生崩溃前使用的消息。

在 **PeekLock** 模式下，接收变成了一个两阶段操作，从而有可能支持无法允许遗漏消息的应用程序。当服务总线收到请求时，它会查找下一条要使用的消息，锁定该消息以防其他使用方接收，然后将该消息返回给应用程序。应用程序完成消息处理（或可靠地存储消息以供将来处理）后，它将通过对收到的消息调用 **Delete** 完成接收过程的第二个阶段。当服务总线发现 **Delete** 调用时，它会将消息标记为“已使用”并将其从队列中删除。

以下示例演示如何使用 **PeekLock** 模式（非默认模式）接收和处理消息。下面的示例将执行无限循环并在消息到达我们的“TestQueue”后进行处理：

```
    try
{
    ReceiveMessageOptions opts = ReceiveMessageOptions.DEFAULT;
    opts.setReceiveMode(ReceiveMode.PEEK_LOCK);

    while(true)  { 
         ReceiveQueueMessageResult resultQM = 
                 service.receiveQueueMessage("TestQueue", opts);
        BrokeredMessage message = resultQM.getValue();
        if (message != null && message.getMessageId() != null)
        {
            System.out.println("MessageID: " + message.getMessageId());    
            // Display the queue message.
            System.out.print("From queue: ");
            byte[] b = new byte[200];
            String s = null;
            int numRead = message.getBody().read(b);
            while (-1 != numRead)
            {
                s = new String(b);
                s = s.trim();
                System.out.print(s);
                numRead = message.getBody().read(b);
            }
            System.out.println();
            System.out.println("Custom Property: " + 
                message.getProperty("MyProperty"));
            // Remove message from queue.
            System.out.println("Deleting this message.");
            //service.deleteMessage(message);
        }  
        else  
        {        
            System.out.println("Finishing up - no more messages.");        
            break; 
            // Added to handle no more messages.
            // Could instead wait for more messages to be added.
        }
    }
}
catch (ServiceException e) {
    System.out.print("ServiceException encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
}
catch (Exception e) {
    System.out.print("Generic exception encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
} 	
```

## 如何处理应用程序崩溃和不可读消息

服务总线提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序出于某种原因无法处理消息，它可以对收到的消息调用 **unlockMessage** 方法（而不是 **deleteMessage** 方法）。这将导致服务总线解锁队列中的消息并使其能够再次被同一个消费应用程序或其他消费应用程序接收。

还存在与队列中的锁定消息关联的超时，如果应用程序未能在锁定超时过期前处理消息（例如，如果应用程序崩溃），则服务总线将自动解锁该消息并使其可再次被接收。

如果在处理消息之后，发出 **deleteMessage** 请求之前应用程序发生崩溃，该消息将在应用程序重新启动时重新传送给它。此情况通常称作**至少处理一次**，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送如果某个场景不允许重复处理，则应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。通常可使用消息的 **getMessageId** 方法实现此操作，这在多个传送尝试中保持不变。

## 后续步骤
现在，你已了解服务总线队列的基础知识，请参阅[队列、主题和订阅][Queues, topics, and subscriptions]以获取更多信息。

有关详细信息，请参阅 [Java 开发人员中心](/develop/java/)。
  [Azure SDK for Java]: /develop/java/
  [Azure Toolkit for Eclipse]: https://msdn.microsoft.com/zh-cn/library/azure/hh694271.aspx

  [Queues, topics, and subscriptions]: ./service-bus-queues-topics-subscriptions.md
  [BrokeredMessage]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description:update wording and link references-->