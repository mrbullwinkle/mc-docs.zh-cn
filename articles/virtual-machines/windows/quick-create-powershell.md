---
title: 快速入门 - 使用 Azure PowerShell 创建 Windows VM | Azure
description: 本快速入门介绍了如何使用 Azure PowerShell 创建 Windows 虚拟机
services: virtual-machines-windows
documentationcenter: virtual-machines
author: rockboyfor
manager: digimobile
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
origin.date: 01/22/2019
ms.date: 02/18/2019
ms.author: v-yeche
ms.custom: mvc
ms.openlocfilehash: a6f100ac3d4d798234ec4c3e43bf8a18181a476e
ms.sourcegitcommit: dd6cee8483c02c18fd46417d5d3bcc2cfdaf7db4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/22/2019
ms.locfileid: "56666109"
---
# <a name="quickstart-create-a-windows-virtual-machine-in-azure-with-powershell"></a>快速入门：使用 PowerShell 在 Azure 中创建 Windows 虚拟机

Azure PowerShell 模块用于从 PowerShell 命令行或脚本创建和管理 Azure 资源。 本快速入门演示如何使用 Azure PowerShell 模块在 Azure 中部署运行 Windows Server 2016 的虚拟机 (VM)。 若要显示运行中的 VM，也可通过 RDP 登录到该 VM 并安装 IIS Web 服务器。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="launch-azure-cloud-shell"></a>启动 Azure 本地 Shell

<!--[!INCLUDE [cloud-shell-powershell](../../../includes/cloud-shell-powershell.md)]-->

[!INCLUDE [updated-for-az-vm.md](../../../includes/updated-for-az-vm.md)]

## <a name="create-resource-group"></a>创建资源组

使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 创建 Azure 资源组。 资源组是在其中部署和管理 Azure 资源的逻辑容器。

```powershell
New-AzResourceGroup -Name myResourceGroup -Location ChinaEast
```

## <a name="create-virtual-machine"></a>创建虚拟机

使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建 VM。 请提供每个资源的名称，如果这些资源不存在，`New-AzVM` cmdlet 会创建它们。

出现提示时，请提供用作 VM 登录凭据的用户名和密码：

```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Location "China East" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80,3389
```

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

在部署完成后，通过 RDP 登录到 VM。 若要查看运行中的 VM，请安装 IIS Web 服务器。

若要查看 VM 的公用 IP 地址，请使用 [Get-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/get-azpublicipaddress) cmdlet：

```powershell
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```

使用以下命令从本地计算机创建远程桌面会话。 将 IP 地址替换为您的虚拟机的公用 IP 地址。 

```powershell
mstsc /v:publicIpAddress
```

在“Windows 安全性”  窗口中，依次选择“更多选择”  、“使用其他帐户”  。 以 **localhost**\\*username* 的形式键入用户名，输入为虚拟机创建的密码，然后单击“确定”。 

你可能会在登录过程中收到证书警告。 单击“是”或“继续”以创建连接  

## <a name="install-web-server"></a>安装 Web 服务器

若要查看运行中的 VM，请安装 IIS Web 服务器。 在 VM 中打开 PowerShell 提示符并运行以下命令：

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

完成后，关闭到 VM 的 RDP 连接。

## <a name="view-the-web-server-in-action"></a>查看运行中的 Web 服务器

IIS 已安装，并且现在已从 Internet 打开 VM 上的端口 80 - 可以使用所选的 Web 浏览器查看默认的 IIS 欢迎页。 使用上一步中获取的 VM 的公用 IP 地址。 以下示例展示了默认 IIS 网站：

![IIS 默认站点](./media/quick-create-powershell/default-iis-website.png)

## <a name="clean-up-resources"></a>清理资源

不再需要时，可以使用 [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) cmdlet 删除资源组、VM 和所有相关资源：

```powershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你部署了简单的虚拟机，打开了 Web 流量的网络端口，并安装了一个基本 Web 服务器。 若要详细了解 Azure 虚拟机，请继续学习 Windows VM 的教程。

> [!div class="nextstepaction"]
> [Azure Windows 虚拟机教程](./tutorial-manage-vm.md)

<!--Update_Description: update meta properties, wording update-->