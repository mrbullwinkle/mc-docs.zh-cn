---
title: 在 Azure Database for MariaDB 中创建和管理只读副本
description: 本文介绍如何使用门户在 Azure Database for MariaDB 中设置和管理只读副本
author: WenJason
ms.author: v-jay
ms.service: mariadb
ms.topic: conceptual
origin.date: 07/12/2019
ms.date: 07/22/2019
ms.openlocfilehash: bd04dfadd7c5ed1c5a698b6234648c40ea432c36
ms.sourcegitcommit: 1dac7ad3194357472b9c0d554bf1362c391d1544
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2019
ms.locfileid: "68308961"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mariadb-using-the-azure-portal"></a>如何使用 Azure 门户在 Azure Database for MariaDB 中创建和管理只读副本

本文介绍如何使用 Azure 门户在 Azure Database for MariaDB 服务中创建和管理只读副本。

> [!IMPORTANT]
> 可以在主服务器所在的区域或所选的任何其他 Azure 区域创建只读副本。 只读副本（相同区域和跨区域）当前以公共预览版提供。

## <a name="prerequisites"></a>先决条件

- 将用作主服务器的 [Azure Database for MariaDB 服务器](quickstart-create-mariadb-server-database-using-azure-portal.md)。

> [!IMPORTANT]
> 只读副本功能仅适用于“常规用途”或“内存优化”定价层中的 Azure Database for MariaDB 服务器。 请确保主服务器位于其中一个定价层中。

## <a name="create-a-read-replica"></a>创建只读副本

可以使用以下步骤创建只读副本服务器：

1. 登录到 [Azure 门户](https://portal.azure.cn/)。

2. 选择要用作主服务器的现有 Azure Database for MariaDB 服务器。 此操作将打开“概述”  页。

3. 从菜单中的“设置”  下，选择“复制”  。

4. 选择“添加副本”  。

   ![Azure Database for MariaDB - 复制](./media/howto-read-replica-portal/add-replica.png)

5. 输入副本服务器的名称。

    ![Azure Database for MariaDB - 副本名称](./media/howto-read-replica-portal/replica-name.png)

6. 选择副本服务器的位置。 可以在任何 Azure 区域中创建副本。 默认位置与主服务器的位置相同。

    ![Azure Database for MariaDB - 副本位置](./media/howto-read-replica-portal/replica-location.png)

7. 选择“确定”以确认创建该副本。 

> [!NOTE]
> 只读副本使用与主服务器相同的服务器配置创建。 副本服务器配置在创建后可以更改。 建议副本服务器的配置应保持在与主服务器相同或更大的值，以确保副本能够跟上主服务器。

一旦创建副本服务器，可以从“复制”  边栏选项卡中进行查看。

   ![Azure Database for MariaDB - 列出副本](./media/howto-read-replica-portal/list-replica.png)

## <a name="stop-replication-to-a-replica-server"></a>停止复制到副本服务器

> [!IMPORTANT]
> 停止复制到服务器操作不可逆。 一旦主服务器和副本服务器之间的复制停止，无法撤消。 然后，副本服务器将成为独立服务器，并且现在支持读取和写入。 此服务器不能再次成为副本服务器。

若要从 Azure 门户停止主服务器和副本服务器之间的复制，请使用以下步骤：

1. 在 Azure 门户中，选择主 Azure Database for MariaDB 服务器。 

2. 从菜单中的“设置”  下，选择“复制”  。

3. 选择要停止复制的副本服务器。

   ![Azure Database for MariaDB - 停止复制选择服务器](./media/howto-read-replica-portal/stop-replication-select.png)

4. 选择“停止复制”  。

   ![Azure Database for MariaDB - 停止复制](./media/howto-read-replica-portal/stop-replication.png)

5. 通过单击“确定”  ，确认要停止复制。

   ![Azure Database for MariaDB - 停止复制确认](./media/howto-read-replica-portal/stop-replication-confirm.png)

## <a name="delete-a-replica-server"></a>删除副本服务器

若要从 Azure 门户删除只读副本服务器，请使用以下步骤：

1. 在 Azure 门户中，选择主 Azure Database for MariaDB 服务器。

2. 从菜单中的“设置”  下，选择“复制”  。

3. 选择要删除的副本服务器。

   ![Azure Database for MariaDB - 删除副本选择服务器](./media/howto-read-replica-portal/delete-replica-select.png)

4. 选择“删除副本” 

   ![Azure Database for MariaDB - 删除副本](./media/howto-read-replica-portal/delete-replica.png)

5. 键入副本的名称，然后单击“删除”  以确认删除副本。  

   ![Azure Database for MariaDB - 删除副本确认](./media/howto-read-replica-portal/delete-replica-confirm.png)

## <a name="delete-a-master-server"></a>删除主服务器

> [!IMPORTANT]
> 删除主服务器会停止复制到所有副本服务器，并删除主服务器本身。 副本服务器成为现在支持读取和写入的独立服务器。

若要从 Azure 门户删除主服务器，请使用以下步骤：

1. 在 Azure 门户中，选择主 Azure Database for MariaDB 服务器。

2. 从“概览”  中，选择“删除”  。

   ![Azure Database for MariaDB - 删除主服务器](./media/howto-read-replica-portal/delete-master-overview.png)

3. 键入主服务器的名称，然后单击“删除”  以确认删除主服务器。  

   ![Azure Database for MariaDB - 删除主服务器](./media/howto-read-replica-portal/delete-master-confirm.png)

## <a name="monitor-replication"></a>监视复制

1. 在 [Azure 门户](https://portal.azure.cn/)中，选择要监视的副本 Azure Database for MariaDB 服务器。

2. 在边栏的“监视”  部分，选择“指标”  ：

3. 从可用指标的下拉列表中选择“复制延迟(秒)”  。

   ![选择复制延迟时间](./media/howto-read-replica-portal/monitor-select-replication-lag.png)

4. 选择要查看的时间范围。 下图选择 30 分钟的时间范围。

   ![选择时间范围](./media/howto-read-replica-portal/monitor-replication-lag-time-range.png)

5. 查看所选时间范围的复制延迟时间。 下图显示了大型工作负荷的过去 30 分钟。

   ![选择时间范围](./media/howto-read-replica-portal/monitor-replication-lag-time-range-thirty-mins.png)

## <a name="next-steps"></a>后续步骤

- 详细了解[只读副本](concepts-read-replicas.md)