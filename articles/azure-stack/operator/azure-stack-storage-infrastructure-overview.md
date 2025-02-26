---
title: 管理 Azure Stack 的存储基础结构 | Microsoft Docs
description: 管理 Azure Stack 的存储基础结构。
services: azure-stack
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: ''
ms.topic: article
origin.date: 03/11/2019
ms.date: 07/29/2019
ms.author: v-jay
ms.lastreviewed: 03/11/2019
ms.reviewer: jiahan
ms.openlocfilehash: b76ed16377777d33e3e29fbfdc58f5210bb6dacc
ms.sourcegitcommit: 4d34571d65d908124039b734ddc51091122fa2bf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68513389"
---
# <a name="manage-storage-infrastructure-for-azure-stack"></a>管理 Azure Stack 的存储基础结构

*适用于：Azure Stack 集成系统和 Azure Stack 开发工具包*

本文介绍 Azure Stack 存储基础结构资源的运行状况和工作状态。 这些资源包括存储驱动器和卷。 尝试排查各种问题（例如，无法将驱动器添加到池）时，本主题中的信息可能非常有参考价值。

## <a name="understand-drives-and-volumes"></a>了解驱动器和卷

### <a name="drives"></a>驱动器

由 Windows Server 软件驱动的 Azure Stack 定义了存储功能，其中包括存储空间直通 (S2D) 和 Windows Server 故障转移群集的组合，以提供高性能、可缩放且具有复原能力的存储服务。

Azure Stack 集成式系统合作伙伴提供众多的解决方案版本，包括各种灵活的存储。 目前，可以选择三种驱动器的组合：NVMe（非易失性快速存储器）、SATA/SAS SSD（固态硬盘）和 HDD（机械硬盘）。

存储空间直通提供缓存用于最大化存储性能。 在采用一种或多种驱动器的 Azure Stack 设备中，存储空间直通自动使用“最快的”(NVMe &gt; SSD &gt; HDD) 的所有驱动器类型进行缓存。 剩余的驱动器用于提供容量。 驱动器可以分组成“全部闪存”或“混合”部署：

![Azure Stack 存储基础结构](media/azure-stack-storage-infrastructure-overview/image1.png)

“全部闪存”部署旨在最大化存储性能，不包括机械硬盘 (HDD)。

![Azure Stack 存储基础结构](media/azure-stack-storage-infrastructure-overview/image2.png)

混合部署旨在平衡性能和容量或者最大化容量，包括机械硬盘 (HDD)。

根据要提供缓存的驱动器类型自动确定缓存的行为。 为固态硬盘提供缓存时（例如，为 SSD 提供 NVMe 缓存），只缓存写入内容。 这可以减小容量驱动器的磨损，减少发往容量驱动器的累积流量，延长驱动器的寿命。 同时，由于读取操作不会严重影响闪存的寿命，并且固态硬盘通常提供较低的读取延迟，因此，不会缓存读取内容。 为机械硬盘提供缓存时（例如，为 HDD 提供 SSD 缓存），会同时缓存读取和写入内容，以便为读取和写入操作提供类似于闪存的延迟（通常可将延迟改善大约 10 倍）。

![Azure Stack 存储基础结构](media/azure-stack-storage-infrastructure-overview/image3.png)

> [!Note]  
> 可以在同时采用 HDD 和 SSD（或 NVMe）驱动器的混合部署中提供 Azure Stack 设备。 但是，速度较快的驱动器类型将用作缓存驱动器，所有剩余驱动器将以池的形式用作容量驱动器。 租户数据（Blob、表、队列和磁盘）放在容量驱动器上。 因此，预配高级磁盘或选择高级存储帐户类型并不意味着保证将对象分配到 SSD 或 NVMe 驱动器并获取更高的性能。

### <a name="volumes"></a>卷

存储服务将可用的存储分区成独立的卷，这些卷可分配用于保存系统数据和租户数据。  卷将驱动器合并到存储池中，带来存储空间直通的容错、可伸缩性和性能优势。

![Azure Stack 存储基础结构](media/azure-stack-storage-infrastructure-overview/image4.png)

可在 Azure Stack 存储池中创建三种类型的卷：

-   基础结构卷：托管 Azure Stack 基础结构 VM 和核心服务使用的文件。

-   VM 临时卷：托管附加到租户 VM 的临时磁盘以及这些磁盘中存储的数据。

-   对象存储卷：托管租户数据服务 Blob、表、队列和 VM 磁盘。

在多节点部署中可以看到三个基础结构卷，VM 临时卷和对象存储卷的数量等于 Azure Stack 部署中的节点数量：

