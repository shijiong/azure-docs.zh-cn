---
title: "Azure IoT 中心操作监视 | Microsoft Docs"
description: "如何使用 Azure IoT 中心操作监视功能实时监视 IoT 中心上的操作状态。"
services: iot-hub
documentationcenter: 
author: nberdy
manager: timlt
editor: 
ms.assetid: a299f3a5-b14d-4586-9c3b-44aea14ed013
ms.service: iot-hub
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/10/2017
ms.author: nberdy
ms.openlocfilehash: db03cfdd486a3172b258379928fac12cc0af730a
ms.sourcegitcommit: 933af6219266cc685d0c9009f533ca1be03aa5e9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/18/2017
---
# <a name="iot-hub-operations-monitoring"></a>IoT 中心操作监视

IoT 中心操作监视可让你实时监视 IoT 中心上的操作状态。 IoT 中心可以跟踪多个操作类别的事件。 可以选择将一个或多个类别的事件发送到 IoT 中心的终结点进行处理。 可以监视数据中是否有错误，或根据数据模式设置更复杂的处理行为。

>[!NOTE]
>IoT 中心操作监视已弃用，在未来将从 IoT 中心中删除。 有关如何监视 IoT 中心的操作和运行状况，请参阅[监视 Azure IoT 中心运行状况并快速诊断问题][lnk-monitor]。 要详细了解弃用日程表，请参阅[利用 Azure Monitor 和 Azure 资源运行状况监视 Azure IoT 解决方案][lnk-blog-announcement]。

IoT 中心监视 6 种类别的事件：

* 设备标识操作
* 设备遥测
* 云到设备的消息
* 连接
* 文件上传
* 消息路由

> [!IMPORTANT]
> IoT 中心操作监视不保证事件传送的可靠性和有序性。 某些事件可能丢失或出现传送顺序混乱，具体取决于 IoT 中心的基础结构。 使用操作监视基于错误信号生成警报，如连接尝试失败或与特定设备的连接频繁断开。 不应依赖操作监视事件为设备状态创建一致存储区，例如，跟踪设备已连接或断开连接状态的存储区。 

## <a name="how-to-enable-operations-monitoring"></a>如何启用操作监视

1. 创建 IoT 中心。 有关如何创建 IoT 中心的说明，请参阅[入门][lnk-get-started]指南。

1. 打开 IoT 中心的边栏选项卡。 在此处单击“**操作监视**”。

    ![访问门户中的操作监视配置][1]

1. 选择要监视的监视类别，并单击“保存”。 可以从“**监视设置**”中所列的与事件中心兼容的终结点读取事件。 IoT 中心终结点称为 `messages/operationsmonitoringevents`。

    ![在 IoT 中心配置操作监视][2]

> [!NOTE]
> 为“连接”类别选择“详细”监视会导致 IoT 中心生成额外的诊断消息。 对于所有其他类别，“详细”设置会更改 IoT 中心在每个错误消息中包含的信息数量。

## <a name="event-categories-and-how-to-use-them"></a>事件类别及其用法

每种操作监视类别跟踪与 IoT 中心之间进行的不同类型的交互，每一种监视类别都有一个架构用于定义如何构建该类别的事件。

### <a name="device-identity-operations"></a>设备标识操作

设备标识操作类别跟踪你尝试在 IoT 中心的标识注册表中创建、更新或删除条目时所发生的错误。 预配方案就很适合跟踪此类别。

```json
{
    "time": "UTC timestamp",
    "operationName": "create",
    "category": "DeviceIdentityOperations",
    "level": "Error",
    "statusCode": 4XX,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "durationMs": 1234,
    "userAgent": "userAgent",
    "sharedAccessPolicy": "accessPolicy"
}
```

### <a name="device-telemetry"></a>设备遥测

设备遥测类别跟踪在 IoT 中心发生的、与遥测管道相关的错误。 此类别包括发送遥测事件（例如限制）和接收遥测事件（例如未经授权的读取者）时发生的错误。 此类别无法捕捉设备本身运行的代码所造成的错误。

