---
title: 有关 Linux 虚拟机的常见问题 | Azure
description: 回答了通过 Resource Manager 模型创建的 Linux 虚拟机的一些常见问题。
services: virtual-machines-linux
documentationcenter: ''
author: rockboyfor
manager: digimobile
editor: ''
tags: azure-resource-management
ms.assetid: 3648e09c-1115-4818-93c6-688d7a54a353
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
origin.date: 05/08/2019
ms.date: 07/01/2019
ms.author: v-yeche
ms.openlocfilehash: 224c3e02160336f617224dc41d5205534dbb0e3d
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67570506"
---
# <a name="frequently-asked-question-about-linux-virtual-machines"></a>有关 Linux 虚拟机的常见问题
本文讨论有关在 Azure 中使用 Resource Manager 部署模型创建的 Linux 虚拟机的一些常见问题。 有关本主题的 Windows 版本，请参阅[有关 Windows 虚拟机的常见问题](../windows/faq.md?toc=%2fvirtual-machines%2fwindows%2ftoc.json)

## <a name="what-can-i-run-on-an-azure-vm"></a>我可以在 Azure VM 上运行什么程序？
所有订户都可以在 Azure 虚拟机上运行服务器软件。 有关详细信息，请参阅 [Azure 认可的分发版本中的 Linux](endorsed-distros.md?toc=%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="how-much-storage-can-i-use-with-a-virtual-machine"></a>使用虚拟机时，我可以使用多少存储？
每个数据磁盘的容量高达 4 TB (4,095 GB)。 可以使用的数据磁盘数取决于虚拟机大小。 有关详细信息，请参阅[虚拟机大小](sizes.md?toc=%2fvirtual-machines%2flinux%2ftoc.json)。

Azure 托管磁盘是推荐用于 Azure 虚拟机的磁盘存储产品，方便永久存储数据。 可对每个虚拟机使用多个托管磁盘。 托管磁盘提供两种类型的持久存储选项：高级和标准托管磁盘。 有关定价信息，请参阅[托管磁盘定价](https://www.azure.cn/pricing/details/storage/)。

Azure 存储帐户还可为操作系统磁盘和任何数据磁盘提供存储空间。 每个磁盘都是一个 .vhd 文件，以页 blob 形式存储。 有关定价详细信息，请参阅 [Storage Pricing Details](https://www.azure.cn/pricing/details/storage/)（存储定价详细信息）。

## <a name="how-can-i-access-my-virtual-machine"></a>如何访问我的虚拟机？
使用安全外壳 (SSH) 建立远程连接，以登录到虚拟机。 请参阅如何[从 Windows](ssh-from-windows.md?toc=%2fvirtual-machines%2flinux%2ftoc.json) 或[从 Linux 和 Mac](mac-create-ssh-keys.md?toc=%2fvirtual-machines%2flinux%2ftoc.json) 进行连接的相关说明。 默认情况下，SSH 允许的并发连接最多为 10 个。 通过编辑配置文件，可以增加此数量。

如果遇到问题，请查阅[排查安全外壳 (SSH) 连接问题](troubleshoot-ssh-connection.md?toc=%2fvirtual-machines%2flinux%2ftoc.json)。

## <a name="can-i-use-the-temporary-disk-devsdb1-to-store-data"></a>我是否可以使用临时磁盘 (/dev/sdb1) 存储数据？
不要使用临时磁盘 (/dev/sdb1) 存储数据。 它只是用于临时存储。 有丢失无法恢复的数据的风险。

## <a name="can-i-copy-or-clone-an-existing-azure-vm"></a>我是否可以复制或克隆现有的 Azure VM？
是的。 有关说明，请参阅[如何在 Resource Manager 部署模型中创建 Linux 虚拟机的副本](copy-vm.md?toc=%2fvirtual-machines%2flinux%2ftoc.json)。

<!-- Not Available ## Why am I not seeing Canada Central and Canada East regions through Azure Resource Manager? -->
## <a name="can-i-add-a-nic-to-my-vm-after-its-created"></a>创建 VM 后能否向 VM 添加 NIC？
能，目前可行。 首先需停止解除分配 VM。 然后便可添加或删除 NIC（除非它是 VM 上的最后一个 NIC）。 

## <a name="are-there-any-computer-name-requirements"></a>是否有任何计算机名称要求？
是的。 计算机名称的最大长度为 64 个字符。 有关命名资源的详细信息，请参阅[命名约定规则和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。

## <a name="are-there-any-resource-group-name-requirements"></a>是否存在资源组名称要求？
是的。 资源组名称的最大长度为 90 个字符。 有关资源组的详细信息，请参阅[命名约定规则和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。

## <a name="what-are-the-username-requirements-when-creating-a-vm"></a>创建 VM 时，用户名有什么要求？

用户名的长度应为 1 到 32 个字符。

不允许使用以下用户名：

| | | | |
|-----------------|-----------|--------------------|----------|
| `administrator` | `admin`   | `user`             | `user1`  |
| `test`          | `user2`   | `test1`            | `user3`  |
| `admin1`        | `1`       | `123`              | `a`      |
| `actuser`       | `adm`     | `admin2`           | `aspnet` |
| `backup`        | `console` | `david`            | `guest`  |
| `john`          | `owner`   | `root`             | `server` |
| `sql`           | `support` | `support_388945a0` | `sys`    |
| `test2`         | `test3`   | `user4`            | `user5`  |
| `video`         |           |                    |          |

<!--MOONCAKE: `video` is tested by users which is invalid-->

## <a name="what-are-the-password-requirements-when-creating-a-vm"></a>创建 VM 时，密码有什么要求？

根据所使用的工具，有不同的密码长度要求：
 - 门户 - 12 到 72 个字符之间
 - PowerShell - 8 到 123 个字符之间
 - CLI - 12 到 123 个字符之间

密码还必须满足以下 4 个复杂性要求中的 3 个要求：

* 具有小写字符
* 具有大写字符
* 具有数字
* 具有特殊字符（正则表达式匹配 [\W_]）

不允许使用以下密码：

<table>
    <tr>
        <td style="text-align:center">abc@123</td>
        <td style="text-align:center">P@$$w0rd</td>
        <td style="text-align:center">P@ssw0rd</td>
        <td style="text-align:center">P@ssword123</td>
        <td style="text-align:center">Pa$$word</td>
    </tr>
    <tr>
        <td style="text-align:center">pass@word1</td>
        <td style="text-align:center">Password!</td>
        <td style="text-align:center">Password1</td>
        <td style="text-align:center">Password22</td>
        <td style="text-align:center">iloveyou!</td>
    </tr>
</table>

<!--Update_Description: update meta properties, wording update -->