-   在四节点部署中，有四个等量的 VM 临时卷和四个等量的对象存储卷。

-   如果将新节点添加到群集，则会为这两种资源类型创建新卷。

-   即使节点出现故障或者被删除，卷数也会保持相同。

-   如果使用 Azure Stack 开发人员包，则会创建包含多个共享的单个卷。

存储空间直通中的卷提供复原能力来防范硬件问题（例如驱动器或服务器故障），并在整个服务器维护期间（例如软件更新）实现持续可用性。 Azure Stack 部署使用三向镜像来确保数据复原能力。 租户数据的三个副本将写入不同的服务器并保存在缓存中：

![Azure Stack 存储基础结构](media/azure-stack-storage-infrastructure-overview/image5.png)

镜像功能通过保存所有数据的多个副本来提供容错。 这些数据的条带化和放置方式非常重要（请参阅此博客了解详细信息），但肯定的是，使用镜像功能存储的任何数据都会完整地写入多次。 每个副本将写入不同的物理硬件（位于不同服务器中的不同驱动器），假设每个硬盘各自都有可能发生故障。 三向镜像可以安全容许至少两个硬件（驱动器或服务器）同时出现问题。 例如，如果你正在重新启动一台服务器，此时另一个驱动器或服务器突然发生故障，在这种情况下，所有数据将保持安全，可供持续访问。

## <a name="volume-states"></a>卷状态

若要确定卷所处的状态，请使用以下 PowerShell 命令：

```powershell
$scaleunit_name = (Get-AzsScaleUnit)[0].name

$subsystem_name = (Get-AzsStorageSubSystem -ScaleUnit $scaleunit_name)[0].name

Get-AzsVolume -ScaleUnit $scaleunit_name -StorageSubSystem $subsystem_name | Select-Object VolumeLabel, HealthStatus, OperationalStatus, RepairStatus, Description, Action, TotalCapacityGB, RemainingCapacityGB
```

以下示例输出显示有一个卷已分离，并且有一个卷已降级/不完整：

| VolumeLabel | HealthStatus | OperationalStatus |
|-------------|--------------|------------------------|
| ObjStore_1 | Unknown | 已分离 |
| ObjStore_2 | 警告 | {已降级，不完整} |

以下部分列出了运行状况和工作状态。

### <a name="volume-health-state-healthy"></a>卷运行状况：Healthy

| 操作状态 | 说明 |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| OK | 卷处于正常状态。 |
| 欠佳 | 数据未均匀写入到各个驱动器。<br> <br>**操作：** 请联系支持人员优化存储池中的驱动器用法。 在此之前，请参考 https://docs.azure.cn/zh-cn/azure-stack/azure-stack-diagnostics#log-collection-tool 中的指导启动日志文件收集过程。 恢复失败的连接后，可能需要从备份还原数据。 |


### <a name="volume-health-state-warning"></a>卷运行状况：警告

如果卷处于“警告”运行状况，则表示数据的一个或多个副本不可用，但 Azure Stack 仍可读取至少一个数据副本。

| 操作状态 | 说明 |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 运行中 | Azure Stack 正在修复卷（例如，在添加或删除驱动器之后）。 修复完成后，卷应该会恢复“正常”运行状况。<br> <br>**操作：** 等待 Azure Stack 完成修复卷，然后检查状态。 |
| 不完整 | 由于一个或多个驱动器出现故障或缺失，卷的复原能力下降。 但是，缺失的驱动器包含数据的最新副本。<br> <br>**操作：** 重新连接所有缺失的驱动器，更换所有出现故障的驱动器，并使脱机的所有服务器联机。 |
| 已降级 | 由于一个或多个驱动器出现故障或缺失，并且这些驱动器包含已过时的数据副本，因此卷的复原能力下降。<br> <br>**操作：** 重新连接所有缺失的驱动器，更换所有出现故障的驱动器，并使脱机的所有服务器联机。 |

 

### <a name="volume-health-state-unhealthy"></a>卷运行状况：不正常

如果某个卷处于“不正常”运行状况，该卷上的部分或所有数据当前将不可访问。

| 操作状态 | 说明 |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 无冗余 | 由于过多的驱动器出现故障，该卷已丢失数据。<br> <br>**操作：** 请联系支持人员。 在此之前，请参考 https://docs.azure.cn/zh-cn/azure-stack/azure-stack-diagnostics#log-collection-tool 中的指导启动日志文件收集过程。 |


### <a name="volume-health-state-unknown"></a>卷运行状况：Unknown

