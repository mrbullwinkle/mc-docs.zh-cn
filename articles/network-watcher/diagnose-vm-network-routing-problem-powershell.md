---
title: 诊断虚拟机网络路由问题 - Azure PowerShell | Azure
description: 本文介绍如何使用 Azure 网络观察程序的“下一个跃点”功能来诊断虚拟机网络路由问题。
services: network-watcher
documentationcenter: network-watcher
author: rockboyfor
manager: digimobile
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose virtual machine (VM) network routing problem that prevents communication to different destinations.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
origin.date: 04/20/2018
ms.date: 07/02/2018
ms.author: v-yeche
ms.custom: ''
ms.openlocfilehash: 8053373060ee392ca5491ca1748c20c3d1ba92b8
ms.sourcegitcommit: 0500a329025e55e4e0bdbf68d48d8bcbfab06ec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/14/2019
ms.locfileid: "65597980"
---
# <a name="diagnose-a-virtual-machine-network-routing-problem---azure-powershell"></a>诊断虚拟机网络路由问题 - Azure PowerShell

在本教程中，请先部署虚拟机 (VM)，然后检查到某个 IP 地址和 URL 的通信。 确定通信失败的原因以及解决方法。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]


如果选择在本地安装和使用 PowerShell，则本文需要 Azure PowerShell `Az` 模块。 要查找已安装的版本，请运行 `Get-Module -ListAvailable Az`。 如果需要进行升级，请参阅 [Install Azure PowerShell module](https://docs.microsoft.com/powershell/azure/install-Az-ps)（安装 Azure PowerShell 模块）。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount` 来创建与 Azure 的连接。



## <a name="create-a-vm"></a>创建 VM

在创建 VM 之前，必须创建该 VM 所属的资源组。 使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.Resources/New-azResourceGroup) 创建资源组。 以下示例在“eastus”  位置创建名为“myResourceGroup”  的资源组。

```PowerShell
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建 VM。 运行此步骤时，会提示输入凭据。 输入的值将配置为用于 VM 的用户名和密码。

```PowerShell
$vM = New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVm" `
    -Location "China East"
```

创建 VM 需要几分钟时间。 在创建好 VM 且 PowerShell 返回输出之前，请勿继续执行剩余的步骤。

## <a name="test-network-communication"></a>测试网络通信

若要通过网络观察程序测试网络通信，必须先在要测试的 VM 所在区域中启用网络观察程序，然后使用网络观察程序的“下一个跃点”功能来测试通信。

## <a name="enable-network-watcher"></a>启用网络观察程序

如果已在中国东部区域启用了网络观察程序，请使用 [Get-AzureRmNetworkWatcher](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermnetworkwatcher) 来检索网络观察程序。 以下示例检索 NetworkWatcherRG  资源组中名为 NetworkWatcher_chinaeast  的现有网络观察程序：

```powershell
$networkWatcher = Get-AzNetworkWatcher `
  -Name NetworkWatcher_chinaeast `
  -ResourceGroupName NetworkWatcherRG
```

如果还没有在中国东部区域启用网络观察程序，请使用 [New-AzNetworkWatcher](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermnetworkwatcher) 在中国东部区域创建网络观察程序：

```powershell
$networkWatcher = New-AzNetworkWatcher `
  -Name "NetworkWatcher_chinaeast" `
  -ResourceGroupName "NetworkWatcherRG" `
  -Location "China East"
```

### <a name="use-next-hop"></a>使用下一个跃点

Azure 自动创建到默认目标的路由。 可以创建自定义路由来覆盖默认路由。 有时候，自定义路由可能会导致通信故障。 要测试来自 VM 的路由，请使用 [Get-AzureRmNetworkWatcherNextHop](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermnetworkwatchernexthop) 命令确定流量发送到特定地址时的下一个路由跃点。

测试从 VM 发往 www.bing.com 的某个 IP 地址的出站通信：

```PowerShell
Get-AzNetworkWatcherNextHop `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $VM.Id `
  -SourceIPAddress 192.168.1.4 `
  -DestinationIPAddress 13.107.21.200
```

数秒钟后，输出结果指示 NextHopType 为“Internet”  ，RouteTableId 为“系统路由”    。 此结果指示存在通往目标的有效路由。

测试从 VM 发往 172.31.0.100 的出站通信：

```PowerShell
Get-AzNetworkWatcherNextHop `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $VM.Id `
  -SourceIPAddress 192.168.1.4 `
  -DestinationIPAddress 172.31.0.100
```

输出结果指示“NextHopType”为“无”，“RouteTableId”仍为“系统路由”。     此结果指示，虽然存在有效的通往目标的系统路由，但是没有将流量路由到目标的下一跃点。

## <a name="view-details-of-a-route"></a>查看路由详细信息

若要进一步分析路由情况，请使用 [Get-AzEffectiveRouteTable](https://docs.microsoft.com/powershell/module/az.network/get-azeffectiveroutetable) 命令查看网络接口的有效路由：

```PowerShell
Get-AzEffectiveRouteTable `
  -NetworkInterfaceName myVm `
  -ResourceGroupName myResourceGroup |
  Format-table
```

将返回包含以下文本的输出：

```powershell
Name State  Source  AddressPrefix           NextHopType NextHopIpAddress
---- -----  ------  -------------           ----------- ----------------
     Active Default {192.168.0.0/16}        VnetLocal   {}              
     Active Default {0.0.0.0/0}             Internet    {}              
     Active Default {10.0.0.0/8}            None        {}              
     Active Default {100.64.0.0/10}         None        {}              
     Active Default {172.16.0.0/12}         None        {}              
```

正如在之前输出中所看到的，带有 0.0.0.0/0  的 AddressPrefix  的路由会将未指定给地址的所有流量路由到以 Internet 的下一个跃点为前缀的其他路由地址内  。 同时还可在输出结果中看到，虽然有一个到 172.16.0.0/12 前缀的默认路由（其中包括地址 172.31.0.100），但“nextHopType”为“无”。   Azure 会创建到 172.16.0.0/12 的默认路由，但不会无故指定下一个跃点类型。 在特定情况下，例如在已将 172.16.0.0/12 地址范围添加到虚拟网络的地址空间的情况下，Azure 会将路由的“nextHopType”更改为“虚拟网络”。   此时进行检查会将“nextHopType”显示为“虚拟网络”   。

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组及其包含的所有资源，请使用 [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) 将其删除：

```PowerShell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>后续步骤

本文介绍了如何创建 VM 并根据该 VM 诊断网络路由问题。 同时说明了 Azure 可以创建多个默认路由，并且还测试了到两个不同目标的路由。 详细了解 [Azure 中的路由](../virtual-network/virtual-networks-udr-overview.md?toc=%2fnetwork-watcher%2ftoc.json)，以及如何[创建自定义路由](../virtual-network/manage-route-table.md?toc=%2fnetwork-watcher%2ftoc.json#create-a-route)。

对于出站 VM 连接，还可以使用网络观察程序的[连接故障排除](network-watcher-connectivity-powershell.md)功能来确定延迟、VM 和终结点之间获得允许的和被拒绝的网络流量。 可以使用网络观察程序的连接监视器功能监视 VM 和终结点（例如 IP 地址或 URL）之间在某段时间的通信情况。 如需了解如何操作，请参阅[监视网络连接](connection-monitor.md)。
<!-- Update_Description: new articles on network watcher diagnose vm network routing problem powershell -->
<!--ms.date: 07/02/2018-->