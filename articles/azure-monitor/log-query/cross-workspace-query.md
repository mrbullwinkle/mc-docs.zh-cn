---
title: 使用 Azure Monitor 跨资源进行查询 | Azure Docs
description: 本文介绍了如何在订阅中跨多个工作区以及从特定的 App Insights 应用查询资源。
services: log-analytics
documentationcenter: ''
author: lingliw
manager: digimobile
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 06/05/2019
ms.author: v-lingwu
ms.openlocfilehash: 168a37238215b42160a88041ff967cc391a9bd99
ms.sourcegitcommit: e78670855b207c6084997f747ad8e8c3afa3518b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68513833"
---
# <a name="perform-cross-resource-log-queries-in-azure-monitor"></a>在 Azure Monitor 中执行跨资源日志查询  

以前，使用 Azure Monitor 只能分析来自当前工作区内的数据，这限制了跨订阅中定义的多个工作区查询数据的能力。  此外，之前只能直接在 Application Insights 中或从 Visual Studio 中使用 Application Insights 搜索通过基于 web 的应用程序收集的遥测数据项。 这还使得难以采用本机方式将操作数据和应用程序数据一起分析。

现在不但可以跨多个 Log Analytics 工作区进行查询，而且还可以查询同一资源组、另一资源组或另一订阅中特定 Application Insights 应用的数据。 这可以提供数据的系统级视图。 你只能在 [Log Analytics](log-query-overview.md) 中执行这些类型的查询。

## <a name="cross-resource-query-limits"></a>跨资源查询限制 

* 可以在单个查询中包含的 Application Insights 资源和 Log Analytics 工作区的数量限制为 100。
* 视图设计器不支持跨资源查询。 可以在 Log Analytics 中创作一个查询，将其固定到 Azure 仪表板，以[将日志查询可视化](../learn/tutorial-logs-dashboards.md)。 


## <a name="querying-across-log-analytics-workspaces-and-from-application-insights"></a>跨 Log Analytics 工作区以及从 Application Insights 进行查询
若要在查询中引用另一个工作区，请使用 [*workspace*](/azure-monitor/log-query/workspace-expression) 标识符，对于 Application Insights 中的应用，请使用 [*app*](/azure-monitor/log-query/app-expression) 标识符。  

### <a name="identifying-workspace-resources"></a>标识工作区资源
以下示例演示了跨 Log Analytics 工作区进行查询以从名为 *contosoretail-it* 的工作区的 Update 表中返回记录的汇总计数。 

可以通过以下任一方式来标识工作区：

* 资源名称 - 用户可读的工作区名称，有时称为“组件名称”  。 

    `workspace("contosoretail-it").Update | count`
 
    >[!NOTE]
    >按名称标识工作区会假设它在所有可访问订阅中是唯一的。 如果具有多个采用指定名称的应用程序，查询将因多义性而失败。 在这种情况下，必须使用其他标识符之一。

* 限定名称 - 工作区的“全名”，由订阅名称、资源组和组件名称组成，并采用以下格式：*subscriptionName/resourceGroup/componentName*。 

    `workspace('contoso/contosoretail/contosoretail-it').Update | count`

    >[!NOTE]
    >因为 Azure 订阅名称不唯一，所以此标识符可能不明确。 
    >

* 工作区 ID - 工作区 ID 是分配给每个工作区的唯一不可变标识符，表示为全局唯一标识符 (GUID)。

    `workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update | count`

* Azure 资源 ID - Azure 定义的工作区唯一标识。 当资源名称不明确时需使用资源 ID。  对于工作区，此格式为：/subscriptions/subscriptionId/resourcegroups/resourceGroup/providers/microsoft.OperationalInsights/workspaces/componentName  。  

    例如：
    ``` 
    workspace("/subscriptions/e427519-5645-8x4e-1v67-3b84b59a1985/resourcegroups/ContosoAzureHQ/providers/Microsoft.OperationalInsights/workspaces/contosoretail-it").Update | count
    ```

