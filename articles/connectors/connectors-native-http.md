---
title: 从 Azure 逻辑应用连接到 HTTP 或 HTTPS 终结点
description: 使用 Azure 逻辑应用在自动化任务、流程和工作流中监视 HTTP 或 HTTPS 终结点
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: v-yiso
ms.reviewer: klam, LADocs
ms.assetid: e11c6b4d-65a5-4d2d-8e13-38150db09c0b
ms.topic: article
tags: connectors
origin.date: 07/05/2019
ms.date: 07/29/2019
ms.openlocfilehash: 667467f7d3920529e6ce2c1f2d7ece7ff2f82f55
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337490"
---
# <a name="call-http-or-https-endpoints-by-using-azure-logic-apps"></a>使用 Azure 逻辑应用调用 HTTP 或 HTTPS 终结点

借助 [Azure 逻辑应用](../logic-apps/logic-apps-overview.md)和内置的 HTTP 连接器，可以通过生成逻辑应用自动执行定期调用任何 HTTP 或 HTTPS 终结点的工作流。 例如，可按指定的计划检查该终结点，来监视网站的服务终结点。 当该终结点上发生特定的事件（例如，网站关闭）时，该事件会触发逻辑应用的工作流并运行指定的操作。

若要按定期计划检查或轮询某个终结点，可将 HTTP 触发器用作工作流中的第一个步骤。  每次检查时，该触发器会向该终结点发送调用或请求。  该终结点的响应确定了逻辑应用的工作流是否运行。 触发器将响应中的任何内容传递到逻辑应用中的操作。

可将 HTTP 操作用作工作流中的另一个步骤，用于调用所需的终结点。 该终结点的响应确定了工作流剩余操作的运行方式。

根据目标终结点的功能，HTTP 连接器支持传输层安全性 (TLS) 版本 1.0、1.1 和 1.2。 逻辑应用通过使用可能支持的最高版本与终结点协商。 因此，例如，如果终结点支持 1.2 版，则连接器首先使用 1.2 版。 否则，连接器将使用下一个受支持的最高版本。

## <a name="prerequisites"></a>先决条件

* Azure 订阅。 如果没有 Azure 订阅，请[注册一个免费 Azure 帐户](https://azure.microsoft.com/free/)。

* 要调用的目标终结点的 URL

* 有关[如何创建逻辑应用](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知识。 如果你不熟悉逻辑应用，请查看[什么是 Azure 逻辑应用？](../logic-apps/logic-apps-overview.md)

* 要从中调用目标终结点的逻辑应用。 若要从 HTTP 触发器开始，请[创建一个空白逻辑应用](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用 HTTP 操作，请使用所需的任何触发器启动逻辑应用。 此示例的第一步是使用 HTTP 触发器。

## <a name="add-an-http-trigger"></a>添加 HTTP 触发器

此内置触发器对终结点的指定 URL 发出 HTTP 调用，并返回响应。

1. 登录到 [Azure 门户](https://portal.azure.com)。 在逻辑应用设计器中打开空白逻辑应用。

1. 在设计器的搜索框中，输入“http”作为筛选器。 在“触发器”列表中，选择“HTTP”触发器。  

   ![选择 HTTP 触发器](./media/connectors-native-http/select-http-trigger.png)

   此示例将触发器重命名为“HTTP trigger”，使步骤的名称更具描述性。 此外，该示例稍后会添加 HTTP 操作，因此，两个名称必须唯一。

1. 提供要包含在目标终结点调用中的 [HTTP 触发器参数](../logic-apps/logic-apps-workflow-actions-triggers.md##http-trigger)的值。 设置重复周期，以确定触发器检查目标终结点的频率。

   ![输入 HTTP 触发器参数](./media/connectors-native-http/http-trigger-parameters.png)

   有关可用于 HTTP 的身份验证类型的详细信息，请参阅[对 HTTP 触发器和操作进行身份验证](../logic-apps/logic-apps-workflow-actions-triggers.md#connector-authentication)。

1. 若要添加其他可用参数，请打开“添加新参数”列表，并选择所需的参数。 

1. 继续使用触发器激发时运行的操作生成逻辑应用的工作流。

1. 完成后，请记得保存逻辑应用。 在设计器工具栏上选择“保存”。 

## <a name="add-an-http-action"></a>添加 HTTP 操作

此内置操作对终结点的指定 URL 发出 HTTP 调用，并返回响应。

1. 登录到 [Azure 门户](https://portal.azure.com)。 在逻辑应用设计器中打开逻辑应用。

   此示例的第一步是使用 HTTP 触发器。

1. 在要添加 HTTP 操作的步骤下，选择“新建步骤”。 

   若要在步骤之间添加操作，请将鼠标指针移到步骤之间的箭头上。 选择出现的加号 ( **+** )，然后选择“添加操作”。 

1. 在设计器的搜索框中，输入“http”作为筛选器。 在“操作”列表中，选择“HTTP”操作。  

   ![选择“HTTP”操作](./media/connectors-native-http/select-http-action.png)

   此示例将操作重命名为“HTTP action”，使步骤的名称更具描述性。

1. 提供要包含在目标终结点调用中的 [HTTP 操作参数](../logic-apps/logic-apps-workflow-actions-triggers.md##http-action)的值。

   ![输入 HTTP 操作参数](./media/connectors-native-http/http-action-parameters.png)

   有关可用于 HTTP 的身份验证类型的详细信息，请参阅[对 HTTP 触发器和操作进行身份验证](../logic-apps/logic-apps-workflow-actions-triggers.md#connector-authentication)。

1. 若要添加其他可用参数，请打开“添加新参数”列表，并选择所需的参数。 

1. 完成后，请记得保存逻辑应用。 在设计器工具栏上选择“保存”。 

## <a name="connector-reference"></a>连接器参考

有关触发器和操作参数的详细信息，请参阅以下部分：

* [HTTP 触发器参数](../logic-apps/logic-apps-workflow-actions-triggers.md##http-trigger)
* [HTTP 操作参数](../logic-apps/logic-apps-workflow-actions-triggers.md##http-action)

### <a name="output-details"></a>输出详细信息

下面是有关 HTTP 触发器或操作的输出的详细信息，输出中将返回以下信息：

| 属性名称 | 类型 | 说明 |
|---------------|------|-------------|
| headers | object | 请求中的标头 |
| body | object | JSON 对象 | 包含请求中正文内容的对象 |
| 状态代码 | int | 请求中的状态代码 |
|||

| 状态代码 | 说明 |
|-------------|-------------|
| 200 | OK |
| 202 | 已接受 |
| 400 | 错误的请求 |
| 401 | 未授权 |
| 403 | 禁止 |
| 404 | 未找到 |
| 500 | 内部服务器错误。 发生未知错误。 |
|||

## <a name="next-steps"></a>后续步骤

* 了解其他[逻辑应用连接器](../connectors/apis-list.md)
