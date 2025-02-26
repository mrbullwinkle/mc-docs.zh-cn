---
title: 教程 - Azure 中 Windows VM 的高可用性 | Azure
description: 本教程介绍如何使用 Azure PowerShell 在可用性集中部署高度可用的虚拟机
documentationcenter: ''
services: virtual-machines-windows
author: rockboyfor
manager: digimobile
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: tutorial
origin.date: 11/30/2018
ms.date: 04/01/2019
ms.author: v-yeche
ms.custom: mvc
ms.openlocfilehash: 08179a8125172d841e2bfc38ee23d185c102111e
ms.sourcegitcommit: 3b05a8982213653ee498806dc9d0eb8be7e70562
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/04/2019
ms.locfileid: "59004027"
---
# <a name="tutorial-create-and-deploy-highly-available-virtual-machines-with-azure-powershell"></a>教程：使用 Azure PowerShell 创建和部署高度可用的虚拟机

本教程介绍如何使用可用性集提高虚拟机 (VM) 的可用性和可靠性。 可用性集确保在 Azure 上部署的 VM 能够跨群集中多个隔离的硬件节点分布。 

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建可用性集
> * 在可用性集中创建 VM
> * 检查可用的 VM 大小
> * 检查 Azure 顾问

## <a name="availability-set-overview"></a>可用性集概述

可用性集是一种逻辑分组功能，可将部署的 VM 资源相互隔离。 Azure 确保可用性集中部署的 VM 能够跨多个物理服务器、计算机架、存储单元和网络交换机运行。 如果发生硬件或软件故障，只有一部分 VM 会受到影响，整体解决方案仍会保持正常运行。 可用性集对于构建可靠的云解决方案至关重要。

假设某个基于 VM 的典型解决方案包含四个前端 Web 服务器，以及 2 个后端 VM。 在 Azure 中，若想在部署 VM 之前先定义两个可用性集：一个用于 Web 层，另一个用于后端层。 创建新 VM 时，请将可用性集指定为参数。 Azure 确保 VM 在多个物理硬件资源之间保持隔离。 如果运行服务器的物理硬件有问题，可以确信服务器的其他实例保持运行，因为它们位于不同的硬件上。

在 Azure 中部署基于 VM 的可靠解决方案时，使用可用性集。

## <a name="launch-azure-powershell"></a>启动 Azure PowerShell

[!INCLUDE [updated-for-az-vm.md](../../../includes/updated-for-az-vm.md)]


## <a name="create-an-availability-set"></a>创建可用性集

同一位置的硬件分为多个更新域和容错域。 更新域  是一组可同时重启的 VM 和基础物理硬件。 同一个容错域  内的 VM 共享公用存储，以及公用电源和网络交换机。  

可以使用 [New-AzAvailabilitySet](https://docs.microsoft.com/powershell/module/az.compute/new-azavailabilityset) 创建可用性集。 在此示例中，更新域和容错域的数目为 *2*，可用性集名为 *myAvailabilitySet*。

创建资源组。

```powershell
Connect-AzAccount -Environment AzureChinaCloud
New-AzResourceGroup `
   -Name myResourceGroupAvailability `
   -Location ChinaEast
```

结合 `-sku aligned` 参数使用 [New-AzAvailabilitySet](https://docs.microsoft.com/powershell/module/az.compute/new-azavailabilityset) 创建托管的可用性集。

```powershell
New-AzAvailabilitySet `
   -Location "ChinaEast" `
   -Name "myAvailabilitySet" `
   -ResourceGroupName "myResourceGroupAvailability" `
   -Sku aligned `
   -PlatformFaultDomainCount 2 `
   -PlatformUpdateDomainCount 2
```

## <a name="create-vms-inside-an-availability-set"></a>在可用性集内创建 VM
必须在可用性集中创建 VM，确保它们正确地分布在硬件中。 创建后，无法将现有 VM 添加到可用性集中。 

通过 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建 VM 时，请使用 `-AvailabilitySetName` 参数指定可用性集的名称。

首先，使用 [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 设置 VM 的管理员用户名和密码：

```powershell
$cred = Get-Credential
```

现在，请使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 在可用性集中创建两个 VM。

```powershell
for ($i=1; $i -le 2; $i++)
{
    New-AzVm `
        -ResourceGroupName "myResourceGroupAvailability" `
        -Name "myVM$i" `
        -Location "China East" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -PublicIpAddressName "myPublicIpAddress$i" `
        -AvailabilitySetName "myAvailabilitySet" `
        -Credential $cred
}
```

创建和配置这两个 VM 需要几分钟的时间。 完成后，你将拥有两个跨基础硬件分布的虚拟机。 

如果转到“资源组” > “myResourceGroupAvailability” > “myAvailabilitySet”在门户中查看可用性集，应会看到 VM 在两个容错域和更新域之间的分布方式。   

![门户中的可用性集](./media/tutorial-availability-sets/fd-ud.png)

## <a name="check-for-available-vm-sizes"></a>检查可用的 VM 大小 

稍后可向可用性集添加更多 VM，但需了解在硬件上可用的 VM 大小。 使用 [Get-AzVMSize](https://docs.microsoft.com/powershell/module/az.compute/get-azvmsize) 列出可用性集的硬件群集上的所有可用大小。

```powershell
Get-AzVMSize `
   -ResourceGroupName "myResourceGroupAvailability" `
   -AvailabilitySetName "myAvailabilitySet"
```

## <a name="check-azure-advisor"></a>检查 Azure 顾问 

还可使用 Azure 顾问获取有关如何提高 VM 可用性的详细信息。 Azure 顾问可分析配置和用量遥测数据，然后推荐解决方案来帮助提高 Azure 资源的经济效益、性能、可用性和安全性。

登录到 [Azure 门户](https://portal.azure.cn)，选择“所有服务”，然后键入“顾问”   。 顾问仪表板显示针对所选订阅的个性化建议。 有关详细信息，请参阅 [Azure 顾问入门](../../advisor/advisor-get-started.md)。

## <a name="next-steps"></a>后续步骤

在本教程中，你已学习了如何执行以下操作：

> [!div class="checklist"]
> * 创建可用性集
> * 在可用性集中创建 VM
> * 检查可用的 VM 大小
> * 检查 Azure 顾问

请转到下一教程，了解虚拟机规模集。

> [!div class="nextstepaction"]
> [创建 VM 规模集](tutorial-create-vmss.md)

<!--Update_Description: update meta properties, wording update -->
