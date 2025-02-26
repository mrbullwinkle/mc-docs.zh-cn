---
title: 适用于 Linux 的 Azure 网络观察程序代理虚拟机扩展 | Azure
description: 使用虚拟机扩展在 Linux 虚拟机上部署网络观察程序代理。
services: virtual-machines-linux
documentationcenter: ''
author: rockboyfor
manager: digimobile
editor: ''
tags: azure-resource-manager
ms.assetid: 5c81e94c-e127-4dd2-ae83-a236c4512345
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
origin.date: 02/14/2017
ms.date: 05/20/2019
ms.author: v-yeche
ms.openlocfilehash: 439bdcb9a4ec8538e1fd4199b24708f3ba1ef413
ms.sourcegitcommit: 1ea0f453e7dcaef67f3c52747778c7f3b82e3e38
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/20/2019
ms.locfileid: "67277539"
---
# <a name="network-watcher-agent-virtual-machine-extension-for-linux"></a>适用于 Linux 的网络观察程序代理虚拟机扩展

## <a name="overview"></a>概述

[Azure 网络观察程序](/network-watcher/)是一项网络性能监视、诊断和分析服务，适用于对 Azure 网络进行监视。 网络观察程序代理虚拟机 (VM) 扩展是 Azure VM 上的某些网络观察程序功能（例如按需捕获网络流量）和其他高级功能所必需的。

本文详细介绍适用于 Linux 的网络观察程序代理 VM 扩展支持的平台和部署选项。 安装代理时不会中断，也不会需要重新启动 VM。 可以将扩展部署到你部署的虚拟机中。 如果虚拟机由 Azure 服务部署，请查看该服务的文档以确定它是否允许在虚拟机中安装扩展。

## <a name="prerequisites"></a>先决条件

### <a name="operating-system"></a>操作系统

可以针对下列 Linux 分发配置网络观察程序代理扩展：

| 分发 | 版本 |
|---|---|
| Ubuntu | 12+ |
| Debian | 7 和 8 |
| SUSE Linux Enterprise Server | 11 和 12 |
| OpenSUSE Leap | 42.3+ |
| CentOS | 6.5+ 和 7 |
| CoreOS | 899.17.0+ |

<!-- Not Available on | Oracle Linux | 6.8+ and 7 |-->
<!-- Not Available on | RedHat | 6 and 7 |-->

### <a name="internet-connectivity"></a>Internet 连接

某些网络观察程序代理功能要求将 VM 连接到 Internet。 如果无法建立传出连接，某些网络观察程序代理功能可能无法正常使用，或者会变得不可使用。 有关需要代理的网络观察程序功能的详细信息，请参阅[网络观察程序文档](/network-watcher/)。

## <a name="extension-schema"></a>扩展架构

以下 JSON 显示网络观察程序代理扩展的架构。 扩展不需要或不支持用户提供的任何设置。 扩展依赖于其默认配置。

```json
{
    "type": "extensions",
    "name": "Microsoft.Azure.NetworkWatcher",
    "apiVersion": "[variables('apiVersion')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.Azure.NetworkWatcher",
        "type": "NetworkWatcherAgentLinux",
        "typeHandlerVersion": "1.4",
        "autoUpgradeMinorVersion": true
    }
}
```

### <a name="property-values"></a>属性值

| 名称 | 值/示例 |
| ---- | ---- |
| apiVersion | 2015-06-15 |
| publisher | Microsoft.Azure.NetworkWatcher |
| type | NetworkWatcherAgentLinux |
| typeHandlerVersion | 1.4 |

## <a name="template-deployment"></a>模板部署

可使用 Azure 资源管理器模板部署 Azure VM 扩展。 若要部署网络观察程序代理扩展，请在模板中使用以前的 json 架构。

## <a name="azure-classic-cli-deployment"></a>Azure 经典 CLI 部署

下面的示例将网络观察程序代理 VM 扩展部署到通过经典部署模型部署的现有 VM：

```azurecli
azure config mode asm
azure vm extension set myVM1 NetworkWatcherAgentLinux Microsoft.Azure.NetworkWatcher 1.4
```

## <a name="azure-cli-deployment"></a>Azure CLI 部署

下面的示例将网络观察程序代理 VM 扩展部署到通过资源管理器部署的现有 VM：

```azurecli
az vm extension set --resource-group myResourceGroup1 --vm-name myVM1 --name NetworkWatcherAgentLinux --publisher Microsoft.Azure.NetworkWatcher --version 1.4
```

## <a name="troubleshooting-and-support"></a>故障排除和支持

### <a name="troubleshooting"></a>故障排除

可以从 Azure 门户或 Azure CLI 检索有关扩展部署状态的数据。

下面的示例演示使用 Azure 经典 CLI 通过经典部署模型部署的 VM 扩展的部署状态：

```azurecli
azure config mode asm
azure vm extension get myVM1
```
扩展执行输出记录到在以下目录中发现的文件：

```
/var/log/azure/Microsoft.Azure.NetworkWatcher.NetworkWatcherAgentLinux/
```

下面的示例演示使用 Azure CLI 通过资源管理器部署的 VM 的 NetworkWatcherAgentLinux 扩展的部署状态：

```azurecli
az vm extension show --name NetworkWatcherAgentLinux --resource-group myResourceGroup1 --vm-name myVM1
```

### <a name="support"></a>支持

如果对本文中的任何内容不了解，可以参阅[网络观察程序文档](/network-watcher/)或联系 [Azure 支持](https://support.azure.cn/zh-cn/support/contact)上的 Azure 专家。 或者，也可以提出 Azure 支持事件。 请转到 [Azure 支持站点](https://www.azure.cn/support/contact/)并选择“获取支持”  。 有关使用 Azure 支持的信息，请参阅 [Azure 支持常见问题](https://www.azure.cn/support/faq/)。

<!-- Update_Description: update meta properties, wording update  -->