---
title: 使用 PowerShell 克隆应用 - Azure 应用服务
description: 了解如何使用 PowerShell 将“应用服务”应用克隆到新的应用。
services: app-service\web
documentationcenter: ''
author: ahmedelnably
manager: stefsch
editor: ''
ms.assetid: f9a5cfa1-fbb0-41e6-95d1-75d457347a35
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
origin.date: 01/14/2018
ms.date: 07/29/2019
ms.author: v-biyu
ms.custom: seodec18
ms.openlocfilehash: c0c3270c0b47f3a38c9be0c0848f094c8f0d6caf
ms.sourcegitcommit: 5f260ee1d8ac487702b554a94cb971a3ee62a40b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2019
ms.locfileid: "68232272"
---
# <a name="azure-app-service-app-cloning-using-powershell"></a>使用 PowerShell 克隆 Azure App Service 应用

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

在 Microsoft 发行的 Azure PowerShell 版本 1.1.0 中，为 `New-AzWebApp` 添加了新选项，可让用户将现有“应用服务”应用克隆到不同区域或相同区域中的新建应用。 使用此选项，客户可跨不同区域快速轻松地部署多个应用。

高级和高级 V2 应用服务计划支持应用克隆。 新功能使用与应用服务备份功能相同的限制，具体请参阅[在 Azure 应用服务中备份应用](manage-backup.md)。

## <a name="cloning-an-existing-app"></a>克隆现有应用
方案：你想要将位于中国东部区域的现有应用的内容克隆到位于中国北部区域的新应用。 结合 -SourceWebApp 选项使用 Azure 资源管理器版本的 PowerShell cmdlet 来创建新的应用，即可实现此目的。

如果知道包含源应用的资源组名称，就可以使用以下 PowerShell 命令来获取源应用的信息（在本例中，该应用名为 `source-webapp`）：

```PowerShell
$srcapp = Get-AzWebApp -ResourceGroupName SourceAzureResourceGroup -Name source-webapp
```

要创建新的应用服务计划，可按以下示例中所示来使用 `New-AzAppServicePlan` 命令

```PowerShell
    New-AzAppServicePlan -Location "China East" -ResourceGroupName DestinationAzureResourceGroup -Name NewAppServicePlan -Tier Premium
```

使用 New-AzWebApp 命令，可在中国北部区域内创建新的 Web 应用，并将其绑定到现有高级层应用服务计划。 此外，还可以使用相同的资源组作为源 Web 应用，或定义新的资源组，如以下命令所示：

```PowerShell
    $destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "China North" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp
```

要克隆现有 Web 应用（包括所有关联的部署槽位），需要使用 `IncludeSourceWebAppSlots` 参数。 以下 PowerShell 命令演示如何在 `New-AzWebApp` 命令中使用该参数：

```PowerShell
    $destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "China North" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -IncludeSourceWebAppSlots
```

要在同一区域中克隆现有应用，需要在同一区域中创建新资源组和新的应用服务计划，然后使用以下 PowerShell 命令来克隆应用：

```PowerShell
    $destapp = New-AzWebApp -ResourceGroupName NewAzureResourceGroup -Name dest-webapp -Location "China East" -AppServicePlan NewAppServicePlan -SourceWebApp $srcap
```

## <a name="cloning-an-existing-app-slot"></a>克隆现有的应用槽
方案：希望将应用的现有部署槽位克隆到新应用或新槽位。 新的应用可与原始应用槽位于相同或不同的区域。

如果知道包含源应用的资源组名称，就可以使用以下 PowerShell 命令来获取与 `source-app` 相关的源应用的槽位信息（在本例中，该应用名为 `source-appslot`）：

```PowerShell
$srcappslot = Get-AzWebAppSlot -ResourceGroupName SourceAzureResourceGroup -Name source-app -Slot source-appslot
```

以下命令演示如何在新应用中创建源应用的克隆：

```PowerShell
    $destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-app -Location "China North" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcappslot
```

## <a name="configuring-traffic-manager-while-cloning-an-app"></a>在克隆应用时配置流量管理器
通过创建多区域应用和配置 Azure 流量管理器来将流量路由到所有这些应用是一个重要的方案，可确保客户应用的高可用性。 克隆现有应用时，可以选择将两个应用都连接到新的或现有的流量管理器配置文件。 仅支持 Azure 资源管理器版本的流量管理器。

### <a name="creating-a-new-traffic-manager-profile-while-cloning-an-app"></a>在克隆应用时创建新的流量管理器配置文件
方案：用户想要将应用克隆到另一个区域，同时配置包含这两个应用的 Azure 资源管理器流量管理器配置文件。 以下命令演示如何在新应用中创建源应用的克隆，同时配置新的流量管理器配置文件：

```PowerShell
    $destapp = New-AzWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "China East" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileName newTrafficManagerProfile
```

### <a name="adding-new-cloned-app-to-an-existing-traffic-manager-profile"></a>将新克隆的应用添加到现有的流量管理器配置文件
方案：用户已有一个 Azure 资源管理器流量管理器配置文件，现在想要将两个应用都添加为终结点。 要执行此操作，首先需要组合现有流量管理器配置文件 ID。 需要订阅 ID、资源组名称和现有流量管理器配置文件名称。

```PowerShell
$TMProfileID = "/subscriptions/<Your subscription ID goes here>/resourceGroups/<Your resource group name goes here>/providers/Microsoft.TrafficManagerProfiles/ExistingTrafficManagerProfileName"
```

获取流量管理器 ID 之后，根据以下命令演示在新应用中创建源应用的克隆，同时将它们添加到现有流量管理器配置文件：

```PowerShell
    $destapp = New-AzWebApp -ResourceGroupName <Resource group name> -Name dest-webapp -Location "China East" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileId $TMProfileID
```

## <a name="current-restrictions"></a>当前限制
下面是应用克隆的已知限制：

* 不会克隆自动缩放设置
* 不会克隆备份计划设置
* 不会克隆 VNET 设置
* 不会自动在目标应用上设置 App Insights
* 不会克隆简易身份验证设置
* 不会克隆 Kudu 扩展
* 不会克隆 TiP 规则
* 不会克隆数据库内容
* 如果克隆到不同的缩放单元，出站 IP 地址会更改

### <a name="references"></a>参考
* [应用服务克隆](app-service-web-app-cloning.md)
* [在 Azure 应用服务中备份应用](manage-backup.md)
* [Azure 流量管理器预览版对 Azure Resource Manager 的支持](../traffic-manager/traffic-manager-powershell-arm.md)
* [将 Azure PowerShell 与 Azure Resource Manager 结合使用](../azure-resource-manager/powershell-azure-resource-manager.md)

