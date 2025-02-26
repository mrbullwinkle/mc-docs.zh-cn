---
title: 了解 Azure IoT 中心云到设备的消息传送 | Azure
description: 本开发人员指南介绍如何在 IoT 中心使用云到设备的消息传送。 其中包含有关消息生命周期和配置选项的信息。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
origin.date: 03/15/2018
ms.author: v-yiso
ms.date: 07/15/2019
ms.openlocfilehash: bc5fbe66a1698a33dd80a8129ea10556c8ebaac7
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67569999"
---
# <a name="send-cloud-to-device-messages-from-an-iot-hub"></a>从 IoT 中心发送云到设备的消息

若要从解决方案后端向设备应用发送单向通知，可从 IoT 中心向设备发送云到设备的消息。 有关 Azure IoT 中心支持的其他云到设备选项的介绍，请参阅[云到设备的通信指南](iot-hub-devguide-c2d-guidance.md)。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

可以通过面向服务的终结点 */messages/devicebound* 发送从云到设备的消息。 随后设备可以通过特定于设备的终结点 */devices/{deviceId}/messages/devicebound* 接收这些消息。

要将每个从云到设备的消息都设为以单个设备为目标，请通过 IoT 中心将 **to** 属性设置为 */devices/{deviceId}/messages/devicebound*。

每个设备队列最多可以保留 50 条云到设备的消息。 尝试将更多消息传送到同一设备会导致错误。

## <a name="the-cloud-to-device-message-lifecycle"></a>云到设备的消息生命周期

为了保证至少一次消息传递，IoT 中心将云到设备的消息保留在每个设备队列中。 设备必须显式确认完成，IoT 中心才会从队列中删除消息  。 这种方法可以保证连接失败和设备故障时能够复原。

下图显示了生命周期状态图：

![云到设备的消息生命周期](./media/iot-hub-devguide-messages-c2d/lifecycle.png)

IoT 中心服务向设备发送消息时，该服务会将消息状态设置为“排队”  。 当设备想要接收某条消息时，IoT 中心会通过将状态设置为“不可见”来锁定该消息。    这种状态使得设备上的其他线程可以开始接收其他消息。 当设备线程完成消息的处理后，会通过完成消息来通知 IoT 中心  。 随后 IoT 中心会将状态设置为“已完成”  。

设备还可以：

* 拒绝  消息，这会使 IoT 中心将此消息设置为“死信”状态  。 通过消息队列遥测传输 (MQTT) 协议进行连接的设备无法拒绝云到设备的消息。

* 放弃消息，这会使 IoT 中心将消息放回队列，并将状态设置为“已排队”   。 通过 MQTT 协议连接的设备无法放弃云到设备的消息。

线程可能无法处理消息，且不通知 IoT 中心。 在此情况下，在可见性超时（或锁定超时）之后，消息从不可见状态自动转换回已排队状态     。 此超时的默认值为一分钟。