```json
{
    "messageSizeInBytes": 1234,
    "batching": 0,
    "protocol": "Amqp",
    "authType": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "time": "UTC timestamp",
    "operationName": "ingress",
    "category": "DeviceTelemetry",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "EventProcessedUtcTime": "UTC timestamp",
    "PartitionId": 1,
    "EventEnqueuedUtcTime": "UTC timestamp"
}
```

### <a name="cloud-to-device-commands"></a>云到设备的命令

云到设备的命令类别跟踪在 IoT 中心发生且与云到设备的消息管道相关的错误。 此类别包括下述情况下发生的错误：发送云到设备的消息（例如未经授权的发送者）、接收云到设备的消息（例如超过传递计数），以及接收云到设备的消息反馈（例如反馈已过期）。 此类别不捕捉未正确处理云到设备的消息但却将其成功传递的设备所发生的错误。

```json
{
    "messageSizeInBytes": 1234,
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "deliveryAcknowledgement": 0,
    "protocol": "Amqp",
    "time": " UTC timestamp",
    "operationName": "ingress",
    "category": "C2DCommands",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "EventProcessedUtcTime": "UTC timestamp",
    "PartitionId": 1,
    "EventEnqueuedUtcTime": "UTC timestamp"
}
```

### <a name="connections"></a>连接

连接类别跟踪设备与 IoT 中心连接或断开连接时发生的错误。 若要识别未经授权的连接尝试，以及在连接质量不佳的区域中的设备断开连接时进行跟踪，就很适合跟踪此类别。

```json
{
    "durationMs": 1234,
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "protocol": "Amqp",
    "time": " UTC timestamp",
    "operationName": "deviceConnect",
    "category": "Connections",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID"
}
```

### <a name="file-uploads"></a>文件上传

文件上传类别跟踪在 IoT 中心发生的、与文件上传功能相关的错误。 此类别包括：

* SAS URI 发生的错误，例如，它在设备向中心通知某个完成的上传前失效。
* 由设备报告的失败上传。
* 创建 IoT 中心通知消息期间在存储中找不到文件时发生的错误。

此类别不能捕获在设备将文件上传到存储时直接发生的错误。

```json
{
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "protocol": "HTTP",
    "time": " UTC timestamp",
    "operationName": "ingress",
    "category": "fileUpload",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "blobUri": "http//bloburi.com",
    "durationMs": 1234
}
```

### <a name="message-routing"></a>消息路由

消息路由类别跟踪消息路由评估期间发生的错误以及 IoT 中心感知到的终结点运行状况。 此类别包括以下事件：规则评估为“未定义”、IoT 中心将终结点标记为“已停用”，以及从终结点中收到的任何其他错误。 此类别不包含有关消息本身的特定错误（如设备限制错误），这些错误在“设备遥测”类别下报告。

```json
{
    "messageSizeInBytes": 1234,
    "time": "UTC timestamp",
    "operationName": "ingress",
    "category": "routes",
    "level": "Error",
    "deviceId": "device-ID",
    "messageId": "ID of message",
    "routeName": "myroute",
    "endpointName": "myendpoint",
    "details": "ExternalEndpointDisabled"
}
```

## <a name="view-events"></a>查看事件

可以使用 *iothub-explorer* 工具快速测试 IoT 中心是否正在生成监视事件。 若要安装该工具，请参阅 [iothub-explorer][lnk-iothub-explorer] GitHub 存储库中的说明。

1. 请确保门户中的“连接”监视类别设置为“详细”。

1. 在命令提示符下，运行以下命令以读取监视终结点：

    ```
    iothub-explorer monitor-ops --login {your iothubowner connection string}
    ```

1. 在另一个命令提示符下，运行以下命令以模拟发送设备到云消息的设备：

    ```
    iothub-explorer simulate-device {your device name} --send "My test message" --login {your iothubowner connection string}
    ```

1. 当模拟设备连接到 IoT 中心时，第一个命令提示符显示监视事件。

## <a name="connect-to-the-monitoring-endpoint"></a>连接到监视终结点

