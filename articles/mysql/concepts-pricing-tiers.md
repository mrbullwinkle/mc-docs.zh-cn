---
title: Azure Database for MySQL 的定价层
description: 本文介绍 Azure Database for MySQL 的定价层。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.topic: conceptual
origin.date: 02/01/2019
ms.date: 07/15/2019
ms.openlocfilehash: b8ac2477fccf0b81d20eebb10182aa765b4bb5eb
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845123"
---
# <a name="azure-database-for-mysql-pricing-tiers"></a>Azure Database for MySQL 定价层

> [!NOTE]
> 将要查看的是 Azure Database for MySQL 的新服务。 若要查看经典 MySQL Database for Azure 的文档，请访问[此页](https://docs.azure.cn/zh-cn/mysql-database-on-azure/)。

可以在以下三个不同的定价层之一中创建 Azure Database for MySQL 服务器：“基本”、“常规用途”和“内存优化”。 定价层的差异表现在可以预配的 vCore 中的计算量、每个 vCore 的内存，以及用于存储数据的存储技术。 所有资源都在 MySQL 服务器级别预配。 一个服务器可以有一个或多个数据库。

|    | **基本** | **常规用途** | **内存优化** |
|:---|:----------|:--------------------|:---------------------|
| 计算的代 | 第 4 代、第 5 代 | 第 4 代、第 5 代 | 第 5 代 |
| vCore 数 | 1, 2 | 2, 4, 8, 16, 32, 64 |2, 4, 8, 16, 32 |
| 每个 vCore 的内存 | 2 GB | 5 GB | 10 GB |
| 存储大小 | 5 GB 到 1 TB | 5GB 到 4TB | 5GB 到 4TB |
| 存储类型 | Azure 标准存储 | Azure 高级存储 | Azure 高级存储 |
| 数据库备份保留期 | 7 到 35 天 | 7 到 35 天 | 7 到 35 天 |

可以从下表着手来选择定价层。

| 定价层 | 目标工作负荷 |
|:-------------|:-----------------|
| 基本 | 需要轻型计算和 I/O 性能的工作负荷。 示例包括用于开发或测试的服务器，或不常使用的小型应用程序。 |
| 常规用途 | 大多数业务工作负荷。此类工作负荷需要均衡的计算和内存以及可缩放的 I/O 吞吐量。 相关示例包括用于托管 Web 和移动应用的服务器，以及其他企业应用程序。|
| 内存优化 | 高性能数据库工作负荷。此类工作负荷需要内存中性能来实现更快的事务处理速度和更高的并发性。 相关示例包括用于处理实时数据的服务器，以及高性能事务性应用或分析应用。|

创建服务器后，只需数秒即可增加或减少 vCore 数、硬件生成和定价层（来回调整基本定价层除外）。 也可在不关闭应用程序的情况下，独立调整存储容量（向上调整）和备份保留期（上下调整）。 创建服务器之后，不能更改备份存储类型。 有关详细信息，请参阅[缩放资源](#scale-resources)部分。

## <a name="compute-generations-and-vcores"></a>计算代数和 vCore 数

计算资源以 vCore 的形式提供，代表基础硬件的逻辑 CPU。 目前提供两代计算（第 4 代和第 5 代）供你选择。 第 4 代逻辑 CPU 基于 Intel E5-2673 v3 (Haswell) 2.4-GHz 处理器。 第 5 代逻辑 CPU 基于 Intel E5-2673 v4 (Broadwell) 2.3-GHz 处理器。 可在以下区域获取第 4 代和第 5 代（“X”表示可用）。

| **Azure 区域** | **第 4 代** | **第 5 代** |
|:---|:----------:|:--------------------:|
| 中国东部 | X |  |
| 中国东部 2 |  | X |
| 中国北部 | X |  |
| 中国北部 2 |  | X |

## <a name="storage"></a>存储

预配的存储是指可供 Azure Database for MySQL 服务器使用的存储容量。 此存储用于数据库文件、临时文件、事务日志和 MySQL 服务器日志。 预配的总存储量也定义了可供服务器使用的 I/O 容量。

|    | **基本** | **常规用途** | **内存优化** |
|:---|:----------|:--------------------|:---------------------|
| 存储类型 | Azure 标准存储 | Azure 高级存储 | Azure 高级存储 |
| 存储大小 | 5 GB 到 1 TB | 5GB 到 4TB | 5GB 到 4TB |
| 存储增量大小 | 1 GB | 1 GB | 1 GB |
| IOPS | 变量 |3 IOPS/GB<br/>至少 100 IOPS<br/>最大 6000 IOPS | 3 IOPS/GB<br/>至少 100 IOPS<br/>最大 6000 IOPS |

在创建服务器的过程中和之后，可以添加更多的存储容量，这样系统就可以根据工作负荷的存储使用情况自动增加存储。 “基本”层不提供 IOPS 保证。 在“常规用途”和“内存优化”定价层中，IOPS 与预配的存储大小按 3:1 的比例缩放。

可以通过 Azure 门户或 Azure CLI 命令监视 I/O 使用情况。 要监视的相关指标是[存储上限、存储百分比、已用存储和 IO 百分比](concepts-monitoring.md)。

### <a name="reaching-the-storage-limit"></a>达到存储限制

对于预配存储不到 100 GB 的服务器，如果可用存储低于 512MB 或 5% 的预配存储大小，则会将该服务器标记为只读。 对于预配存储超出 100 GB 的服务器，当可用存储不到 5 GB 时，会将该服务器标记为只读。

例如，如果已预配 110 GB 的存储，而实际使用量超过 105 GB，则会将服务器标记为只读。 或者，如果已预配 5 GB 的存储，则当可用存储少于 512 MB 时，服务器会标记为只读。

当服务试图将服务器标记为只读时，会阻止所有新的写入事务请求，现有的活动事务将继续执行。 当服务器设置为只读时，所有后续写入操作和事务提交均会失败。 读取查询将继续不间断工作。 增加预配的存储后，服务器将准备好再次接受写入事务。

我们建议你启用存储自动增长或设置警报，以便在服务器存储接近阈值时通知你，避免进入只读状态。 有关详细信息，请参阅有关[如何设置警报](howto-alert-on-metric.md)的文档。

### <a name="storage-auto-grow"></a>存储自动增长

如果启用了存储自动增长，存储会在不影响工作负荷的情况下自动增长。 对于预配的存储大小小于 100 GB 的服务器，可用存储空间一小于 1 GB 或预配的存储的 10%，预配的存储大小就会立即增加 5 GB。 对于预配的存储大小大于 100 GB 的服务器，可用存储空间一小于预配的存储大小的 5%，预配的存储大小就会立即增加 5%。 适用上面指定的最大存储限制。

例如，如果已预配 1000 GB 的存储，而实际使用量超过 950 GB，则服务器存储大小会增加到 1050 GB。 或者，如果已预配 10 GB 的存储，则当可用存储少于 1 GB 时，存储大小会增加到 15 GB。

## <a name="backup"></a>Backup

服务自动对服务器进行备份。 备份的最短保留期为七天。 可以设置长达 35 天的保留期。 可以在服务器的生存期内随时对保留期进行调整。 可以在本地冗余备份和异地冗余备份之间进行选择。 异地冗余备份也存储在创建服务器时所在区域的异地配对区域中。 这种冗余可以在发生灾难时提供一定级别的保护。 也可获得将服务器还原到任何其他 Azure 区域的功能，前提是该区域提供的服务带有异地冗余备份。 创建服务器后，无法在这两个备份存储选项之间进行更改。

## <a name="scale-resources"></a>缩放资源

创建服务器之后，可以独立地更改 vCore 数、硬件生成、定价层（基本层的操作除外）、存储量和备份保留期。 创建服务器之后，不能更改备份存储类型。 可以向上或向下调整 VCore 数。 备份保留期可以从 7 天到 35 天进行上下调整。 存储大小只能增加。 可以通过门户或 Azure CLI 缩放资源。 有关使用 Azure CLI 进行缩放的示例，请参阅[使用 Azure CLI 监视和缩放 Azure Database for MySQL 服务器](scripts/sample-scale-server.md)。

更改 vCore 数、硬件生成或定价层时，将会使用新的计算分配创建原始服务器的副本。 启动并运行新服务器后，连接将切换到新服务器。 在系统切换到新服务器的短暂期间，无法建立新的连接，所有未提交的连接将会回退。 此时段不定，但大多数情况下短于一分钟。

缩放存储和更改备份保留期是真正的联机操作。 不会造成停机，应用程序不会受影响。 当 IOPS 随已预配存储的大小缩放时，可以通过扩大存储来增加提供给服务器的 IOPS。

## <a name="pricing"></a>定价

有关最新定价信息，请参阅服务的[定价页](https://www.azure.cn/zh-cn/pricing/details/mysql/)。 若要查看所需配置的具体成本，可以单击 [Azure 门户](https://portal.azure.cn/#create/Microsoft.MySQLServer)的“定价层”选项卡，系统就会根据选定的选项显示每月成本。  如果没有 Azure 订阅，可使用 Azure 定价计算器获取估计的价格。 在 [Azure 定价计算器](https://azure.cn/pricing/calculator/)网站上，选择“添加项”  ，展开“数据库”  类别，选择“Azure Database for MySQL”  自定义选项。

## <a name="next-steps"></a>后续步骤

- 了解如何[在门户中创建 MySQL 服务器](howto-create-manage-server-portal.md)。
- 了解[服务限制](concepts-limits.md)。
- 了解如何[使用只读副本进行横向扩展](howto-read-replicas-portal.md)。
