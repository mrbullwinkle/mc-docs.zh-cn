---
title: 使用虚拟网络对等互连连接虚拟网络 - PowerShell | Azure
description: 在本文中，你将学习如何使用 Azure PowerShell 通过虚拟网络对等互连来连接虚拟网络。
services: virtual-network
documentationcenter: virtual-network
author: rockboyfor
manager: digimobile
editor: ''
tags: azure-resource-manager
Customer intent: I want to connect two virtual networks so that virtual machines in one virtual network can communicate with virtual machines in the other virtual network.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: article
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
origin.date: 03/13/2018
ms.date: 07/22/2019
ms.author: v-yeche
ms.custom: ''
ms.openlocfilehash: f4cde142e46e637dbb7eec8dc5c16c856bc8c4fe
ms.sourcegitcommit: 021dbf0003a25310a4c8582a998c17729f78ce42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68514439"
---
# <a name="connect-virtual-networks-with-virtual-network-peering-using-powershell"></a>通过 PowerShell 使用虚拟网络对等互连连接虚拟网络

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

可以使用虚拟网络对等互连将虚拟网络互相连接。 将虚拟网络对等互连后，两个虚拟网络中的资源将能够以相同的延迟和带宽相互通信，就像这些资源位于同一个虚拟网络中一样。 在本文中，学习如何：

* 创建两个虚拟网络
* 使用虚拟网络对等互连连接两个虚拟网络。
* 将虚拟机 (VM) 部署到每个虚拟网络
* VM 之间进行通信

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

<!--[!INCLUDE [cloud-shell-powershell](../../../includes/cloud-shell-powershell.md)]-->