IoT 中心上的监视终结点是与事件中心兼容的终结点。 可使用任何适用于事件中心的机制从此终结点读取监视消息。 以下示例创建的基本读取器不适用于高吞吐量部署。 若要深入了解如何处理来自事件中心的消息，请参阅[事件中心入门][lnk-eventhubs-tutorial]教程。

若要连接到监视终结点，需要一个连接字符串和终结点名称。 以下步骤介绍如何在门户中查找必需的值：

1. 在门户中，导航到 IoT 中心资源边栏选项卡。

1. 选择“操作监视”，记下“与事件中心兼容的名称”和“与事件中心兼容的终结点”值：

    ![“与事件中心兼容的终结点”值][img-endpoints]

1. 选择“共享访问策略”，并选择“服务”。 记下“主密钥”值：

    ![服务共享访问策略主密钥][img-service-key]

以下 C# 代码示例取自 Visual Studio **Windows 经典桌面** C# 控制台应用。 该项目安装了 **WindowsAzure.ServiceBus** NuGet 包。

* 如以下示例所示，将连接字符串占位符替换为使用之前记下的“与事件中心兼容的终结点”和服务“主密钥”值的连接字符串：

    ```cs
    "Endpoint={your Event Hub-compatible endpoint};SharedAccessKeyName=service;SharedAccessKey={your service primary key value}"
    ```

* 将监视终结点名称占位符替换为之前记下的“与事件中心兼容的名称”值。

```cs
class Program
{
    static string connectionString = "{your monitoring endpoint connection string}";
    static string monitoringEndpointName = "{your monitoring endpoint name}";
    static EventHubClient eventHubClient;

    static void Main(string[] args)
    {
        Console.WriteLine("Monitoring. Press Enter key to exit.\n");

        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, monitoringEndpointName);
        var d2cPartitions = eventHubClient.GetRuntimeInformation().PartitionIds;
        CancellationTokenSource cts = new CancellationTokenSource();
        var tasks = new List<Task>();

        foreach (string partition in d2cPartitions)
        {
            tasks.Add(ReceiveMessagesFromDeviceAsync(partition, cts.Token));
        }

        Console.ReadLine();
        Console.WriteLine("Exiting...");
        cts.Cancel();
        Task.WaitAll(tasks.ToArray());
    }

    private static async Task ReceiveMessagesFromDeviceAsync(string partition, CancellationToken ct)
    {
        var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow);
        while (true)
        {
            if (ct.IsCancellationRequested)
            {
                await eventHubReceiver.CloseAsync();
                break;
            }

            EventData eventData = await eventHubReceiver.ReceiveAsync(new TimeSpan(0,0,10));

            if (eventData != null)
            {
                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                Console.WriteLine("Message received. Partition: {0} Data: '{1}'", partition, data);
            }
        }
    }
}
```

## <a name="next-steps"></a>后续步骤
若要进一步探索 IoT 中心的功能，请参阅：

* [IoT 中心开发人员指南][lnk-devguide]
* [使用 Azure IoT Edge 将 AI 部署到边缘设备][lnk-iotedge]

<!-- Links and images -->
[1]: media/iot-hub-operations-monitoring/enable-OM-1.png
[2]: media/iot-hub-operations-monitoring/enable-OM-2.png
[img-endpoints]: media/iot-hub-operations-monitoring/monitoring-endpoint.png
[img-service-key]: media/iot-hub-operations-monitoring/service-key.png

[lnk-blog-announcement]: https://azure.microsoft.com/blog/monitor-your-azure-iot-solutions-with-azure-monitor-and-azure-resource-health
[lnk-monitor]: iot-hub-monitor-resource-health.md
[lnk-get-started]: iot-hub-csharp-csharp-getstarted.md
[lnk-diagnostic-metrics]: iot-hub-metrics.md
[lnk-scaling]: iot-hub-scaling.md
[lnk-dr]: iot-hub-ha-dr.md

[lnk-devguide]: iot-hub-devguide.md
[lnk-iotedge]: ../iot-edge/tutorial-simulate-device-linux.md
[lnk-iothub-explorer]: https://github.com/azure/iothub-explorer
[lnk-eventhubs-tutorial]: ../event-hubs/event-hubs-csharp-ephcs-getstarted.md
