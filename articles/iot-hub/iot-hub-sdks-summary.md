---
title: Azure IoT 中心 SDK 的列表 | Azure
description: 有关各种 Azure IoT 中心设备和服务 SDK 的信息以及指向这些 SDK 的链接。
services: iot-hub
documentationCenter: ''
authors: dominicbetts
manager: timlt
editor: ''

ms.service: iot-hub
ms.date: 09/13/2016
wacn.date: 01/04/2017
---

# IoT 中心 SDK

本文提供有关各种 [Azure IoT SDK][] 的信息以及指向其他资源的链接。

## IoT 中心设备 SDK

Azure IoT 设备 SDK 包含的代码可帮助构建连接到 Azure IoT 中心服务并由这些服务管理的设备和应用程序。

以下 IoT 设备 SDK 可以从 GitHub 进行下载：

- [适用于 C 的 Azure IoT 设备 SDK][]，采用 ANSI C (C99) 编写，具有可移植性和广泛的平台兼容性。
- 适用于 .NET 的 Azure IoT 设备 SDK
- [适用于 Java 的 Azure IoT 设备 SDK][]
- 适用于 Python 2.7 的 Azure IoT 设备 SDK

### 操作系统平台和硬件兼容性

有关与特定硬件设备兼容性的详细信息，请参阅以下文章：

- [OS 平台和硬件与设备 SDK 的兼容性][OS Platforms and hardware compatibility]

## IoT 中心服务 SDK

Azure IoT 服务 SDK 包含的代码可帮助构建直接与 IoT 中心进行交互以管理设备和安全性的应用程序。

可从 GitHub 下载以下 IoT 服务 SDK：

- [适用于 Java 的 Azure IoT 服务 SDK][]

## Azure IoT 网关 SDK

此 Azure IoT 网关 SDK 包含创建 IoT 网关解决方案的基础结构和模块。你可以扩展此 SDK 来创建适用于任何端到端场景的网关。

你可以从 GitHub 下载 [Azure IoT 网关 SDK][]。

## 联机 API 参考文档

下面是 Azure IoT 设备、服务和网关库的联机 API 参考文档的链接列表：

- [物联网 (IoT) .NET][]
- [IoT 中心 REST][]
- [适用于 C 的 Azure IoT 设备 SDK][]
- [适用于 Java 的 Azure IoT 设备 SDK][]
- [适用于 Java 的 Azure IoT 服务 SDK][]
- [Azure IoT 网关 SDK][]

## 后续步骤

若要进一步探索 IoT 中心的功能，请参阅：

- [设计你的解决方案][lnk-design]
- [使用网关 SDK 模拟设备][lnk-gateway]
- [使用 Azure 门户管理 IoT 中心][lnk-portal]

[Azure IoT SDK]: https://github.com/Azure/azure-iot-sdks/blob/master/readme.md
[适用于 C 的 Azure IoT 设备 SDK]: https://github.com/Azure/azure-iot-sdks/blob/master/c/readme.md
[适用于 .NET 的 Azure IoT 设备 SDK]: https://github.com/Azure/azure-iot-sdks/blob/master/csharp/device/readme.md
[适用于 Java 的 Azure IoT 设备 SDK]: https://github.com/Azure/azure-iot-sdks/blob/master/java/device/readme.md
[适用于 Java 的 Azure IoT 服务 SDK]: https://github.com/Azure/azure-iot-sdks/blob/master/java/service/readme.md
[适用于 Node.js 的 Azure IoT 设备 SDK]: https://github.com/Azure/azure-iot-sdks/blob/master/node/device/readme.md
[适用于 Node.js 的 Azure IoT 服务 SDK]: https://github.com/Azure/azure-iot-sdks/blob/master/node/service/README.md
[适用于 Python 2.7 的 Azure IoT 设备 SDK]: https://github.com/Azure/azure-iot-sdks/blob/master/python/device/readme.md
[OS Platforms and hardware compatibility]: ./iot-hub-tested-configurations.md

[Azure IoT Gateway SDK]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/README.md

[物联网 (IoT) .NET]: https://msdn.microsoft.com/zh-cn/library/mt488521.aspx
[适用于 C 的 Azure IoT 设备 SDK]: http://azure.github.io/azure-iot-sdks/c/api_reference/index.html
[适用于 Java 的 Azure IoT 设备 SDK]: http://azure.github.io/azure-iot-sdks/java/device/api_reference/index.html
[适用于 Node.js 的 Azure IoT 设备 SDK]: http://azure.github.io/azure-iot-sdks/node/api_reference/azure-iot-device/1.0.3/index.html
[IoT 中心 REST]: https://msdn.microsoft.com/zh-cn/library/mt548492.aspx
[适用于 Java 的 Azure IoT 服务 SDK]: http://azure.github.io/azure-iot-sdks/java/service/api_reference/index.html
[适用于 Node.js 的 Azure IoT 服务 SDK]: http://azure.github.io/azure-iot-sdks/node/api_reference/azure-iothub/1.0.3/index.html
[Azure IoT 网关 SDK]: http://azure.github.io/azure-iot-gateway-sdk/api_reference/c/html/
[lnk-design]: ./iot-hub-guidance.md
[lnk-gateway]: ./iot-hub-linux-gateway-sdk-simulated-device.md
[lnk-portal]: ./iot-hub-manage-through-portal.md

<!---HONumber=Mooncake_Quality_Review_1230_2016-->