消息可以在“已排队”与“不可见”状态之间转换的次数，以 IoT 中心上“最大传送计数”属性中指定的次数为上限    。 在该转换次数之后，IoT 中心会将消息的状态设置为“死信”  。 同样，IoT 中心也会在消息的到期时间之后，将消息的状态设置为“死信”  。 有关详细信息，请参阅[生存时间](#message-expiration-time-to-live)。

[如何使用 IoT 中心发送云到设备的消息](iot-hub-csharp-csharp-c2d.md)一文介绍了如何从云端发送云到设备的消息以及如何在设备上接收这些消息。

通常只要丢失消息不影响应用程序逻辑，设备就会完成云到设备的消息。 例如，设备已在本地保留消息内容或已成功执行操作。 消息还可能携带暂时性信息，这类信息的丢失不会影响应用程序的功能。 有时，对于长时间运行的任务，可以：

* 在设备将任务说明保留到本地存储后完成该云到设备的消息。
* 在作业进度的不同阶段，可以使用一条或多条设备到云的消息通知解决方案后端。

## <a name="message-expiration-time-to-live"></a>消息到期时间（生存时间）

每条云到设备的消息都有过期时间。 可通过以下任一方式设置此时间：

* 服务中的 **ExpiryTimeUtc** 属性
* 使用了指定为 IoT 中心属性的默认生存时间的 IoT 中心  。

请参阅[云到设备的配置选项](#cloud-to-device-configuration-options)。

利用消息到期时间并避免将消息发送到已断开连接的设备的常见方法是设置较短的生存时间值  。 此方法可达到与维护设备连接状态一样的效果，而且更加有效。 请求消息确认时，IoT 中心将通知你哪些设备：

* 可以接收消息。
* 不处于联机状态，或出现故障。

## <a name="message-feedback"></a>消息反馈

发送云到设备的消息时，服务可以请求传送每条消息的反馈（关于该消息的最终状态）。 为此，可将要发送的设备到云消息中的 **iothub-ack** 应用程序属性设置为以下四个值之一：

| Ack 属性值 | 行为 |
| ------------ | -------- |
| 无     | IoT 中心不生成反馈消息（默认行为）。 |
| 积极 | 如果云到设备的消息达到“已完成”状态，IoT 中心将生成反馈消息  。 |
| negative | 云到设备的消息达到“死信”状态时，IoT 中心生成反馈消息  。 |
| full     | IoT 中心在任一情况下都会生成反馈消息。 |

如果 **Ack** 值为 *full*，且未收到反馈消息，则意味着反馈消息已过期。 该服务无法了解原始消息的经历。 实际上，服务应该确保它可以在反馈过期之前对其进行处理。 最长过期时间是两天，因此当发生故障时，有时间让服务再次运行。

如[终结点](iot-hub-devguide-endpoints.md)中所述，IoT 中心通过面向服务的终结点 */messages/servicebound/feedback* 以消息方式传送反馈。 接收反馈的语义与云到设备消息的语义相同。 可能的话，消息反馈将放入单个消息中，其格式如下：

| 属性     | 说明 |
| ------------ | ----------- |
| EnqueuedTime | 一个时间戳，指示中心收到反馈消息的时间 |
| UserId       | `{iot hub name}` |
| ContentType  | `application/vnd.microsoft.iothub.feedback.json` |

正文是记录的 JSON 序列化数组，每条记录具有以下属性：

| 属性           | 说明 |
| ------------------ | ----------- |
| EnqueuedTimeUtc    | 一个时间戳，指示消息的结果（例如，中心已收到反馈消息，或原始消息已过期） |
| OriginalMessageId  | 与此反馈信息相关的从云到设备的消息的 *MessageId* |
| StatusCode         | 必需的字符串，在 IoT 中心生成的反馈消息中使用： <br/> *Success* <br/> *Expired* <br/> *DeliveryCountExceeded* <br/> *Rejected* <br/> *Purged* |
| 说明        | *StatusCode* 的字符串值 |
| DeviceId           | 与此反馈信息相关的从云到设备的消息的目标设备的 *DeviceId* |
| DeviceGenerationId | 与此反馈信息相关的从云到设备的消息的目标设备的 *DeviceGenerationId* |

服务必须指定 *MessageId* ，云到设备的消息才能将其反馈与原始消息相关联。

以下代码演示了反馈消息的正文：

```json
[
  {
    "OriginalMessageId": "0987654321",
    "EnqueuedTimeUtc": "2015-07-28T16:24:48.789Z",
    "StatusCode": 0,
    "Description": "Success",
    "DeviceId": "123",
    "DeviceGenerationId": "abcdefghijklmnopqrstuvwxyz"
  },
  {
    ...
  },
  ...
]
```

## <a name="cloud-to-device-configuration-options"></a>云到设备的配置选项

每个 IoT 中心都针对云到设备的消息传送公开以下配置选项：

| 属性                  | 说明 | 范围和默认值 |
| ------------------------- | ----------- | ----------------- |
| defaultTtlAsIso8601       | 云到设备消息的默认 TTL | ISO_8601 间隔，最大为 2 天（最小为 1 分钟）；默认值：1 小时 |
| maxDeliveryCount          | 每个设备队列的云到设备最大传送计数 | 1 到 100；默认值：10 个 |
| feedback.ttlAsIso8601     | 服务绑定反馈消息的保留时间 | ISO_8601 间隔，最大为 2 天（最小为 1 分钟）；默认值：1 小时 |
| feedback.maxDeliveryCount | 反馈队列的最大传送计数 | 1 到 100；默认值：100 |

有关如何设置这些配置选项的详细信息，请参阅[创建 IoT 中心][lnk-portal]。

## <a name="next-steps"></a>后续步骤

有关可用于接收云到设备消息的 SDK 的信息，请参阅 [Azure IoT SDK][lnk-sdks]。

若要尝试接收云到设备消息，请参阅[发送云到设备][lnk-c2d-tutorial]教程。

[img-lifecycle]: ./media/iot-hub-devguide-messages-c2d/lifecycle.png

[lnk-portal]: ./iot-hub-create-through-portal.md
[lnk-c2d-guidance]: ./iot-hub-devguide-c2d-guidance.md
[lnk-endpoints]: ./iot-hub-devguide-endpoints.md
[lnk-sdks]: ./iot-hub-devguide-sdks.md
[lnk-ttl]: #message-expiration-time-to-live
[lnk-c2d-configuration]: #cloud-to-device-configuration-options
[lnk-lifecycle]: #message-lifecycle
[lnk-c2d-tutorial]: ./iot-hub-csharp-csharp-c2d.md
