---
title: 将 Linux 虚拟机从非托管磁盘转换为托管磁盘 | Azure
description: 如何在资源管理器部署模型中使用 Azure CLI 将 Linux VM 从非托管磁盘转换为托管磁盘
services: virtual-machines-linux
documentationcenter: ''
author: rockboyfor
manager: digimobile
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
origin.date: 12/15/2017
ms.date: 07/01/2019
ms.author: v-yeche
ms.subservice: disks
ms.openlocfilehash: f5d741211be83c7d8670932d1ed08dc673b747a8
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67569671"
---
# <a name="convert-a-linux-virtual-machine-from-unmanaged-disks-to-managed-disks"></a>将 Linux 虚拟机从非托管磁盘转换为托管磁盘

如果有使用非托管磁盘的现有 Linux 虚拟机 (VM)，可以将这些 VM 转换为使用 [Azure 托管磁盘](../linux/managed-disks-overview.md)。 此过程会同时转换 OS 磁盘和任何附加的数据磁盘。

本文介绍如何使用 Azure CLI 转换 VM。 如果需要安装或升级它，请参阅[安装 Azure CLI](https://docs.azure.cn/zh-cn/cli/install-azure-cli?view=azure-cli-latest)。 

## <a name="before-you-begin"></a>准备阶段
* 请查看[有关迁移到托管磁盘的常见问题](faq-for-disks.md#migrate-to-managed-disks)。

[!INCLUDE [virtual-machines-common-convert-disks-considerations](../../../includes/virtual-machines-common-convert-disks-considerations.md)]

* 不会删除在转换之前由 VM 使用的原始 VHD 和存储帐户。 它们会继续产生费用。 若要避免这些项目产生的费用，请在验证转换已完成后删除原始 VHD Blob。 如果需要找到这些未附加的磁盘以删除它们，请参阅我们的文章[查找并删除未附加的 Azure 托管和非托管磁盘](find-unattached-disks.md)。

## <a name="convert-single-instance-vms"></a>转换单实例 VM
本节介绍如何将单实例 Azure VM 从非托管磁盘转换为托管磁盘。 （如果 VM 位于可用性集中，请参阅下一部分。）可通过此过程，将 VM 从高级 (SSD) 非托管磁盘转换为高级托管磁盘，或从标准 (HDD) 非托管磁盘转换为标准托管磁盘。

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

1. 使用 [az vm deallocate](https://docs.azure.cn/zh-cn/cli/vm?view=azure-cli-latest#az-vm-deallocate) 解除分配 VM。 以下示例在名为 `myResourceGroup` 的资源组中解除分配名为 `myVM` 的 VM：

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

2. 使用 [az vm convert](https://docs.azure.cn/zh-cn/cli/vm?view=azure-cli-latest#az-vm-convert) 将 VM 转换为托管磁盘。 以下过程转换名为 `myVM` 的 VM，包括 OS 磁盘和任何数据磁盘：

    ```azurecli
    az vm convert --resource-group myResourceGroup --name myVM
    ```

3. 使用 [az vm start](https://docs.azure.cn/zh-cn/cli/vm?view=azure-cli-latest#az-vm-start) 在转换为托管磁盘后启动 VM。 以下示例启动名为 `myResourceGroup` 的资源组中名为 `myVM` 的 VM。

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```

## <a name="convert-vms-in-an-availability-set"></a>转换可用性集中的 VM

如果要转换为托管磁盘的 VM 位于可用性集中，则需要先将可用性集转换为托管可用性集。

可用性集中的所有 VM 都必须在转换可用性集之前解除分配。 可用性集本身转换为托管可用性集后，计划将所有 VM 转换为托管磁盘。 然后，启动所有 VM，并继续照常操作。

1. 使用 [az vm availability-set list](https://docs.azure.cn/zh-cn/cli/vm/availability-set?view=azure-cli-latest#az-vm-availability-set-list) 列出可用性集中的所有 VM。 以下示例列出了名为 `myResourceGroup` 的资源组中名为 `myAvailabilitySet` 的可用性集中的所有 VM：

    ```azurecli
    az vm availability-set show \
        --resource-group myResourceGroup \
        --name myAvailabilitySet \
        --query [virtualMachines[*].id] \
        --output table
    ```

2. 使用 [az vm deallocate](https://docs.azure.cn/zh-cn/cli/vm?view=azure-cli-latest#az-vm-deallocate) 解除分配所有 VM。 以下示例在名为 `myResourceGroup` 的资源组中解除分配名为 `myVM` 的 VM：

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

3. 使用 [az vm availability-set convert](https://docs.azure.cn/zh-cn/cli/vm/availability-set?view=azure-cli-latest#az-vm-availability-set-convert) 转换可用性集。 以下示例转换名为 `myResourceGroup` 的资源组中名为 `myAvailabilitySet` 的可用性集：

    ```azurecli
    az vm availability-set convert \
        --resource-group myResourceGroup \
        --name myAvailabilitySet
    ```

4. 使用 [az vm convert](https://docs.azure.cn/zh-cn/cli/vm?view=azure-cli-latest#az-vm-convert) 将所有 VM 转换为托管磁盘。 以下过程转换名为 `myVM` 的 VM，包括 OS 磁盘和任何数据磁盘：

    ```azurecli
    az vm convert --resource-group myResourceGroup --name myVM
    ```

5. 使用 [az vm start](https://docs.azure.cn/zh-cn/cli/vm?view=azure-cli-latest#az-vm-start) 在转换为托管磁盘后启动所有 VM。 以下示例在名为 `myResourceGroup` 的资源组中启动名为 `myVM` 的 VM：

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```

<!--Verify successfully-->

## <a name="convert-using-the-azure-portal"></a>使用 Azure 门户进行转换

还可以使用 Azure 门户将非托管磁盘转换为托管磁盘。

1. 登录到 [Azure 门户](https://portal.azure.cn)。
2. 从门户的 VM 列表中选择 VM。
3. 在 VM 的边栏选项卡中，从菜单中选择“磁盘”  。
4. 在“磁盘”  边栏选项卡的顶部，选择“迁移到托管磁盘”  。
5. 如果 VM 位于可用性集中，则“迁移到托管磁盘”  边栏选项卡上会出现“首先需要转换可用性集”的警告。 此警告应该有一个链接，单击该链接即可转换可用性集。 转换可用性集后，或者如果 VM 不在可用性集中，请单击“迁移”  以启动将磁盘迁移到托管磁盘的过程。

VM 将会停止并在完成迁移后重新启动。

## <a name="next-steps"></a>后续步骤

有关存储选项的详细信息，请参阅 [Azure 托管磁盘概述](../windows/managed-disks-overview.md)。

<!--Update_Description: update meta properties -->