如果虚拟磁盘已分离，卷也有可能处于“未知”运行状况。

| 操作状态 | 说明 |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 已分离 | 某个存储设备出现故障，从而可能导致卷不可访问。 某些数据可能已丢失。<br> <br>**操作：** <br>1.检查所有存储设备的物理连接和网络连接。<br>2.如果所有设备连接正确，请联系支持人员。 在此之前，请参考 https://docs.azure.cn/zh-cn/azure-stack/azure-stack-diagnostics#log-collection-tool 中的指导启动日志文件收集过程。 恢复失败的连接后，可能需要从备份还原数据。 |

## <a name="drive-states"></a>驱动器状态

使用以下 PowerShell 命令监视驱动器的状态：

```powershell
$scaleunit_name = (Get-AzsScaleUnit)[0].name

$subsystem_name = (Get-AzsStorageSubSystem -ScaleUnit $scaleunit_name)[0].name

Get-AzsDrive -ScaleUnit $scaleunit_name -StorageSubSystem $subsystem_name | Select-Object StorageNode, PhysicalLocation, HealthStatus, OperationalStatus, Description, Action, Usage, CanPool, CannotPoolReason, SerialNumber, Model, MediaType, CapacityGB
```

以下部分描述了驱动器可能处于的运行状况。

### <a name="drive-health-state-healthy"></a>驱动器运行状况：Healthy

| 操作状态 | 说明 |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| OK | 卷处于正常状态。 |
| 运行中 | 驱动器正在执行某些内部保养操作。 操作完成后，驱动器应会恢复“正常”运行状况。 |

### <a name="drive-health-state-healthy"></a>驱动器运行状况：Healthy

处于“警告”状态的驱动器可以成功读取和写入数据，但存在问题。

| 操作状态 | 说明 |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 通信断开 | 与驱动器的连接已断开。<br> <br>**操作：** 使所有服务器恢复联机。 如果仍未解决问题，请重新连接驱动器。 如果此问题持续发生，请更换驱动器，以确保能够全面复原。 |
| 预测性故障 | 预测驱动器即将发生故障。<br> <br>**操作：** 请尽快更换驱动器，以确保能够全面复原。 |
| IO 错误 | 访问驱动器时发生暂时性错误。<br> <br>**操作：** 如果此问题持续发生，请更换驱动器，以确保能够全面复原。 |
| 暂时性错误 | 驱动器出现暂时性错误。 这通常表示驱动器无响应，但也可能表示不恰当地从驱动器中删除了存储空间直通的保护分区。 <br> <br>**操作：** 如果此问题持续发生，请更换驱动器，以确保能够全面复原。 |
| 异常延迟 | 驱动器有时无响应并出现故障迹象。<br> <br>**操作：** 如果此问题持续发生，请更换驱动器，以确保能够全面复原。 |
| 从池中删除 | Azure Stack 正在从其存储池中删除驱动器。<br> <br>**操作：** 等待 Azure Stack 完成删除驱动器，然后检查状态。<br>如果仍旧出现此状态，请联系支持人员。 在此之前，请参考 https://docs.azure.cn/zh-cn/azure-stack/azure-stack-diagnostics#log-collection-tool 中的指导启动日志文件收集过程。 |
|  |  |
| 启动维护模式 | Azure Stack 正在将驱动器置于维护模式。 这是一种暂时性的状态 - 驱动器应该很快就会处于“维护中模式”状态。<br> <br>**操作：** 等待 Azure Stack 完成该过程，然后检查状态。 |
| 维护中模式 | 驱动器处于维护模式，因此对其的读取和写入操作已停止。 这通常表示正在执行 Azure Stack 管理任务，例如，PNU 或 FRU 正在操作驱动器。 但是，管理员也可将驱动器置于维护模式。<br> <br>**操作：** 等待 Azure Stack 完成管理任务，然后检查状态。<br>如果仍旧出现此状态，请联系支持人员。 在此之前，请参考 https://docs.azure.cn/zh-cn/azure-stack/azure-stack-diagnostics#log-collection-tool 中的指导启动日志文件收集过程。 |
|  |  |
| 停止维护模式 | Azure Stack 正在将驱动器恢复联机。 这是一种暂时性的状态 - 驱动器应该很快就会处于另一种状态，最好是“正常运行”状态。<br> <br>**操作：** 等待 Azure Stack 完成该过程，然后检查状态。 |

 

### <a name="drive-health-state-unhealthy"></a>驱动器运行状况：不正常

当前无法写入或访问处于“不正常”状态的驱动器。

