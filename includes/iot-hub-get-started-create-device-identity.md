## 创建设备标识
在本部分中，使用名为 [IoT 中心资源管理器][iot-hub-explorer]的 Node.js 工具为本教程创建设备标识。

1. 在命令行环境中运行以下命令：

    ```
    npm install -g iothub-explorer@latest
    ```
2. 然后，运行以下命令登录中心，请记得使用之前复制的 IoT 中心连接字符串替换 `{iot hub connection string}`：

    ```
    iothub-explorer login "{iot hub connection string}"
    ```
3. 最后，以下使用命令创建名为 `myDeviceId` 的新设备标识：

    ```
    iothub-explorer create myDeviceId --connection-string
    ```

记下结果中的设备连接字符串。设备应用使用此设备连接字符串以设备身份连接到 IoT 中心。

![][img-identity]  

若要了解以编程方式创建设备标识的方法，请参阅 [IoT 中心入门][lnk-getstarted]。

<!-- images and links -->

[img-identity]: ./media/iot-hub-get-started-create-device-identity/devidentity.png

[iot-hub-explorer]: https://github.com/Azure/iothub-explorer/blob/master/readme.md

[lnk-getstarted]: ../articles/iot-hub/iot-hub-csharp-csharp-getstarted.md

<!---HONumber=Mooncake_0206_2017-->