---
title: Azure 通知中心：常见问题解答 (FAQ) | Azure Docs
description: 关于设计/实现有关通知中心的解决方案的常见问题
services: notification-hubs
documentationcenter: mobile
author: jwargo
manager: patniko
editor: spelluru
keywords: 推送通知, 推送通知, iOS 推送通知, android 推送通知, ios 推送, android 推送
ms.assetid: 7b385713-ef3b-4f01-8b1f-ffe3690bbd40
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
origin.date: 04/14/2018
ms.date: 07/29/2019
ms.author: v-biyu
ms.openlocfilehash: 06b18070916f3d66afbe463cd560c0acbfeba77f
ms.sourcegitcommit: 5f260ee1d8ac487702b554a94cb971a3ee62a40b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2019
ms.locfileid: "68232307"
---
# <a name="push-notifications-with-azure-notification-hubs-frequently-asked-questions"></a>使用 Azure 通知中心推送通知：常见问题

## <a name="general"></a>常规

### <a name="what-is-the-resource-structure-of-notification-hubs"></a>通知中心的资源结构是怎样的？

Azure 通知中心有两个资源级别：中心和命名空间。 中心是单个推送资源，可以保存一个应用的跨平台推送信息。 命名空间是一个区域中所有中心的集合。

建议的映射将一个命名空间与一个应用相匹配。 在命名空间内，可以创建适用于生产应用的生产中心、适用于测试应用的测试中心，等等。

### <a name="what-is-the-price-model-for-notification-hubs"></a>通知中心的价格模型有哪些？

可以在[通知中心定价]页上找到最新的定价详细信息。 通知中心根据命名空间级别计费。 （有关命名空间的定义，请参阅“通知中心的资源结构是怎样的？”）通知中心提供三个层：

* **免费**：此层是探索推送功能的极佳起点。 不建议对生产应用使用此层。 在每个命名空间中，每个月可以预配 500 个设备和执行 100 万次推送，但无法享受服务级别协议 (SLA) 保证。
* **基本**：建议将此层（或标准层）用于较小型的生产应用。 在每个命名空间中，每个月可以预配 200,000 个设备和执行 1,000 万次推送（基准）。
* **标准**：建议将此层用于中型到大型生产应用。 在每个命名空间中，每个月可以预配 1,000 万个设备和执行 1,000 万次推送（基准）。 包含丰富的遥测数据（提供有关推送状态的其他数据）。

标准层功能：

* **丰富的遥测功能**：可以根据消息遥测使用通知中心来跟踪任何推送请求和平台通知系统反馈，以便进行调试。
* **多租户**：可在命名空间级别使用平台通知系统凭据。 此选项可让你轻松地将租户拆分到相同命名空间内的中心。
* **计划推送**：可以计划随时发出通知。
* **批量操作**：如[注册信息导出/导入]文档中所述启用注册信息导出/导入功能。

### <a name="what-is-the-notification-hubs-sla"></a>什么是通知中心 SLA？

