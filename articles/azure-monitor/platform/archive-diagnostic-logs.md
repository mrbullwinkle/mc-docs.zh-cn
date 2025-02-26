---
title: 存档 Azure 诊断日志
description: 了解如何存档 Azure 诊断日志，将其长期保留在存储帐户中。
author: lingliw
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 04/12/2019
ms.author: v-lingwu
ms.subservice: logs
ms.openlocfilehash: 75ead1110eb55dfc2398f75cc9faff055e3ce33c
ms.sourcegitcommit: e78670855b207c6084997f747ad8e8c3afa3518b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68514014"
---
# <a name="archive-azure-diagnostic-logs"></a>存档 Azure 诊断日志

本文介绍如何使用 Azure 门户、PowerShell Cmdlet、CLI 或 REST API 将 [Azure 诊断日志](diagnostic-logs-overview.md)存档到存储帐户中。 如果要使用可选保留策略来保留诊断日志以便进行审核、静态分析或备份，那么此选项十分有用。 只要配置设置的用户同时拥有两个订阅的相应 RBAC 访问权限，存储帐户就不必位于发出日志的资源所在的订阅中。

## <a name="prerequisites"></a>先决条件

在开始之前，需要[创建存储帐户](../../storage/common/storage-quickstart-create-account.md)，以便将诊断日志存档到其中。 强烈建议用户不要使用其中存储了其他非监视数据的现有存储帐户，以便更好地控制监视数据所需的访问权限。 但是，如果还要将活动日志和诊断指标存档到存储帐户，则也可将该存储帐户用于诊断日志，使得所有监视数据都位于一个中心位置。

## <a name="diagnostic-settings"></a>诊断设置