### <a name="identifying-an-application"></a>标识应用程序
以下示例返回针对 Application Insights 中名为 *fabrikamapp* 的应用发出的请求的汇总计数。 

可以使用 *app(Identifier)* 表达式标识 Application Insights 中的应用程序。  *Identifier* 参数使用下列项之一来指定应用：

* 资源名称 - 人类可读的应用名称，有时称为“组件名称”  。  

    `app("fabrikamapp")`

* 限定名称 - 应用的“全名”，由订阅名称、资源组和组件名称组成，并采用以下格式：*subscriptionName/resourceGroup/componentName*。 

    `app("AI-Prototype/Fabrikam/fabrikamapp").requests | count`

     >[!NOTE]
    >因为 Azure 订阅名称不唯一，所以此标识符可能不明确。 
    >

* ID - 应用程序的应用 GUID。

    `app("b459b4f6-912x-46d5-9cb1-b43069212ab4").requests | count`

* Azure 资源 ID - Azure 定义的应用唯一标识。 当资源名称不明确时需使用资源 ID。 格式为： */subscriptions/subscriptionId/resourcegroups/resourceGroup/providers/microsoft.OperationalInsights/components/componentName*。  

    例如：
    ```
    app("/subscriptions/b459b4f6-912x-46d5-9cb1-b43069212ab4/resourcegroups/Fabrikam/providers/microsoft.insights/components/fabrikamapp").requests | count
    ```

### <a name="performing-a-query-across-multiple-resources"></a>跨多个资源执行查询
可以从任何资源实例查询多个资源，这些资源实例可以是工作区和应用程序的组合。
    
跨两个工作区进行查询的示例：    

```
union Update, workspace("contosoretail-it").Update, workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update
| where TimeGenerated >= ago(1h)
| where UpdateState == "Needed"
| summarize dcount(Computer) by Classification
```

## <a name="using-cross-resource-query-for-multiple-resources"></a>针对多个资源使用跨资源查询
使用跨资源查询来关联来自多个 Log Analytics 工作区和 Application Insights 资源的数据时，查询可能变得复杂且难以维护。 应利用 [Azure Monitor 日志查询中的函数](functions.md)将查询逻辑与查询资源的范围分开，这样可以简化查询结构。 以下示例演示了如何监视多个 Application Insights 资源，并按应用程序名称显示失败请求的计数。 

创建如下所示的引用 Application Insights 资源范围的查询。 `withsource= SourceApp` 命令可添加用于指定发送日志的应用程序名称的列。 使用别名 applicationsScoping  [ 将查询另存为函数 ](functions.md#create-a-function)。

```Kusto
// crossResource function that scopes my Application Insights resources
union withsource= SourceApp
app('Contoso-app1').requests, 
app('Contoso-app2').requests,
app('Contoso-app3').requests,
app('Contoso-app4').requests,
app('Contoso-app5').requests
```



现可在跨资源查询中[使用此函数](../../azure-monitor/log-query/functions.md#use-a-function)，如下所示。 函数别名 applicationsScoping 返回来自所有已定义应用程序的请求表的并集  。 然后，查询筛选失败的请求，并按应用程序显示趋势。 在此示例中，分析运算符是可选的  。 该运算符从 SourceApp 属性中提取应用程序名称  。

```Kusto
applicationsScoping 
| where timestamp > ago(12h)
| where success == 'False'
| parse SourceApp with * '(' applicationName ')' * 
| summarize count() by applicationName, bin(timestamp, 1h) 
| render timechart
```
![时间表](media/cross-workspace-query/chart.png)

## <a name="next-steps"></a>后续步骤

- 查看[在 Azure Monitor 中分析日志数据](log-query-overview.md)来大致了解日志查询以及 Azure Monitor 日志数据是如何构造的。
- 查看 [Azure Monitor 日志查询](query-language.md)来了解适用于 Azure Monitor 日志查询的所有资源。




