---
title: 使用 Azure 门户导出 Azure 资源管理器模板
description: 使用 Azure 门户从订阅中的资源导出 Azure 资源管理器模板。
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: conceptual
origin.date: 06/19/2019
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: 977421ad67dc6e5dbab97fd60daa806ac9235ab9
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337335"
---
# <a name="single-and-multi-resource-export-to-template-in-azure-portal"></a>在 Azure 门户中将单个资源和多个资源导出到模板

为了帮助创建 Azure 资源管理器模板，可以从现有的资源导出模板。 导出的模板可帮助你了解用于部署资源的 JSON 语法和属性。 若要自动完成将来的部署，可从导出的模板着手，根据具体的方案修改此模板。

在资源管理器中，可以选择一个或多个要导出到模板的资源。 你可以完全专注于模板中所需的资源。

本文介绍如何通过门户导出模板。 也可以使用 [Azure CLI](manage-resource-groups-cli.md#export-resource-groups-to-templates)、[Azure PowerShell](manage-resource-groups-powershell.md#export-resource-groups-to-templates) 或 [REST API](https://docs.microsoft.com/rest/api/resources/resourcegroups/exporttemplate)。

## <a name="choose-the-right-export-option"></a>选择适当的导出选项

可以通过两种方式来导出模板：

* **从资源组或资源导出**。 此选项基于现有的资源生成新模板。 导出的模板是资源组当前状态的“快照”。 可以导出整个资源组，或该资源组中的特定资源。

* **在部署之前导出或从历史记录导出**。 此选项检索用于部署的确切模板副本。

根据所选的选项，导出的模板具有不同的质量。

| 从资源组或资源 | 在部署之前或从历史记录 |
| --------------------- | ----------------- |
| 模板是资源当前状态的快照。 其中包含你在部署之后所做的任何手动更改。 | 模板仅显示资源在部署时的状态。 不包含部署之后所做的任何手动更改。 |
| 可以从资源组中选择要导出的资源。 | 包含特定部署的所有资源。 不能选取其中的一部分资源，或者包含在不同时间添加的资源。 |
| 模板包含资源的所有属性，包括部署过程中通常不会设置的某些属性。 在重复使用模板之前，你可能需要删除或清理这些属性。 | 模板仅包含部署所需的属性。 模板随时可供使用。 |
| 模板可能不包含重复使用它所需的所有参数。 大多数属性值在模板中已硬编码。 若要在其他环境中重新部署模板，需要添加参数，以提高配置资源的能力。 | 模板包含一些参数以方便在不同的环境中重新部署。 |

在以下情况下，请从资源组或资源导出模板：

* 需要捕获在原始部署之后对资源所做的更改。
* 想要选择要导出的资源。

在以下情况下，请在部署之前或者从历史记录导出模板：

* 想要一个易于重复使用的模板。
* 不需要包含原始部署之后所做的更改。

## <a name="export-template-from-resource-group"></a>从资源组导出模板

若要从资源组中导出一个或多个资源：

1. 选择包含所要导出的资源的资源组。

1. 若要导出资源组中的所有资源，请依次选择“全部”、“导出模板”。  只有在至少选择了一个资源之后，才会启用“导出模板”选项。 

    ![导出所有资源](./media/export-template-portal/select-all-resources.png)

1. 若要选取要导出的特定资源，请选中这些资源旁边的复选框。 然后选择“导出模板”。 

    ![选择要导出的资源](./media/export-template-portal/select-resources.png)

1. 随后，导出的模板将会显示，并可供下载。

    ![显示模板](./media/export-template-portal/show-template.png)

## <a name="export-template-from-resource"></a>从资源导出模板

导出一个资源：

1. 选择包含所要导出的资源的资源组。

1. 选择要导出的资源。

    ![选择资源](./media/export-template-portal/select-link-resource.png)

1. 在左窗格中选择该资源对应的“导出模板”。 

    ![导出资源](./media/export-template-portal/export-single-resource.png)

1. 随后，导出的模板将会显示，并可供下载。 模板只包含单个资源。

## <a name="export-template-before-deployment"></a>在部署之前导出模板

1. 选择要部署的 Azure 服务。

1. 填写新服务的值。

1. 在通过验证之后、开始部署之前，请选择“下载自动化模板”。 

    ![下载模板](./media/export-template-portal/download-before-deployment.png)

1. 随后，该模板将会显示，并可供下载。

    ![显示模板](./media/export-template-portal/show-template-before-deployment.png)

## <a name="export-template-after-deployment"></a>在部署之后导出模板

可以导出用于部署现有资源的模板。 获取的模板正是用于部署的模板。

1. 选择要导出的资源组。

1. 选择“部署”下面的链接。 

    ![选择部署历史记录](./media/export-template-portal/select-deployment-history.png)

1. 从部署历史记录中选择一个部署。

    ![选择部署](./media/export-template-portal/select-details.png)

1. 选择“模板”。  随后，用于此部署的模板将会显示，并可供下载。

    ![选择模板](./media/export-template-portal/show-template-from-history.png)

## <a name="next-steps"></a>后续步骤

- 了解如何使用 [Azure CLI](manage-resource-groups-cli.md#export-resource-groups-to-templates)、[Azure PowerShell](manage-resource-groups-powershell.md#export-resource-groups-to-templates) 或 [REST API](https://docs.microsoft.com/rest/api/resources/resourcegroups/exporttemplate) 导出模板。
- 若要了解资源管理器模板语法，请参阅[了解 Azure 资源管理器模板的结构和语法](./resource-group-authoring-templates.md)。
- 若要了解如何开发模板，请参阅[分步教程](/azure-resource-manager/)。

<!--Not Available on [template reference](https://docs.microsoft.com/zh-cn/azure/templates/)-->

<!-- Update_Description: new article about export template portal -->
<!--ms.date: 07/22/2019-->