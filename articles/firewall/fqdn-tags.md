---
title: Azure 防火墙的 FQDN 标记概述
description: 了解 Azure 防火墙中的 FQDN 标记
services: firewall
author: rockboyfor
ms.service: firewall
ms.topic: article
origin.date: 04/23/2019
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: 22bbec59f7a5fac902a724ae185241b88376c1c8
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337605"
---
# <a name="fqdn-tags-overview"></a>FQDN 标记概述

FQDN 标记表示与已知的 Azure 服务关联的一组完全限定的域名 (FQDN)。 可以在应用程序规则中使用 FQDN 标记，以允许所需出站网络流量通过防火墙。

例如，若要手动允许 Windows 更新网络流量通过防火墙，需要根据 Azure 文档创建多个应用程序规则。 使用 FQDN 标记，可以创建一个应用程序规则，其中包括 **Windows 更新**标记，现在到 Microsoft Windows 更新终结点的网络流量可以流经防火墙。

你无法创建自己的 FQDN 标记，也无法指定标记中包含哪些 FQDN。 Azure 管理 FQDN 标记包含的 FQDN，并在 FQDN 更改时更新标记。 

<!--- screenshot of application rule with a FQDN tag.-->

下表显示了当前可使用的 FQDN 标记。 Azure 维护这些标记，你可以期望定期添加其他标记。

## <a name="current-fqdn-tags"></a>当前 FQDN 标记

|FQDN 标记  |说明  |
|---------|---------|
|Windows 更新     |允许出站访问 Microsoft 更新，如[如何为软件更新配置防火墙](https://technet.microsoft.com/library/bb693717.aspx)中所述。|
|Windows 诊断|允许出站访问所有 [Windows 诊断终结点](https://docs.microsoft.com/zh-cn/windows/privacy/configure-windows-diagnostic-data-in-your-organization#endpoints)。|
|Microsoft 主动保护服务 (MAPS)|允许出站访问 [MAPS](https://cloudblogs.microsoft.com/enterprisemobility/2016/05/31/important-changes-to-microsoft-active-protection-service-maps-endpoint/)。|
|应用服务环境 (ASE)|允许出站访问 ASE 平台流量。 此标记未涵盖特定于客户的存储和由 ASE 创建的 SQL 终结点。 这些应通过[服务终结点](../virtual-network/tutorial-restrict-network-access-to-resources.md)启用或手动添加。<br /><br />|
|Azure 备份|允许对 Azure 备份服务进行出站访问。|
|Azure HDInsight<br />（预览版）|允许出站访问 HDInsight 平台流量。 此标记未涵盖特定于客户的存储和来自 HDInsight 的 SQL 流量。 使用[服务终结点](../virtual-network/tutorial-restrict-network-access-to-resources.md)启用这些项或手动添加它们。|

<!--Not Available on Line 32 App Service Environment (ASE) [Locking down an App Service Environment](../app-service/environment/firewall-integration.md#configuring-azure-firewall-with-your-ase)-->

> [!NOTE]
> 在应用程序规则中选择 FQDN 标记时，“协议:端口”字段必须设置为 **https**。

## <a name="next-steps"></a>后续步骤

若要了解如何部署 Azure 防火墙，请参阅[教程：使用 Azure 门户部署和配置 Azure 防火墙](tutorial-firewall-deploy-portal.md)。

<!-- Update_Description: new articles on fqdn tags -->
<!--ms.date: 07/22/2019-->