| 操作状态 | 说明 |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 拆分 | 已从池中分离驱动器。<br> <br>**操作：** 使用新磁盘替换该驱动器。 如果必须使用此磁盘，请从系统中删除该磁盘，确保该磁盘上没有任何有用的数据，擦除该磁盘，然后重新安装磁盘。 |
| 不可用 | 物理磁盘已被隔离，因为它不受解决方案供应商的支持。 仅支持批准用于解决方案并且包含正确磁盘固件的磁盘。<br> <br>**操作：** 将该驱动器替换为由解决方案批准的制造商提供的且带有认可型号的磁盘。 |
| 已过时的元数据 | 用于更换的磁盘之前已用过，可能包含未知存储系统中的数据。 该磁盘已被隔离。        <br> <br>**操作：** 使用新磁盘替换该驱动器。 如果必须使用此磁盘，请从系统中删除该磁盘，确保该磁盘上没有任何有用的数据，擦除该磁盘，然后重新安装磁盘。 |
| 无法识别的元数据 | 在驱动器上找到了无法识别的元数据，这通常表示驱动器包含来自其他池的元数据。<br> <br>**操作：** 使用新磁盘替换该驱动器。 如果必须使用此磁盘，请从系统中删除该磁盘，确保该磁盘上没有任何有用的数据，擦除该磁盘，然后重新安装磁盘。 |
| 介质故障 | 驱动器出现故障，不再可供存储空间使用。<br> <br>**操作：** 请尽快更换驱动器，以确保能够全面复原。 |
| 设备硬件故障 | 此驱动器上出现硬件故障。 <br> <br>**操作：** 请尽快更换驱动器，以确保能够全面复原。 |
| 正在更新固件 | Azure Stack 正在更新驱动器上的固件。 这是一种暂时性的状态，其持续时间通常小于一分钟，在此期间，池中的其他驱动器会处理所有读取和写入操作。<br> <br>**操作：** 等待 Azure Stack 完成更新，然后检查状态。 |
| 正在启动 | 驱动器正在为操作做好准备。 这应该是一种暂时性的状态 - 完成后，驱动器应会过渡到另一种工作状态。<br> <br>**操作：** 等待 Azure Stack 完成操作，然后检查状态。 |
 

## <a name="reasons-a-drive-cant-be-pooled"></a>驱动器无法入池的原因

某些驱动器尚未做好加入 Azure Stack 存储池的准备。 通过查看驱动器的 CannotPoolReason 属性，可以确定驱动器为何不符合入池条件的原因。 下表更具体地描述了每种原因。

| Reason | 说明 |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 硬件不合规 | 使用运行状况服务指定的已批准存储模型列表中不包括该驱动程序。<br> <br>**操作：** 使用新磁盘替换该驱动器。 |
| 固件不合规 | 使用运行状况服务指定的已批准固件修订版列表中不包括该物理驱动器上的固件。<br> <br>**操作：** 使用新磁盘替换该驱动器。 |
| 已由群集使用 | 该驱动器当前已由故障转移群集使用。<br> <br>**操作：** 使用新磁盘替换该驱动器。 |
| 可移动媒体 | 该驱动器分类为可移动驱动器。 <br> <br>**操作：** 使用新磁盘替换该驱动器。 |
| 不正常 | 该驱动器不处于正常状态，可能需要更换。<br> <br>**操作：** 使用新磁盘替换该驱动器。 |
| 容量不足 | 某些分区占用了驱动器上的可用空间。<br> <br>**操作：** 使用新磁盘替换该驱动器。 如果必须使用此磁盘，请从系统中删除该磁盘，确保该磁盘上没有任何有用的数据，擦除该磁盘，然后重新安装磁盘。 |
| 正在验证 | 运行状况服务正在检查是否已批准使用驱动器上的固件。<br> <br>**操作：** 等待 Azure Stack 完成该过程，然后检查状态。 |
| 验证失败 | 运行状况服务无法检查是否已批准使用驱动器上的固件。<br> <br>**操作：** 请联系支持人员。 在此之前，请参考 https://docs.azure.cn/zh-cn/azure-stack/azure-stack-diagnostics#log-collection-tool 中的指导启动日志文件收集过程。 |
| 脱机 | 驱动器已脱机。 <br> <br>**操作：** 请联系支持人员。 在此之前，请参考 https://docs.azure.cn/zh-cn/azure-stack/azure-stack-diagnostics#log-collection-tool 中的指导启动日志文件收集过程。 |

## <a name="next-step"></a>后续步骤

[管理存储容量](azure-stack-manage-storage-shares.md) 