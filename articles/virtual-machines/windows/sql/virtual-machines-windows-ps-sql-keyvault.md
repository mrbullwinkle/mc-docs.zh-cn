---
title: 将 Key Vault 与 Azure 中 Windows VM 上的 SQL Server 集成 (Resource Manager) | Azure
description: 了解如何自动配置用于 Azure 密钥保管库的 SQL Server 加密。 本主题说明如何将 Azure 密钥保管库集成用于通过 Resource Manager 创建的 SQL Server 虚拟机。
services: virtual-machines-windows
documentationcenter: ''
author: rockboyfor
manager: digimobile
editor: ''
tags: azure-service-management
ms.assetid: cd66dfb1-0e9b-4fb0-a471-9deaf4ab4ab8
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
origin.date: 04/30/2018
ms.date: 07/01/2019
ms.author: v-yeche
ms.reviewer: jroth
ms.openlocfilehash: cff9203680950c9412f3ca47ca1af73a1b79e1e3
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67570246"
---
# <a name="configure-azure-key-vault-integration-for-sql-server-on-azure-virtual-machines-resource-manager"></a>在 Azure 虚拟机上配置 SQL Server 的 Azure Key Vault 集成 (Resource Manager)

> [!div class="op_single_selector"]
> * [Resource Manager](virtual-machines-windows-ps-sql-keyvault.md)
> * [经典](../sqlclassic/virtual-machines-windows-classic-ps-sql-keyvault.md)

## <a name="overview"></a>概述
SQL Server 加密功能多种多样，包括[透明数据加密 (TDE)](https://msdn.microsoft.com/library/bb934049.aspx)、[列级加密 (CLE)](https://msdn.microsoft.com/library/ms173744.aspx) 和[备份加密](https://msdn.microsoft.com/library/dn449489.aspx)。 这些加密形式要求管理和存储用于加密的加密密钥。 Azure Key Vault (AKV) 服务专用于在一个高度可用的安全位置改进这些密钥的安全性和管理。 [SQL Server 连接器](https://www.microsoft.com/download/details.aspx?id=45344) 使 SQL Server 能够使用 Azure 密钥保管库中的这些密钥。

如果在本地计算机上运行 SQL Server，请[按照此处步骤通过本地 SQL Server 计算机访问 Azure 密钥保管库](https://msdn.microsoft.com/library/dn198405.aspx)。 但对于 Azure VM 中的 SQL Server，可以通过使用 *Azure 密钥保管库集成* 功能节省时间。

启用此功能后，它会自动安装 SQL Server 连接器、配置 EKM 提供程序以访问 Azure 密钥保管库，并创建凭据以使你能够访问保管库。 在前面提到的本地文档列出的步骤中，可以看到此功能自动完成步骤 2 和步骤 3。 仍需手动执行的唯一操作是创建密钥保管库和密钥。 之后，自动进行 SQL VM 的整个设置。 在此功能完成设置后，可以执行 T-SQL 语句，以按照通常的方式加密数据库或备份。

[!INCLUDE [AKV Integration Prepare](../../../../includes/virtual-machines-sql-server-akv-prepare.md)]

  >[!NOTE]
  > EKM 提供程序版本 1.0.4.0 通过 [SQL IaaS 扩展](/virtual-machines/windows/sql/virtual-machines-windows-sql-server-agent-extension)安装在 SQL Server VM 上。 更新 SQL IaaS 扩展将不会更新提供程序版本。 如有需要（例如，迁移到 SQL 托管实例时），请考虑手动更新 EKM 提供程序版本。

## <a name="enabling-and-configuring-akv-integration"></a>启用和配置 AKV 集成
为现有 VM 预配或配置 AKV 集成时可以启用该集成。

### <a name="new-vms"></a>新的 VM
如果使用资源管理器预配新的 SQL Server 虚拟机，Azure 门户提供了启用 Azure 密匙保管库集成的方式。 Azure 密钥保管库功能仅适用于企业版、开发人员版和评估版的 SQL Server。

![SQL Azure 密钥保管库集成](./media/virtual-machines-windows-ps-sql-keyvault/azure-sql-arm-akv.png)

有关预配的详细演练，请参阅[在 Azure 门户中预配 SQL Server 虚拟机](virtual-machines-windows-portal-sql-server-provision.md)。

### <a name="existing-vms"></a>现有 VM
对于现有的 SQL Server 虚拟机，请选择 SQL Server 虚拟机。 然后选择“设置”边栏选项卡的“SQL Server 配置”部分。  

![现有 VM 的 SQL AKV 集成](./media/virtual-machines-windows-ps-sql-keyvault/azure-sql-rm-akv-existing-vms.png)

在“SQL Server 配置”边栏选项卡的“自动 Key Vault 集成”部分中，单击“编辑”按钮。  

![配置现有 VM 的 SQL AKV 集成](./media/virtual-machines-windows-ps-sql-keyvault/azure-sql-rm-akv-configuration.png)

完成后，单击“SQL Server 配置”边栏选项卡底部的“确定”按钮保存更改。  

> [!NOTE]
> 我们在此处创建的凭据名称将在稍后映射到 SQL 登录名。 这允许 SQL 登录名访问密钥保管库。 
>
>

> [!NOTE]
> 还可以使用模板配置 AKV 集成。 有关详细信息，请参阅 [Azure quickstart template for Azure Key Vault integration](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-sql-existing-keyvault-update)（用于 Azure 密钥保管库集成的 Azure 快速入门模板）。
> 
> 

[!INCLUDE [AKV Integration Next Steps](../../../../includes/virtual-machines-sql-server-akv-next-steps.md)]

<!--Update_Description: update meta porperties, wording update -->