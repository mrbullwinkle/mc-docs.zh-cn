---
title: 请求限制 - Azure 资源管理器 | Microsoft Azure
description: 介绍在达到订阅限制后如何对 Azure Resource Manager 请求进行限制。
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: conceptual
origin.date: 07/09/2019
ms.date: 07/22/2019
ms.author: v-yeche
ms.custom: seodec18
ms.openlocfilehash: 3ffa322e20742f04bac24d58f48070219412bf5f
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337391"
---
# <a name="throttling-resource-manager-requests"></a>限制 Resource Manager 请求数

对于每个 Azure 订阅和租户，资源管理器最多允许每小时 12,000 个读取请求和每小时 1,200 个写入请求。 这些限制的范围是发出请求的安全主体（用户或应用程序）和订阅 ID 或租户 ID。 如果请求来自多个安全主体，则在整个订阅或租户中实施的限制大于每小时 12,000 个和 1,200 个。

请求将应用到订阅或租户。 订阅请求是需要传递订阅 ID 的请求，例如在订阅中检索资源组。 租户请求不包括订阅 ID，例如，检索有效的 Azure 位置。

这些限制适用于每个 Azure 资源管理器实例。 每个 Azure 区域中有多个实例，Azure 资源管理器将部署到所有 Azure 区域。  因此，在实践中，限制实际上比上述限制要高得多，因为用户请求通常是由多个不同的实例提供服务。

如果应用程序或脚本达到了这些限制，则需对请求数进行限制。 本文说明如何在达到限制之前确定剩余的请求数，以及达到限制时如何做出响应。

达到限制时，会收到 HTTP 状态代码“429 请求过多”  。

