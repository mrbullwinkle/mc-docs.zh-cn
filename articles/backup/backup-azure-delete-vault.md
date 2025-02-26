---
title: 删除 Azure 中的恢复服务保管库
description: 介绍如何删除恢复服务保管库。
services: backup
author: lingliw
manager: digimobile
ms.service: backup
ms.topic: conceptual
ms.date: 07/11/2019
ms.author: v-lingwu
ms.openlocfilehash: ff894fc59c93ff11116cfa7c617a3b6c6d699f87
ms.sourcegitcommit: 68f7c41974143a8f7bd9b7a54acf41c09893e587
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68332212"
---
# <a name="delete-a-recovery-services-vault"></a>删除恢复服务保管库

本文介绍如何删除 [Azure 备份](backup-overview.md)恢复服务保管库。 其中分别说明了如何删除依赖项，以及如何删除保管库。


## <a name="before-you-start"></a>开始之前

无法删除具有依赖项（例如，与保管库关联的受保护服务器或备份管理服务器）的恢复服务保管库。<br/>
无法删除包含备份数据的保管库（即，停止了保护，但保留了备份数据）。

如果删除包含依赖项的保管库，将显示以下错误：

![删除保管库错误](./media/backup-azure-delete-vault/error.png)

若要正常删除保管库，请按以下顺序遵循所述的步骤：
- 停止保护并删除备份数据
- 删除受保护的服务器或备份管理服务器
- 删除保管库


## <a name="delete-backup-data-and-backup-items"></a>删除备份数据和备份项

