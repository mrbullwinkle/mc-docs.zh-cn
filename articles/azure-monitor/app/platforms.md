---
title: Application Insights：语言、平台和集成 | Azure Docs
description: 适用于 Application Insights 的语言、平台和集成
services: application-insights
documentationcenter: ''
author: lingliw
manager: digimobile
ms.assetid: 974db106-54ff-4318-9f8b-f7b3a869e536
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 6/4/2019
ms.reviewer: olegan
ms.author: v-lingwu
ms.openlocfilehash: 30f44d8b1694aa8ccf8ac853cee6e327e0cd5d71
ms.sourcegitcommit: e78670855b207c6084997f747ad8e8c3afa3518b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68513877"
---
# <a name="developer-analytics-languages-platforms-and-integrations"></a>开发人员分析：语言、平台和集成
这些项是我们听说过的 [Application Insights](../../azure-monitor/app/app-insights-overview.md) 的实现，包括第三方的一些实现。

## <a name="languages---officially-supported-by-application-insights-team"></a>语言 - 由 Application Insights 团队提供正式支持
* [C#|VB (.NET)](../../azure-monitor/app/asp-net.md)
* [Java](../../azure-monitor/app/java-get-started.md)
* [JavaScript 网页](../../azure-monitor/app/javascript.md)
* [Node.JS](../../azure-monitor/app/nodejs.md)

## <a name="languages---community-supported"></a>语言 - 社区支持
* [F#](https://safe-stack.github.io/docs/template-azure-ai/)
* [Go](https://github.com/Microsoft/ApplicationInsights-go)
* [PHP](https://github.com/Microsoft/ApplicationInsights-PHP)
* [Python](https://pypi.python.org/pypi/applicationinsights/0.1.0)
* [Ruby](https://rubygems.org/gems/application_insights)
* [任何其他项目](#projects)

## <a name="platforms-and-frameworks"></a>平台和框架
* [ASP.NET](../../azure-monitor/app/asp-net.md)
* [ASP.NET - 适用于已处于活动状态的应用](../../azure-monitor/app/monitor-performance-live-website-now.md)
* [ASP.NET Core](../../azure-monitor/app/asp-net-core.md)
* [Android](../../azure-monitor/learn/mobile-center-quickstart.md) (App Center)
* [Android](https://github.com/Microsoft/ApplicationInsights-Android) (App Center)
* [Angular](https://github.com/MarkPieszak/angular-application-insights)
* [Azure 应用服务](../../azure-monitor/app/azure-web-apps.md)
* [Azure 云服务](../../azure-monitor/app/cloudservices.md)&#151;包括 Web 角色和辅助角色
* [Azure Functions](/azure-functions/functions-monitoring)
* [Docker](../../azure-monitor/app/docker.md)
* [Glimpse](https://azure.microsoft.com/blog/glimpse-application-insights/)
* [iOS](../../azure-monitor/learn/mobile-center-quickstart.md) (App Center)
* [Ionic](https://github.com/SoftwarePioniere/ionic-application-insights)
* [iOS](https://github.com/Microsoft/ApplicationInsights-iOS) (App Center)
* [Java EE](../../azure-monitor/app/java-get-started.md)
* [Node.JS](https://www.npmjs.com/package/applicationinsights)
* [OSX](https://github.com/Microsoft/ApplicationInsights-OSX)
* [安全堆栈](https://safe-stack.github.io/docs/template-azure-ai/)
* [Spring](https://joe.blog.freemansoft.com/2015/12/enabling-microsoft-application-insight.html)
* [通用 Windows 应用](../../azure-monitor/learn/mobile-center-quickstart.md) (App Center)
* [WCF](https://github.com/Microsoft/ApplicationInsights-SDK-Labs/blob/master/WCF/readme.md)
* [Windows 桌面应用程序、服务和辅助角色](../../azure-monitor/app/windows-desktop.md)
* [任何其他项目](#projects)

## <a name="logging-frameworks"></a>记录框架
* [ILogger](/azure-monitor/app/ilogger)
* [Log4Net、NLog 或 System.Diagnostics.Trace](../../azure-monitor/app/asp-net-trace-logs.md)
* [Java、Log4J 或 Logback](../../azure-monitor/app/java-trace-logs.md)
* [语义式日志记录 (SLAB)](https://github.com/fidmor89/SLAB_AppInsights) - 与[语义式日志记录应用程序块](https://msdn.microsoft.com/library/dn440729.aspx)集成
* [基于云的负载测试](https://blogs.msdn.com/b/visualstudioalm/archive/2015/07/30/getting-application-insights-counters-with-cloud-based-load-testing.aspx)
* [LogStash 插件](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-output-applicationinsights)
* [Logary](https://www.nuget.org/packages/Logary.Targets.AppInsights/)
* [Logrus](https://github.com/jjcollinge/logrus-appinsights)
* [Azure Monitor](https://blogs.technet.microsoft.com/msoms/2016/09/26/application-insights-connector-in-oms/)

## <a name="content-management-systems"></a>内容管理系统
* [Concrete](https://github.com/fidmor89/appInsights-Concrete)
* [Drupal](https://github.com/fidmor89/AppInsights-Drupal)
* [Joomla](https://github.com/fidmor89/AppInsights-Joomla)
* [Orchard](https://azure.microsoft.com/blog/integrating-application-insights-into-a-modular-cms-and-a-multi-tenant-public-saas/preview/)
* [SharePoint](../../azure-monitor/app/sharepoint.md)
* [WordPress](https://wordpress.org/plugins/application-insights/)


<a name="projects"></a>
##  <a name="build-your-own-sdk"></a>构建自己的 SDK
如果尚不存在适用于你的语言或平台的 SDK，也许你想要生成一个？ 查看 [GitHub 上的 Application Insights SDK 项目](https://github.com/Microsoft/AppInsights-Home)中列出的现有 SDK 的代码。




