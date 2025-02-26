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
origin.date: 04/24/2018
ms.date: 02/18/2019
ms.author: v-yeche
ms.custom: mvc
ms.openlocfilehash: 794f0ab6b87d5cef321a6752da3223cc49e5bc43
ms.sourcegitcommit: dd6cee8483c02c18fd46417d5d3bcc2cfdaf7db4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/22/2019
ms.locfileid: "56665961"
---
# <a name="quickstart-create-a-windows-virtual-machine-with-the-azure-cli"></a>快速入门：使用 Azure CLI 创建 Windows 虚拟机

Azure CLI 用于从命令行或脚本创建和管理 Azure 资源。 本快速入门展示了如何使用 Azure CLI 在 Azure 中部署运行 Windows Server 2016 的虚拟机 (VM)。 若要查看运行中的 VM，可以通过 RDP 登录到该 VM 并安装 IIS Web 服务器。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

<!--MOONCAKE: az cli 2.0 is put next paragrapgh-->

如果选择在本地安装并使用 CLI，本快速入门要求运行 Azure CLI 2.0.30 或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI](https://docs.azure.cn/zh-cn/cli/install-azure-cli?view=azure-cli-latest)。

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](https://docs.azure.cn/zh-cn/cli/group?view=azure-cli-latest#az-group-create) 命令创建资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 以下示例在“chinaeast”  位置创建名为“myResourceGroup”  的资源组：

```azurecli
az group create --name myResourceGroup --location chinaeast
```

## <a name="create-virtual-machine"></a>创建虚拟机

使用 [az vm create](https://docs.azure.cn/zh-cn/cli/vm?view=azure-cli-latest#az-vm-create) 创建 VM。 以下示例创建一个名为 myVM  的 VM。 此示例使用 azureuser  作为管理用户名，使用 myPassword12  作为密码。 更新这些值，使其适用于环境。 连接到 VM 时需要这些值。

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2016datacenter \
    --admin-username azureuser \
    --admin-password myPassword12
```

创建 VM 和支持资源需要几分钟时间。 以下示例输出表明 VM 创建操作已成功。

```azurecli
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "chinaeast",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

记下 VM 输出中自己的 `publicIpAddress`。 在后续步骤中，将使用此地址访问 VM。

## <a name="open-port-80-for-web-traffic"></a>为 Web 流量打开端口 80

默认情况下，在 Azure 中创建 Windows VM 时仅会打开 RDP 连接。 请使用 [az vm open-port](https://docs.azure.cn/zh-cn/cli/vm?view=azure-cli-latest#az-vm-open-port) 打开 TCP 端口 80 以供 IIS Web 服务器使用：

```azurecli
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

使用以下命令从本地计算机创建远程桌面会话。 将 IP 地址替换为你的 VM 的公用 IP 地址。 出现提示时，输入创建 VM 时使用的凭据：

```powershell
mstsc /v:publicIpAddress
```

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

如果不再需要资源组、VM 和所有相关的资源，可以使用 [az group delete](https://docs.azure.cn/zh-cn/cli/group?view=azure-cli-latest#az-group-delete) 命令将其删除：

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你部署了简单的虚拟机，打开了 Web 流量的网络端口，并安装了一个基本 Web 服务器。 若要详细了解 Azure 虚拟机，请继续学习 Windows VM 的教程。

> [!div class="nextstepaction"]
> [Azure Windows 虚拟机教程](./tutorial-manage-vm.md)

<!--Update_Description: update meta properties -->