如果选择在本地安装和使用 PowerShell，则本文需要 Azure PowerShell 模块 1.0.0 或更高版本。 运行 `Get-Module -ListAvailable Az` 查找已安装的版本。 如果需要进行升级，请参阅 [Install Azure PowerShell module](https://docs.microsoft.com/powershell/azure/install-az-ps)（安装 Azure PowerShell 模块）。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount -Environment AzureChinaCloud` 来创建与 Azure 的连接。

## <a name="create-virtual-networks"></a>创建虚拟网络

创建虚拟网络之前，必须为虚拟网络创建资源组以及本文中创建的所有其他资源。 使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 创建资源组。 以下示例在“chinaeast”  位置创建名为“myResourceGroup”  的资源组。

```powershell
Connect-AzAccount -Environment AzureChinaCloud
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location ChinaEast
```

使用 [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork) 创建虚拟网络。 以下示例创建地址前缀为 *10.0.0.0/16* 且名为 *myVirtualNetwork1* 的虚拟网络。

```powershell
$virtualNetwork1 = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location ChinaEast `
  -Name myVirtualNetwork1 `
  -AddressPrefix 10.0.0.0/16
```

使用 [New-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetworksubnetconfig) 创建子网配置。 以下示例创建地址前缀为 10.0.0.0/24 的子网配置：

```powershell
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name Subnet1 `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork1
```

使用 [Set-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/Set-azVirtualNetwork) 将子网配置写入虚拟网络，从而创建子网：

```powershell
$virtualNetwork1 | Set-AzVirtualNetwork
```

创建地址前缀为 10.1.0.0/16 的虚拟网络和一个子网：

```powershell
# Create the virtual network.
$virtualNetwork2 = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location ChinaEast `
  -Name myVirtualNetwork2 `
  -AddressPrefix 10.1.0.0/16

# Create the subnet configuration.
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name Subnet1 `
  -AddressPrefix 10.1.0.0/24 `
  -VirtualNetwork $virtualNetwork2

# Write the subnet configuration to the virtual network.
$virtualNetwork2 | Set-AzVirtualNetwork
```

## <a name="peer-virtual-networks"></a>将虚拟网络对等互连

使用 [Add-AzVirtualNetworkPeering](https://docs.microsoft.com/powershell/module/az.network/add-azvirtualnetworkpeering) 创建对等互连。 以下示例将 *myVirtualNetwork1* 对等互连到 *myVirtualNetwork2*。

```powershell
Add-AzVirtualNetworkPeering `
  -Name myVirtualNetwork1-myVirtualNetwork2 `
  -VirtualNetwork $virtualNetwork1 `
  -RemoteVirtualNetworkId $virtualNetwork2.Id
```

在上一个命令执行后返回的输出中，可以看到 **PeeringState** 为 *Initiated*。 对等互连将保持 *Initiated* 状态，直到你创建从 *myVirtualNetwork2* 到 *myVirtualNetwork1* 的对等互连。 创建从 *myVirtualNetwork2* 到 *myVirtualNetwork1* 的对等互连。

```powershell
Add-AzVirtualNetworkPeering `
  -Name myVirtualNetwork2-myVirtualNetwork1 `
  -VirtualNetwork $virtualNetwork2 `
  -RemoteVirtualNetworkId $virtualNetwork1.Id
```

在上一个命令执行后返回的输出中，可以看到 **peeringState** 为 *Connected*。 Azure 还将 *myVirtualNetwork1-myVirtualNetwork2* 对等互连的对等互连状态更改为 *Connected*。 使用 [Get-AzVirtualNetworkPeering](https://docs.microsoft.com/powershell/module/az.network/get-azvirtualnetworkpeering) 确认 myVirtualNetwork1-myVirtualNetwork2  对等互连的对等互连状态是否已更改为“Connected”  。

```powershell
Get-AzVirtualNetworkPeering `
  -ResourceGroupName myResourceGroup `
  -VirtualNetworkName myVirtualNetwork1 `
  | Select PeeringState
```

在两个虚拟网络中的对等互连的 **PeeringState** 为 *Connected* 之前，在一个虚拟网络中的资源无法与另一个虚拟网络中的资源通信。

## <a name="create-virtual-machines"></a>创建虚拟机

在稍后的步骤中，会在每个虚拟网络中创建一个 VM，以便可以在它们之间进行通信。

### <a name="create-the-first-vm"></a>创建第一个 VM

使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建 VM。 以下示例在 *myVirtualNetwork1* 虚拟网络中创建一个名为 *myVm1* 的 VM。 `-AsJob` 选项会在后台创建 VM，因此可继续执行下一步。 系统提示时，请输入想要用来登录到 VM 的用户名和密码。

```powershell
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "China East" `
  -VirtualNetworkName "myVirtualNetwork1" `
  -SubnetName "Subnet1" `
  -ImageName "Win2016Datacenter" `
  -Name "myVm1" `
  -Size "Standard_A1" `
  -AsJob
```

### <a name="create-the-second-vm"></a>创建第二个 VM

```powershell
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "China East" `
  -VirtualNetworkName "myVirtualNetwork2" `
  -SubnetName "Subnet1" `
  -ImageName "Win2016Datacenter" `
  -Size "Standard_A1" `
  -Name "myVm2"
```

创建 VM 需要几分钟时间。 在 Azure 创建 VM 并将输出返回到 PowerShell 之前，不要继续执行后续步骤。

## <a name="communicate-between-vms"></a>VM 之间进行通信

可以从 Internet 连接到 VM 的公用 IP 地址。 使用 [Get-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/get-azpublicipaddress) 返回 VM 的公共 IP 地址。 以下示例返回 myVm1 VM 的公共 IP 地址  ：

```powershell
Get-AzPublicIpAddress `
  -Name myVm1 `
  -ResourceGroupName myResourceGroup | Select IpAddress
```

从本地计算机使用以下命令创建与 myVm1 VM 的远程桌面会话  。 将 `<publicIpAddress>` 替换为上一命令返回的 IP 地址。

```
mstsc /v:<publicIpAddress>
```

此时会创建远程桌面协议 (.rdp) 文件，并下载到计算机，同时打开该文件。 输入用户名和密码（可能需要选择“更多选择”，然后选择“使用其他帐户”，以便指定在创建 VM 时输入的凭据），然后单击“确定”。    你可能会在登录过程中收到证书警告。 单击“是”或“继续”继续进行连接。  

在 *myVm1* VM 上，允许 Internet 控制消息协议 (ICMP) 通过 Windows 防火墙，以便在稍后的步骤中使用 PowerShell 从 *myVm2* ping 此 VM：

```powershell
New-NetFirewallRule -DisplayName "Allow ICMPv4-In" -Protocol ICMPv4
```

虽然本文中使用 ping 在 VM 之间进行通信，但在进行生产部署时，不建议允许 ICMP 通过 Windows 防火墙。

若要连接到 *myVm2* VM，请在 *myVm1* VM 上通过命令提示符输入以下命令：

```
mstsc /v:10.1.0.4
```

由于已在 *myVm1* 上启用了 ping，因此，现在可以在 *myVm2* VM 上通过命令提示符按 IP 地址对它执行 ping 操作：

```
ping 10.0.0.4
```

会收到四条回复。 断开到 *myVm1* 和 *myVm2* 的 RDP 会话。

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组及其包含的所有资源，请使用 [Remove-AzResourcegroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) 将其删除。

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>后续步骤

在本文中，你已学习了如何使用虚拟网络对等互连来连接同一 Azure 区域中的两个网络。 还可以将不同的[受支持区域](virtual-network-manage-peering.md#cross-region)和[不同的 Azure 订阅](create-peering-different-subscriptions.md#powershell)中的虚拟网络对等互连。 若要详细了解虚拟网络对等互连，请参阅[虚拟网络对等互连概述](virtual-network-peering-overview.md)和[管理虚拟网络对等互连](virtual-network-manage-peering.md)。

<!-- Not Available on [hub and spoke network designs](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fvirtual-network%2ftoc.json#vnet-peering)-->

可以通过 VPN [将自己的计算机连接到虚拟网络](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md?toc=%2fvirtual-network%2ftoc.json)，并可与虚拟网络或对等虚拟网络中的资源进行交互。 有关用来完成虚拟网络文章中涉及的许多任务的可重用脚本，请参阅[脚本示例](powershell-samples.md)。

<!-- Update_Description: update meta properties -->
