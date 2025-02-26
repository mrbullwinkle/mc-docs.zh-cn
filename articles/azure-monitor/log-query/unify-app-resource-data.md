---
title: 统一多个 Azure Monitor Application Insights 资源 | Azure Docs
description: 本文详细介绍了如何在 Azure Monitor 日志中使用函数来查询多个 Application Insights 资源并直观呈现这些数据。
services: azure-monitor
documentationcenter: ''
author: lingliw
manager: digimobile
editor: ''
ms.service: azure-monitor
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 02/19/2019
ms.author: v-lingwu
ms.openlocfilehash: 3d342377a63b94ea044f24fe4812dd5552dce031
ms.sourcegitcommit: e78670855b207c6084997f747ad8e8c3afa3518b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68513827"
---
# <a name="unify-multiple-azure-monitor-application-insights-resources"></a>统一多个 Azure Monitor Application Insights 资源 
本文介绍如何在一个位置查询和查看所有 Application Insights 应用程序日志数据（即使这些数据位于不同 Azure 订阅），可作为弃用 Application Insights 连接器的替换方式。 可以在单个查询中包含的资源（Application Insights 资源）数量限制为 100。  

## <a name="recommended-approach-to-query-multiple-application-insights-resources"></a>用于查询多个 Application Insights 资源的建议做法 
在查询中列出多个 Application Insights 资源可能很繁琐且难以维护。 可以改为利用函数将查询逻辑从应用程序范围中分离出来。  

本示例演示了如何监视多个 Application Insights 资源，并按应用程序名称显示失败请求的计数。 开始之前，请在该连接到 Application Insights 资源的工作区中运行查询，获取已连接应用程序的列表： 

```
ApplicationInsights
| summarize by ApplicationName
```

使用 union 运算符和应用程序列表创建函数，然后在工作区中使用别名 *applicationsScoping* 将该查询保存为函数。  

```
union withsource=SourceApp 
app('Contoso-app1').requests,  
app('Contoso-app2').requests, 
app('Contoso-app3').requests, 
app('Contoso-app4').requests, 
app('Contoso-app5').requests 
| parse SourceApp with * "('" applicationName "')" *  
```

>[!NOTE]
>可以通过导航到工作区中的查询资源管理器，然后选择要编辑的函数并保存，或使用 `SavedSearch` PowerShell cmdlet 来随时修改列出的应用程序。 `withsource= SourceApp` 命令可向结果添加用于指定发送日志的应用程序的列。 
>
>虽然是在工作区中执行查询，但该查询使用 Application Insights 架构，因为 applicationsScoping 函数会返回 Application Insights 数据结构。 
>
>本示例中的 parse 运算符为可选项，可从 SourceApp 属性中提取应用程序名称。 

现在便可以在跨资源查询中使用 applicationsScoping 函数：  

```
applicationsScoping 
| where timestamp > ago(12h)
| where success == 'False'
| parse SourceApp with * '(' applicationName ')' * 
| summarize count() by applicationName, bin(timestamp, 1h) 
| render timechart
```

函数别名返回来自所有已定义应用程序的请求的并集。 然后，查询筛选失败的请求，并按应用程序显示趋势。

![跨查询结果示例](media/unify-app-resource-data/app-insights-query-results.png)

## <a name="query-across-application-insights-resources-and-workspace-data"></a>跨 Application Insights 资源和工作区数据的查询 
停止连接器时，如果需要在 Application Insights 数据保留（90 天）调整的时间范围内执行查询，则需于中期在工作区和 Application Insights 资源上执行[跨资源查询](../../azure-monitor/log-query/cross-workspace-query.md)。 应用程序数据按照上述新的 Application Insights 数据保留累积之前均是如此。 由于 Application Insights 和工作区中的架构不同，因此查询需要一些操作。 请参阅本节后面的表格，其中突出显示了架构差异。 


例如，如果连接器于 2018 年 11 月 1 日停止工作，则在 Application Insights 资源和工作区中的应用程序数据之间查询日志时，查询的构造方式类似于以下示例：

```
applicationsScoping //this brings data from Application Insights resources 
| where timestamp between (datetime("2018-11-01") .. now()) 
| where success == 'False' 
| where duration > 1000 
| union ( 
    ApplicationInsights //this is Application Insights data in Log Analytics workspace 
    | where TimeGenerated < (datetime("2018-12-01") 
    | where RequestSuccess == 'False' 
    | where RequestDuration > 1000 
    | extend duration = RequestDuration //align to Application Insights schema 
    | extend timestamp = TimeGenerated //align to Application Insights schema 
    | extend name = RequestName //align to Application Insights schema 
    | extend resultCode = ResponseCode //align to Application Insights schema 
    | project-away RequestDuration , RequestName , ResponseCode , TimeGenerated 
) 
| project timestamp , duration , name , resultCode 
```

## <a name="application-insights-and-log-analytics-workspace-schema-differences"></a>Application Insights 和 Log Analytics 工作区的架构差异
下表显示了 Log Analytics 和 Application Insights 之间的架构差异。  

| Log Analytics 工作区属性| Application Insights 资源属性|
|------------|------------| 
| AnonUserId | user_id|
| ApplicationId | appId|
| ApplicationName | appName|
| ApplicationTypeVersion | application_Version |
| AvailabilityCount | itemCount |
| AvailabilityDuration | duration |
| AvailabilityMessage | message |
| AvailabilityRunLocation | location |
| AvailabilityTestId | id |
| AvailabilityTestName | name |
| AvailabilityTimestamp | timestamp |
| 浏览器 | client_browser |
| 城市 | client_city |
| ClientIP | client_IP |
| Computer | cloud_RoleInstance | 
| 国家/地区 | client_CountryOrRegion | 
| CustomEventCount | itemCount | 
| CustomEventDimensions | customDimensions |
| CustomEventName | name | 
| DeviceModel | client_Model | 
| DeviceType | client_type | 
| ExceptionCount | itemCount | 
| ExceptionHandledAt | handledAt |
| ExceptionMessage | message | 
| ExceptionType | type |
| OperationID | operation_Id |
| OperationName | operation_Name | 
| 操作系统 | client_OS | 
| PageViewCount | itemCount |
| PageViewDuration | duration | 
| PageViewName | name | 
| ParentOperationID | operation_Id | 
| RequestCount | itemCount | 
| RequestDuration | duration | 
| RequestID | id | 
| RequestName | name | 
| RequestSuccess | success | 
| ResponseCode | resultCode | 
| 角色 | cloud_RoleName |
| RoleInstance | cloud_RoleInstance |
| SessionId | session_Id | 
| SourceSystem | operation_SyntheticSource |
| TelemetryType | type |
| URL | url |
| UserAccountId | user_AccountId |

## <a name="next-steps"></a>后续步骤

使用[日志搜索](../../azure-monitor/log-query/log-query-overview.md)可以查看 Application Insights 应用的详细信息。




