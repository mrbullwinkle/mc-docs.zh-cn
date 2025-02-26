---
title: 在 Azure 存储中存储和查看诊断数据
description: 将 Azure 诊断数据传输到 Azure 存储并查看
services: azure-monitor
author: lingliw
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 01/21/2019
ms.author: v-lingwu
ms.component: diagnostic-extension
ms.openlocfilehash: f98c28fd14a9f99831be1eb366cbd398b8852d61
ms.sourcegitcommit: e78670855b207c6084997f747ad8e8c3afa3518b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68513798"
---
# <a name="store-and-view-diagnostic-data-in-azure-storage"></a>在 Azure 存储中存储和查看诊断数据
诊断数据不会永久存储，除非将其传输到世纪互联 Azure 存储模拟器或 Azure 存储。 如果诊断数据位于存储中，可以使用提供的工具之一进行查看。

## <a name="specify-a-storage-account"></a>指定存储帐户
指定要在 ServiceConfiguration.cscfg 文件中使用的存储帐户。 帐户信息被定义为配置设置中的连接字符串。 以下示例显示的是在 Visual Studio 中针对新的云服务项目创建的默认连接字符串：

```
    <ConfigurationSettings>
       <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
    </ConfigurationSettings>
```

可以更改此连接字符串，为 Azure 存储帐户提供帐户信息。

根据收集的诊断数据类型，Azure 诊断使用 Blob 服务或表服务。 下表显示了保留的数据源及其格式。

| 数据源 | 存储格式 |
| --- | --- |
| Azure 日志 |表 |
| IIS 7.0 日志 |Blob |
| Azure 诊断基础结构日志 |表 |
| 失败请求跟踪日志 |Blob |
| Windows 事件日志 |表 |
| 性能计数器 |表 |
| 故障转储 |Blob |
| 自定义错误日志 |Blob |

## <a name="transfer-diagnostic-data"></a>传输诊断数据
对于 SDK 2.5 及更高版本，可以请求通过配置文件传输诊断数据。 可以按照配置中指定的计划时间间隔传输诊断数据。

对于 SDK 2.4 及更低版本，可以请求通过配置文件以及编程方式传输诊断数据。 编程方式还可以进行按需传输。

> [!IMPORTANT]
> 将诊断数据传输到 Azure 存储帐户时会产生费用，具体取决于诊断数据使用的存储资源。
> 
> 

## <a name="store-diagnostic-data"></a>存储诊断数据
日志数据存储在使用以下名称的 Blob 或表存储中：

**表**

* **WadLogsTable** - 使用跟踪侦听器以代码编写的日志。
* **WADDiagnosticInfrastructureLogsTable** - 诊断监视器和配置更改。
* **WADDirectoriesTable** - 诊断监视器监视的目录。  这包括 IIS 日志、IIS 失败请求日志和自定义目录。  在“容器”字段中指定 blob 日志文件的位置，在 RelativePath 字段中指定 blob 的名称。  AbsolutePath 字段指示文件的位置和名称，就像文件存在于 Azure 虚拟机上一样。
* **WADPerformanceCountersTable** - 性能计数器。
* **WADWindowsEventLogsTable** - Windows 事件日志。

**Blob**

* wad-control-container -（仅适用于 SDK 2.4 及更低版本）包含用于控制 Azure 诊断的 XML 配置文件  。
* **wad-iis-failedreqlogfiles** - 包含 IIS 失败请求日志中的信息。
* **wad-iis-logfiles** - 包含有关 IIS 日志的信息。
* **"custom"** - 自定义容器，基于配置由诊断监视器监视的目录。  此 blob 容器的名称在 WADDirectoriesTable 中指定。

## <a name="tools-to-view-diagnostic-data"></a>用于查看诊断数据的工具
将数据传输到存储后，可以使用多个工具进行查看。 例如：

* Visual Studio 中的服务器资源管理器 - 如果已安装 Azure Tools for Microsoft Visual Studio，则可以在服务器资源管理器中使用“Azure 存储”节点从 Azure 存储帐户查看只读 Blob 和表数据。 既可以从本地存储模拟器帐户显示数据，也可以从为 Azure 创建的存储帐户显示数据。 有关详细信息，请参阅[使用服务器资源管理器浏览和管理存储资源](/visualstudio/azure/vs-azure-tools-storage-resources-server-explorer-browse-manage)。
* [世纪互联 Azure 存储资源管理器](../../vs-azure-tools-storage-manage-with-storage-explorer.md) 是一款独立应用，可用于在 Windows、OSX 和 Linux 上轻松处理 Azure 存储数据。
* [Azure Management Studio](https://www.cerebrata.com/products/azure-management-studio/introduction) 随附 Azure 诊断管理器，可用于查看、下载和管理 Azure 中运行的应用程序收集的诊断数据。

## <a name="next-steps"></a>后续步骤
[使用 Azure 诊断跟踪云服务应用程序中的流](../../cloud-services/cloud-services-dotnet-diagnostics-trace-flow.md)