若要使用下述任意方法存档诊断日志，可针对特定资源设置“诊断设置”  。 资源的诊断设置定义发送到目标的日志和指标数据类别（存储帐户、事件中心命名空间或 Log Analytics 工作区）。 此外，它还定义存储在存储帐户中的每个日志类别和指标数据的事件保留策略（需保留的天数）。 如果将保留策略设置为零，则会无限期（即永久）存储该日志类别的事件。 如果不需要无限期存储，可将保留策略设置为 1 到 2147483647 之间的任意天数。 [单击此处详细了解诊断设置](../../azure-monitor/platform/diagnostic-logs-overview.md#diagnostic-settings)。 保留策略按天应用，因此在一天结束时 (UTC)，会删除当天已超过保留策略期限的日志。 例如，假设保留策略的期限为一天，则在今天开始时，会删除前天的日志。 删除过程从午夜 (UTC) 开始，但请注意，可能最多需要 24 小时才能将日志从存储帐户中删除。 

> [!NOTE]
> 当前不支持通过诊断设置发送多维指标。 多维指标将按平展后的单维指标导出，并跨维值聚合。
>
> 例如：  可以基于每个队列级别浏览和绘制事件中心上的“传入消息”指标。 但是，当通过诊断设置导出时，该指标将表示为事件中心的所有队列中的所有传入消息。
>
>

## <a name="archive-diagnostic-logs-using-the-portal"></a>使用门户存档诊断日志

1. 在门户中，导航到 Azure Monitor 并单击“诊断设置” 

    ![Azure Monitor 的“监视”部分](media/archive-diagnostic-logs/diagnostic-settings-blade.png)

2. （可选）按资源组或资源类型筛选列表，并单击要为其设置诊断设置的资源。

3. 如果选定的资源上不存在任何设置，系统会提示创建设置。 单击“启用诊断”。

   ![添加诊断设置 - 没有现有的设置](media/archive-diagnostic-logs/diagnostic-settings-none.png)

   如果资源上有现有的设置，则会看到已在此资源上配置的设置列表。 单击“添加诊断设置”。

   ![添加诊断设置 - 现有的设置](media/archive-diagnostic-logs/diagnostic-settings-multiple.png)

3. 命名设置并选中与“导出到存储帐户”  对应的框，然后选择存储帐户。 （可选）使用“保留期(天)”  滑块设置这些日志的保留天数。 如果保留期为 0 天，则会无限期存储日志。

   ![添加诊断设置 - 现有的设置](media/archive-diagnostic-logs/diagnostic-settings-configure.png)

4. 单击“保存”  。

稍后，新设置就会显示在此资源的设置列表中，只要生成新的事件数据，就会立即将诊断日志存档到该存储帐户。

## <a name="archive-diagnostic-logs-via-azure-powershell"></a>通过 Azure PowerShell 存档诊断日志

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

```
Set-AzDiagnosticSetting -ResourceId /subscriptions/s1id1234-5679-0123-4567-890123456789/resourceGroups/testresourcegroup/providers/Microsoft.Network/networkSecurityGroups/testnsg -StorageAccountId /subscriptions/s1id1234-5679-0123-4567-890123456789/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -Category networksecuritygroupevent,networksecuritygrouprulecounter -Enabled $true -RetentionEnabled $true -RetentionInDays 90
```

| 属性 | 必须 | 说明 |
| --- | --- | --- |
| ResourceId |是 |要设置诊断设置的资源的资源 ID。 |
| StorageAccountId |否 |诊断日志应保存到的存储帐户的资源 ID。 |
| Category |否 |要启用的日志类别的逗号分隔列表。 |
| Enabled |是 |一个布尔值，表示此资源是启用还是禁用了诊断。 |
| RetentionEnabled |否 |一个布尔值，表示此资源是否启用了保留策略。 |
| RetentionInDays |否 |事件的保留天数，介于 1 到 365 之间。 值为零时，将无限期存储日志。 |

## <a name="archive-diagnostic-logs-via-the-azure-cli"></a>通过 Azure CLI 存档诊断日志

```azurecli
az monitor diagnostic-settings create --name <diagnostic name> \
    --storage-account <name or ID of storage account> \
    --resource <target resource object ID> \
    --resource-group <storage account resource group> \
    --logs '[
    {
        "category": <category name>,
        "enabled": true,
        "retentionPolicy": {
            "days": <# days to retain>,
            "enabled": true
        }
    }]'
```

可以通过将字典添加到以 `--logs` 参数传递的 JSON 数组来将其他类别添加到诊断日志。

仅当 `--storage-account` 不是对象 ID 时，才需要 `--resource-group` 参数。 有关将诊断日志存档到存储的完整文档，请参阅 [CLI 命令参考](https://docs.azure.cn/zh-cn/cli/monitor/diagnostic-settings?view=azure-cli-latest#az-monitor-diagnostic-settings-create)。

## <a name="archive-diagnostic-logs-via-the-rest-api"></a>通过 REST API 存档诊断日志

若要了解如何使用 Azure 监视器 REST API 设置诊断设置，请[参阅此文档](https://docs.microsoft.com/rest/api/monitor/diagnosticsettings)。

## <a name="schema-of-diagnostic-logs-in-the-storage-account"></a>存储帐户中诊断日志的架构

设置存档以后，一旦在某个已启用的日志类别中出现事件，就会在存储帐户中创建存储容器。 容器中的 blob 在活动日志和诊断日志中采用相同的命名约定，如下所示：

```
insights-logs-{log category name}/resourceId=/SUBSCRIPTIONS/{subscription ID}/RESOURCEGROUPS/{resource group name}/PROVIDERS/{resource provider name}/{resource type}/{resource name}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json
```

例如，blob 名称可以为：

```
insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUP/TESTNSG/y=2016/m=08/d=22/h=18/m=00/PT1H.json
```

每个 PT1H.json blob 都包含一个 JSON blob，其中的事件为在 blob URL 中指定的小时（例如 h=12）内发生的。 在当前的小时内发生的事件将附加到 PT1H.json 文件。 分钟值始终为 00 (m=00)，因为诊断日志事件按小时细分成单个 blob。

在 PT1H.json 文件中，每个事件都按以下格式存储在“records”数组中：

``` JSON
{
    "records": [
        {
            "time": "2016-07-01T00:00:37.2040000Z",
            "systemId": "46cdbb41-cb9c-4f3d-a5b4-1d458d827ff1",
            "category": "NetworkSecurityGroupRuleCounter",
            "resourceId": "/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/TESTNSG",
            "operationName": "NetworkSecurityGroupCounters",
            "properties": {
                "vnetResourceGuid": "{12345678-9012-3456-7890-123456789012}",
                "subnetPrefix": "10.3.0.0/24",
                "macAddress": "000123456789",
                "ruleName": "/subscriptions/ s1id1234-5679-0123-4567-890123456789/resourceGroups/testresourcegroup/providers/Microsoft.Network/networkSecurityGroups/testnsg/securityRules/default-allow-rdp",
                "direction": "In",
                "type": "allow",
                "matchedConnections": 1988
            }
        }
    ]
}
```

| 元素名称 | 说明 |
| --- | --- |
| time |处理与事件对应的请求的 Azure 服务生成事件时的时间戳。 |
| ResourceId |受影响资源的资源 ID。 |
| operationName |操作的名称。 |
| category |事件的日志类别。 |
| properties |`<Key, Value>` 对集合（即字典），描述事件的详细信息。 |

> [!NOTE]
> 这些属性的属性和使用情况各不相同，具体取决于资源。

## <a name="next-steps"></a>后续步骤

* [下载 blob 进行分析](../../storage/blobs/storage-quickstart-blobs-dotnet.md)
* [将诊断日志流式传输到事件中心命名空间](../../azure-monitor/platform/diagnostic-logs-stream-event-hubs.md)
* [详细了解诊断日志](../../azure-monitor/platform/diagnostic-logs-overview.md)