对于基本和标准通知中心层，正确配置的应用程序可在 99.9% 的时间发送推送通知或执行注册管理操作。 若要详细了解 SLA，请访问[通知中心 SLA](https://www.azure.cn/support/sla/notification-hubs/) 页。

> [!NOTE]
> 由于推送通知取决于第三方平台通知系统（例如 Apple APNS），所以这些消息的发送不具有 SLA 保证。 在通知中心将批处理发送到平台通知系统（有 SLA 保证）后，平台通知系统将负责执行推送（无 SLA 保证）。

### <a name="how-do-i-upgrade-or-downgrade-my-hub-or-namespace-to-a-different-tier"></a>如何将中心升级或降级到不同层的命名空间？

转到 **[Azure 门户]**  > **通知中心命名空间**或**通知中心**。 选择要更新的资源，转到“定价层”。  请注意以下要求：

* 更新的定价层应用到正在使用的命名空间中的*所有*中心。
* 如果设备计数超出所要降级到的层的限制，则需要删除设备才能降级。

## <a name="design-and-development"></a>设计和开发

### <a name="which-server-side-platforms-do-you-support"></a>支持哪些服务器端平台？

服务器 SDK 适用于 .NET、Java、Node.js、PHP 和 Python。 通知中心 API 基于 REST 接口，因此如果要处理不同的平台或不希望有其他依赖项，可以直接使用 REST API。 有关详细信息，请转到[通知中心 REST API] 页。

### <a name="which-client-platforms-do-you-support"></a>支持哪些客户端平台？

[iOS](notification-hubs-ios-apple-push-notification-apns-get-started.md)、Android、[Windows Universal](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)、[Windows Phone](notification-hubs-windows-mobile-push-notifications-mpns.md)、[Kindle](notification-hubs-kindle-amazon-adm-push-notification.md)、[Android China（通过百度）](notification-hubs-baidu-china-android-notifications-get-started.md)、Xamarin ([iOS](xamarin-notification-hubs-ios-push-notification-apns-get-started.md)) 和 [Safari](https://github.com/Azure/azure-notificationhubs-samples/tree/master/PushToSafari) 支持推送通知。 有关详细信息，请转到[通知中心入门教程]页。

### <a name="do-you-support-text-message-email-or-web-notifications"></a>是否支持短信、电子邮件或 Web 通知？

通知中心主要用于将通知发送到移动应用。 它不提供电子邮件或短信功能。 但是，提供这些功能的第三方平台可与通知中心集成，使用移动应用发送原生推送通知。

通知中心也不提供现成的浏览器内推送通知传递服务。 客户可以在支持的服务器端平台上使用 SignalR 实现此功能。 

### <a name="how-are-mobile-apps-and-azure-notification-hubs-related-and-when-do-i-use-them"></a>移动应用与 Azure 通知中心之间的关系如何？它们各自适用于什么场合？

如果有现成的移动应用后端并且只想添加发送推送通知的功能，则可以使用 Azure 通知中心。 如果想要从头开始安装移动应用后端，请考虑使用 Azure 应用服务的移动应用功能。 移动应用会自动预配通知中心，方便从移动应用后端发送推送通知。 移动应用的定价包括通知中心的基本费用。 只需在超出附送的推送套餐时支付费用。 有关费用的详细信息，请转到[应用服务定价]页。

### <a name="how-many-devices-can-i-support-if-i-send-push-notifications-via-notification-hubs"></a>如果通过通知中心发送推送通知，可以支持多少个设备？

有关支持的设备数目的详细信息，请参阅[通知中心定价]页。

如果需要支持超过 1000 万个注册设备，则必须将设备分布到多个中心。

### <a name="how-many-push-notifications-can-i-send-out"></a>我可以发送多少推送通知？

Azure 通知中心根据系统中通过的通知数量自动向上扩展，具体取决于所选的层。

> [!NOTE]
> 整体使用成本可能会根据目前提供的推送通知数量而增加。 请务必留意[通知中心定价]页上概述的层限制。

我们的客户每天使用通知中心发送数百万条推送通知。 只要使用 Azure 通知中心，就不需要执行任何特别的操作来调整推送通知可及范围。

### <a name="how-long-does-it-take-for-sent-push-notifications-to-reach-my-device"></a>已发送的推送通知到达我的设备需要多长时间？

在正常使用情况下（传入负载相当一致），Azure 通知中心能够*在 1 分钟内处理至少 100 万次推送通知发送*。 此速率可能因标记数、传入发送的性质以及其他外部因素而有所不同。

在预计的传递时间内，服务能计算每个平台的目标，并根据注册的标记或标记表达式将消息路由到推送通知服务 (PNS)。 PNS 负责将通知发送到设备。

PNS 对于传递通知不提供任何 SLA 保证。 但是，大多数推送通知会在发送到通知中心之后的数分钟内（通常在 10 分钟内）传递到目标设备。 少量的通知可能需要更长时间。

> [!NOTE]
> Azure 通知中心有一个策略，可删除任何无法在 30 分钟内传递到 PNS 的推送通知。 这一延迟的出现原因有多种，但最常见的是因为 PNS 限制了应用程序。

### <a name="is-there-any-latency-guarantee"></a>是否有任何延迟保证？

推送通知的特性（由平台特定的外部 PNS 传递）导致没有延迟保证。 通常情况下，大部分推送通知会在几分钟内完成传递。

### <a name="what-do-i-need-to-consider-when-designing-a-solution-with-namespaces-and-notification-hubs"></a>设计包含命名空间和通知中心的解决方案时需要考虑哪些因素？

#### <a name="mobile-appenvironment"></a>移动应用/环境

* 为每个环境中的每个移动应用使用一个通知中心。
* 在多租户方案中，每个租户都应具有一个单独的中心。
* 切勿对生产环境和测试环境使用同一个通知中心。 这种做法可能导致发送通知时出现问题。 （Apple 提供沙盒和生产推送终结点，每个终结点具有单独的凭据。）
* 默认情况下，可以通过 Azure 门户或 Visual Studio 中的 Azure 集成组件将测试通知发送到已注册的设备。 阈值设置为从注册池中随机选取的 10 个设备。

> [!NOTE]
> 如果中心最初的配置使用 Apple 沙盒证书，后来又使用 Apple 生产证书重新配置，则原始设备令牌会失效。 无效的令牌会导致推送失败。 请将生产和测试环境分开，针对不同的环境使用不同的中心。

#### <a name="pns-credentials"></a>PNS 凭据

将移动应用注册到某个平台的开发人员门户后，会发送应用标识符和安全令牌。 应用后端将这些令牌提供给平台的 PNS，以便能够将推送通知发送到设备。 安全令牌的形式可以是证书（例如，在 Apple iOS 或 Windows Phone 中）或安全密钥。 必须在通知中心内配置安全令牌。 配置通常在通知中心级别完成，但是，也可以在多租户方案中的命名空间级别完成。

#### <a name="namespaces"></a>命名空间

命名空间可用于部署分组。 在多租户方案中，还可以使用命名空间来表示同一应用的所有租户的所有通知中心。

#### <a name="geo-distribution"></a>地理分布

在推送通知方案中，地理分布并非总是关键所在。 用于向设备传递推送通知的各个 PNS（例如 APNS）不会均匀分布。

如果有一个在全球范围内使用的应用程序，可以在全球不同的 Azure 区域使用通知中心服务在命名空间中创建中心。

> [!NOTE]
> 我们不建议采用这种做法，因为这会增大管理成本，尤其是注册成本。 仅当确实有需要时，才采用这种做法。

### <a name="should-i-do-registrations-from-the-app-backend-or-directly-through-client-devices"></a>我该从应用后端注册还是直接通过客户端设备注册？

在创建注册之前，如果必须对客户端进行身份验证，从应用后端进行注册很有用。 如果标记必须由应用后端根据应用逻辑创建或修改，这种注册方法也很有用。 有关详细信息，请转到[后端注册指南]和[后端注册指南 2] 页。

### <a name="what-is-the-push-notification-delivery-security-model"></a>什么是推送通知传递安全模型？

Azure 通知中心使用基于[共享访问签名](../storage/common/storage-dotnet-shared-access-signature-part-1.md)的安全模型。 可以在根命名空间级别或细粒度通知中心级别使用共享访问签名令牌。 可以使用不同的授权规则（例如，发送消息权限，或侦听通知权限）设置共享访问签名令牌。 有关详细信息，请参阅[通知中心安全模型]文档。

### <a name="how-should-i-handle-sensitive-payload-in-push-notifications"></a>如何处理推送通知中的敏感有效负载？

所有通知都由平台的 PNS 传递到目标设备。 将通知发送到 Azure 通知中心后，系统会对通知进行处理并将其传递到相应的 PNS。

从发送方到 Azure 通知中心、再到 PNS 的所有连接都使用 HTTPS。

> [!NOTE]
> Azure 通知中心不会以任何方式记录消息的有效负载。

若要发送敏感有效负载，我们建议使用安全推送模式。 发送方将带有消息标识符的 ping 通知传递到设备，其中不包含敏感有效负载。 当设备上的应用收到该有效负载后，可直接调用安全 API 来提取消息详细信息。 有关如何实现此模式的指导，请转到[通知中心安全推送教程]页。

## <a name="operations"></a>操作

### <a name="what-support-is-provided-for-disaster-recovery"></a>为灾难恢复提供哪种支持？

我们提供元数据灾难恢复范围（通知中心名称、连接字符串和其他重要信息）。 触发灾难恢复方案后，注册数据是通知中心基础结构中丢失的*唯一片段*。 需要实施某种解决方案，将此数据重新填充到恢复后的新中心。

1. 在另一个数据中心创建辅助通知中心。 我们建议一开始就创建一个辅助通知中心，以便在发生影响管理功能的灾难恢复事件时保护自己。 也可以在发生灾难恢复事件时创建一个辅助通知中心。

2. 使用主通知中心的注册信息填充辅助通知中心。 我们不建议尝试同时在两个中心维护注册信息，并在收到注册信息时在两个中心之间同步。 由于注册信息会在 PNS 端过期这一固有性质，这种做法通常效果不佳。 在收到 PNS 关于过期的或无效注册的反馈后，通知中心会清除这些注册信息。  

对于应用后端，我们提出以下两点建议：

* 使用一个应用后端来维护通知中心一端的一组给定注册信息。 然后，通知中心可以将这些信息批量插入辅助通知中心。
* 使用一个应用后端从主通知中心获取注册信息的常规转储作为备份。 然后，通知中心可以将这些信息批量插入辅助通知中心。

> [!NOTE]
> [注册信息导出/导入]文档中介绍了标准层中可用的注册信息导出/导入功能。

如果没有后端，当应用在目标设备上启动时，它们会在辅助通知中心执行新注册。 辅助通知中心最终将拥有所有已注册的活动设备。

在一段时间内，包含未打开的应用的设备将收不到通知。

### <a name="is-there-audit-log-capability"></a>是否有审核日志功能？

是的。 所有通知中心管理操作都会更新 [Azure 门户]中公开的 Azure 活动日志。 Azure 活动日志可使用户了解对订阅中的资源执行的操作。 通过活动日志，可确定对订阅中的资源进行的任何写入操作（PUT、POST、DELETE）的内容、执行者和时间。 还可以了解操作和其他相关属性的状态。 但是， 活动日志不包括读取 (GET) 操作。

## <a name="monitoring-and-troubleshooting"></a>监视和故障排除

### <a name="what-troubleshooting-capabilities-are-available"></a>故障排除功能有哪些？

Azure 通知中心提供多项可用于故障排除的功能，尤其是针对通知被删除的最常见情况。 有关详细信息，请参阅[通知中心故障排除]白皮书。

### <a name="what-telemetry-features-are-available"></a>遥测功能有哪些？

Azure 通知中心允许在 [Azure 门户]中查看遥测数据。 可以在[通知中心指标]页上找到有关可用指标的详细信息。




> [!NOTE]
> 通知成功仅意味着推送通知已传递到外部 PNS。 PNS 负责将通知传递到目标设备。 PNS 通常不会向第三方公开传递指标。  

[Azure 门户]: https://portal.azure.cn
[通知中心定价]: https://www.azure.cn/pricing/details/notification-hubs/
[Notification Hubs SLA]: https://www.azure.cn/support/legal/sla/
[通知中心 REST API]: https://msdn.microsoft.com/library/azure/dn530746.aspx
[通知中心入门教程]: ./notification-hubs-ios-apple-push-notification-apns-get-started.md
[Mobile Services Pricing]: https://www.azure.cn/pricing/details/mobile-services/
[后端注册指南]: https://msdn.microsoft.com/library/azure/dn743807.aspx
[后端注册指南 2]: https://msdn.microsoft.com/library/azure/dn530747.aspx
[通知中心安全模型]: https://msdn.microsoft.com/library/azure/dn495373.aspx
[通知中心安全推送教程]: ./notification-hubs-aspnet-backend-ios-push-apple-apns-secure-notification.md
[通知中心故障排除]: ./notification-hubs-push-notification-fixer.md
[Notification Hubs Metrics sample]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/FetchNHTelemetryInExcel
[注册信息导出/导入]: https://msdn.microsoft.com/library/dn790624.aspx
[Azure 门户]: https://portal.azure.cn
[complete samples]: https://github.com/Azure/azure-notificationhubs-samples
[应用服务定价]: https://www.azure.cn/pricing/details/app-service/