在继续操作之前，请阅读 **[此部分](#before-you-start)** ，以了解依赖项和保管库删除过程。

### <a name="for-protected-items-in-cloud"></a>对于云中的受保护项

若要停止保护并删除备份数据，请执行以下操作：

1. 在门户中导航到“恢复服务保管库”>“备份项”，并选择云中的受保护项。

    ![选择备份类型](./media/backup-azure-delete-vault/azure-storage-selected.jpg)

2. 对于每个项，需要单击右键并选择“停止备份”。 

    ![选择备份类型](./media/backup-azure-delete-vault/stop-backup-item.png)

3. 在“停止备份” > “选择选项”中，选择“删除备份数据”。   
4. 键入项的名称，然后单击“停止备份”。 
   - 这表示你确认要删除该项。
   - 确认后，“停止备份”按钮将会激活。 
   - 如果保留（而不是删除）该数据，便无法删除保管库。

     ![删除备份数据](./media/backup-azure-delete-vault/stop-backup-blade-delete-backup-data.png)

5. 检查“通知”![删除备份数据](./media/backup-azure-delete-vault/messages.png)。  完成后，服务将显示以下消息：“正在停止 <备份项> 的备份并删除备份数据。  已成功完成该操作”。 
6. 单击“备份项”菜单中的“刷新”，以检查是否删除了备份项。  

      ![删除备份数据](./media/backup-azure-delete-vault/empty-items-list.png)

### <a name="for-mars-agent"></a>对于 MARS 代理

若要停止保护并删除备份数据，请按下列顺序执行步骤：

- [步骤 1：从 MARS 管理控制台中删除备份项](#step-1-delete-backup-items-from-mars-management-console)
- [步骤 2：从门户中删除 Azure 备份代理](#step-1-delete-backup-items-from-mars-management-console)


#### 步骤 1：从 MARS 管理控制台中删除备份项 <a name="step-1-delete-backup-items-from-mars-management-console"></a>

如果由于服务器不可用而无法执行此步骤，请与 Microsoft 支持部门联系。

- 启动 MARS 管理控制台，转到“操作”窗格并选择“计划备份”   。
- 从“修改或停止计划的备份”向导中选择“停止使用此备份计划并删除所有存储的备份”选项，然后单击“下一步”    。

    ![修改或停止计划的备份](./media/backup-azure-delete-vault/modify-schedule-backup.png)

- 在“停止计划的备份”向导中单击“完成”。  

    ![停止计划的备份](./media/backup-azure-delete-vault/stop-schedule-backup.png)
- 系统会提示输入安全 PIN。 若要生成该 PIN，请执行以下步骤：
  - 登录到 Azure 门户。
  - 浏览到“恢复服务保管库”   > “设置”   >   “属性”。
  - 单击“安全 PIN”下的“生成”   。 复制此 PIN。（此 PIN 的有效时间仅为五分钟）
- 在管理控制台（客户端应用）中粘贴此 PIN 并单击“确定”。 

  ![安全 PIN](./media/backup-azure-delete-vault/security-pin.png)

- 在“修改备份进度”向导中，  会看到“删除的备份数据会保留 14 天。  该时间过后，备份数据会被永久删除。”  

    ![删除备份基础结构](./media/backup-azure-delete-vault/deleted-backup-data.png)

删除本地的备份项以后，请在门户中完成后续步骤。

#### 步骤 2：从门户中删除 Azure 备份代理 <a name="step-2-from-portal-remove-azure-backup-management-servers"></a>

在继续操作之前，请确保已完成[步骤 1](#step-1-delete-backup-items-from-mars-management-console)：

1. 在保管库仪表板菜单中，单击“备份基础结构”  。
2. 单击“受保护服务器”以查看基础结构服务器。 

    ![选择保管库，以打开它的仪表板](./media/backup-azure-delete-vault/identify-protected-servers.png)

3. 在“受保护的服务器”  列表中，单击“Azure 备份代理”。

    ![选择备份类型](./media/backup-azure-delete-vault/list-of-protected-server-types.png)

4. 在使用 Azure 备份代理保护的服务器列表中单击该服务器。

    ![选择受保护的具体服务器](./media/backup-azure-delete-vault/azure-backup-agent-protected-servers.png)

5. 在选定服务器的仪表板上，单击“删除”  。

    ![删除选定服务器](./media/backup-azure-delete-vault/selected-protected-server-click-delete.png)

6. 在“删除”  菜单上，键入服务器名称，再单击“删除”  。

     ![删除备份数据](./media/backup-azure-delete-vault/delete-protected-server-dialog.png)

> [!NOTE]
> 如果看到以下错误，则首先执行[从管理控制台中删除备份项](#step-1-delete-backup-items-from-mars-management-console)中列出的步骤。
>
>![删除失败](./media/backup-azure-delete-vault/deletion-failed.png)
>
>例如，如果无法从管理控制台执行删除备份的步骤（例如，由于服务器不可用），请与 Microsoft 支持部门联系。

7. 检查“通知”![删除备份数据](./media/backup-azure-delete-vault/messages.png)。  完成后，服务将显示以下消息：“正在停止 <备份项> 的备份并删除备份数据。  已成功完成该操作”。 
8. 单击“备份项”菜单中的“刷新”，以检查是否删除了备份项。  


### <a name="for-mabs-agent"></a>对于 MABS 代理

若要停止保护并删除备份数据，请按下列顺序执行步骤：

- [步骤 1：从 MABS 管理控制台中删除备份项](#step-1-delete-backup-items-from-mabs-management-console)
- [步骤 2：从门户删除 Azure 备份管理服务器](#step-2-from-portal-remove-azure-backup-agent)

#### <a name="step-1-delete-backup-items-from-mabs-management-console"></a>步骤 1：从 MABS 管理控制台中删除备份项

如果由于服务器不可用而无法执行此步骤，请与 Microsoft 支持部门联系。

**方法 1**：若要停止保护并删除备份数据，请执行以下步骤：

1.  在 DPM 管理员控制台中，单击导航栏上的“保护”  。
2.  在显示窗格中，选择要删除的保护组成员。 通过右键单击选择“停止对组成员的保护”选项。 
3.  在“停止保护”对话框中，选择“删除受保护的数据” > “删除在线存储”复选框，然后单击“停止保护”。    

    ![删除在线存储](./media/backup-azure-delete-vault/delete-storage-online.png)

受保护成员状态现在更改为“停用副本可用”。 

5. 右键单击停用保护组，然后选择“删除停用保护”。 

    ![删除停用保护](./media/backup-azure-delete-vault/remove-inactive-protection.png)

6. 在“删除停用保护”窗口中，选择“删除在线存储”，然后单击“确定”。   

    ![删除磁盘上的和联机的副本](./media/backup-azure-delete-vault/remove-replica-on-disk-and-online.png)

**方法 2**：启动 **MABS 管理**控制台。 在“选择数据保护方法”部分，取消选择“我需要在线保护”。  

  ![选择数据保护方法](./media/backup-azure-delete-vault/data-protection-method.png)

删除本地的备份项以后，请在门户中完成后续步骤。

#### 步骤 2：从门户删除 Azure 备份管理服务器 <a name="step-2-from-portal-remove-azure-backup-agent"></a>

在继续操作之前，请确保已完成[步骤 1](#step-1-delete-backup-items-from-mabs-management-console)：

1. 在保管库仪表板菜单中，单击“备份基础结构”  。
2. 单击“备份管理服务器”以查看服务器。 

    ![选择保管库，打开其仪表板](./media/backup-azure-delete-vault/delete-backup-management-servers.png)

3. 右键单击该项并选择“删除”。 
4. 在“删除”  菜单上，键入服务器名称，再单击“删除”  。

     ![删除备份数据](./media/backup-azure-delete-vault/delete-protected-server-dialog.png)

> [!NOTE]
> 如果看到以下错误，则首先执行[从管理控制台中删除备份项](#step-2-from-portal-remove-azure-backup-management-servers)中列出的步骤。
>
>![删除失败](./media/backup-azure-delete-vault/deletion-failed.png)
>
> 例如，如果无法从管理控制台执行删除备份的步骤（例如，由于服务器不可用），请与 Microsoft 支持部门联系。

5. 检查“通知”![删除备份数据](./media/backup-azure-delete-vault/messages.png)。  完成后，服务将显示以下消息：“正在停止 <备份项> 的备份并删除备份数据。  已成功完成该操作”。 
6. 单击“备份项”菜单中的“刷新”，以检查是否删除了备份项。  


## <a name="delete-the-recovery-services-vault"></a>删除恢复服务保管库

1. 删除所有依赖项后，在保管库菜单中滚动到“概要”窗格。 
2. 确认是否未列出任何“备份项”、“备份管理服务器”或“复制的项”。    如果保管库中仍有项，请[删除](#delete-backup-data-and-backup-items)这些项。

3. 当保管库中没有其他任何项时，请单击保管库仪表板上的“删除”  。

    ![删除备份数据](./media/backup-azure-delete-vault/vault-ready-to-delete.png)

4. 若确认要删除保管库，请单击“是”  。 随即会删除该保管库，门户返回“新建”服务菜单。 

## <a name="delete-the-recovery-services-vault-using-azure-resource-manager-client"></a>使用 Azure 资源管理器客户端删除恢复服务保管库

仅当已删除所有依赖项后，仍然收到“保管库删除错误”时，才建议使用删除恢复服务保管库的选项。 



- 在“概要”窗格中的保管库菜单内，确认是否未列出任何“备份项”、“备份管理服务器”或“复制的项”。     如果有备份项，请执行[删除备份数据和备份项](#delete-backup-data-and-backup-items)中所述的步骤。
- 重试[从门户删除保管库](#delete-the-recovery-services-vault)。
- 如果删除了所有依赖项，但仍收到“保管库删除错误”，请使用 ARMClient 工具执行以下步骤： 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

1. 从[此处](https://chocolatey.org/)安装 chocolatey，若要安装 ARMClient，请运行以下命令：

   `choco install armclient --source=https://chocolatey.org/api/v2/`
2. 登录到 Azure 帐户并运行以下命令：

    `ARMClient.exe login [environment name]`

3. 在 Azure 门户中，收集所要删除的保管库的订阅 ID 和资源组名称。

有关 ARMClient 命令的详细信息，请参阅此[文档](https://github.com/projectkudu/ARMClient/blob/master/README.md)。

### <a name="use-azure-resource-manager-client-to-delete-recovery-services-vault"></a>使用 Azure 资源管理器客户端删除恢复服务保管库

1. 使用订阅 ID、资源组名称和保管库名称运行以下命令。 如果没有任何依赖项，则运行该命令会删除保管库。

   ```
   ARMClient.exe delete /subscriptions/<subscriptionID>/resourceGroups/<resourcegroupname>/providers/Microsoft.RecoveryServices/vaults/<recovery services vault name>?api-version=2015-03-15
   ```
2. 如果该保管库不是空的，则会出现错误“由于此保管库中存在现有资源，因此无法将其删除”。 若要删除保管库中受保护的项/容器，请执行以下操作：

   ```
   ARMClient.exe delete /subscriptions/<subscriptionID>/resourceGroups/<resourcegroupname>/providers/Microsoft.RecoveryServices/vaults/<recovery services vault name>/registeredIdentities/<container name>?api-version=2016-06-01
   ```

3. 在 Azure 门户中，确认要删除该保管库。


## <a name="next-steps"></a>后续步骤

[了解](backup-azure-recovery-services-vault-overview.md)恢复服务保管库。<br/>
[了解](backup-azure-manage-windows-server.md)如何监视和管理恢复服务保管库。