<!--Not Available on [Throttle in Azure Resource Graph](../governance/resource-graph/overview.md#throttling)-->

## <a name="remaining-requests"></a>剩余请求数
可以通过检查响应标头确定剩余请求数。 读取请求在标头中返回一个值，表示剩余读取请求的数目。 写入请求包含的值表示剩余写入请求的数目。 下表说明了各种响应标头，用户可以检查其中是否存在这些值：

| 响应标头 | 说明 |
| --- | --- |
| x-ms-ratelimit-remaining-subscription-reads |划归到订阅的剩余读取数。 执行读取操作时返回此值。 |
| x-ms-ratelimit-remaining-subscription-writes |划归到订阅的剩余写入数。 执行写入操作时返回此值。 |
| x-ms-ratelimit-remaining-tenant-reads |租户范围的剩余读取数 |
| x-ms-ratelimit-remaining-tenant-writes |租户范围的剩余写入数 |
| x-ms-ratelimit-remaining-subscription-resource-requests |划归到订阅的剩余资源类型请求数。<br /><br />仅当服务重写了默认限制时，才返回此标头值。 Resource Manager 将累加此值而不是订阅读取/写入数。 |
| x-ms-ratelimit-remaining-subscription-resource-entities-read |划归到订阅的剩余资源类型集合请求数。<br /><br />仅当服务重写了默认限制时，才返回此标头值。 此值提供剩余集合请求数（列出资源）。 |
| x-ms-ratelimit-remaining-tenant-resource-requests |划归到租户的剩余资源类型请求数。<br /><br />仅当服务重写了默认限制时，才为租户级别的请求添加此标头。 Resource Manager 将累加此值而不是租户读取/写入数。 |
| x-ms-ratelimit-remaining-tenant-resource-entities-read |划归到租户的剩余资源类型集合请求数。<br /><br />仅当服务重写了默认限制时，才为租户级别的请求添加此标头。 |

## <a name="retrieving-the-header-values"></a>检索标头值
检索代码或脚本中的这些标头值与检索任何标头值没有什么不同。 

例如，在 **C#** 中，可使用以下代码从名为 **response** 的 **HttpWebResponse** 对象中检索标头值：

```cs
response.Headers.GetValues("x-ms-ratelimit-remaining-subscription-reads").GetValue(0)
```

在 **PowerShell**中，可以通过 Invoke-WebRequest 操作检索标头值。

```powershell
$r = Invoke-WebRequest -Uri https://management.chinacloudapi.cn/subscriptions/{guid}/resourcegroups?api-version=2016-09-01 -Method GET -Headers $authHeaders
$r.Headers["x-ms-ratelimit-remaining-subscription-reads"]
```

有关完整的 PowerShell 示例，请参阅[查看订阅的资源管理器限制](https://github.com/Microsoft/csa-misc-utils/tree/master/psh-GetArmLimitsViaAPI)。

若要查看剩余请求数以进行调试，可在 **PowerShell** cmdlet 中提供 **-Debug** 参数。

```powershell
Get-AzResourceGroup -Debug
```

这会返回许多值，包括以下响应值：

```powershell
DEBUG: ============================ HTTP RESPONSE ============================

Status Code:
OK

Headers:
Pragma                        : no-cache
x-ms-ratelimit-remaining-subscription-reads: 11999
```

若要获取写入限制，请使用写入操作： 

```powershell
New-AzResourceGroup -Name myresourcegroup -Location chinanorth -Debug
```

这会返回许多值，包括以下值：

```powershell
DEBUG: ============================ HTTP RESPONSE ============================

Status Code:
Created

Headers:
Pragma                        : no-cache
x-ms-ratelimit-remaining-subscription-writes: 1199
```

在 **Azure CLI**中，可以使用更详细的选项检索标头值。

```azurecli
az group list --verbose --debug
```

这会返回许多值，包括以下值：

```azurecli
msrest.http_logger : Response status: 200
msrest.http_logger : Response headers:
msrest.http_logger :     'Cache-Control': 'no-cache'
msrest.http_logger :     'Pragma': 'no-cache'
msrest.http_logger :     'Content-Type': 'application/json; charset=utf-8'
msrest.http_logger :     'Content-Encoding': 'gzip'
msrest.http_logger :     'Expires': '-1'
msrest.http_logger :     'Vary': 'Accept-Encoding'
msrest.http_logger :     'x-ms-ratelimit-remaining-subscription-reads': '11998'
```

若要获取写入限制，请使用写入操作： 

```azurecli
az group create -n myresourcegroup --location chinanorth --verbose --debug
```

这会返回许多值，包括以下值：

```azurecli
msrest.http_logger : Response status: 201
msrest.http_logger : Response headers:
msrest.http_logger :     'Cache-Control': 'no-cache'
msrest.http_logger :     'Pragma': 'no-cache'
msrest.http_logger :     'Content-Length': '163'
msrest.http_logger :     'Content-Type': 'application/json; charset=utf-8'
msrest.http_logger :     'Expires': '-1'
msrest.http_logger :     'x-ms-ratelimit-remaining-subscription-writes': '1199'
```

## <a name="waiting-before-sending-next-request"></a>在发送下一请求之前等待
达到请求限制时，Resource Manager 会在标头中返回 **429** HTTP 状态代码和 **Retry-After** 值。 **Retry-After** 值指定在发送下一个请求之前应用程序应该等待（或休眠）的秒数。 如果在尚未达到重试时间值的情况下发送请求，该请求不会得到处理，并会返回一个新的重试时间值。

## <a name="next-steps"></a>后续步骤

* 有关完整的 PowerShell 示例，请参阅[查看订阅的资源管理器限制](https://github.com/Microsoft/csa-misc-utils/tree/master/psh-GetArmLimitsViaAPI)。
* 有关限制和配额的详细信息，请参阅 [Azure 订阅和服务限制、配额和约束](../azure-subscription-service-limits.md)。
* 若要了解如何处理异步 REST 请求，请参阅[跟踪异步 Azure 操作](resource-manager-async-operations.md)。

<!--Update_Description: update meta properties, wording update -->