---
title: 将 Azure 资源移到新的订阅或资源组 | Azure
description: 使用 Azure Resource Manager 将资源移到新的资源组或订阅。
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: conceptual
origin.date: 07/09/2019
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: 030d6fb061a48f18b9de850f650f1bd84aa44490
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337452"
---
# <a name="move-resources-to-a-new-resource-group-or-subscription"></a>将资源移到新的资源组或订阅

本文说明了如何将 Azure 资源移动到另一 Azure 订阅，或移动到同一订阅下的另一资源组。 可以使用 Azure 门户、Azure PowerShell、Azure CLI 或 REST API 移动资源。

在移动操作过程中，源组和目标组都会锁定。 在完成移动之前，将阻止对资源组执行写入和删除操作。 此锁意味着将无法添加、更新或删除资源组中的资源，但并不意味着资源已被冻结。 例如，如果将 SQL Server 及其数据库移到新的资源组中，使用数据库的应用程序体验不到停机， 仍可读取和写入到数据库。

移动资源只会将其移到新的资源组或订阅。 它不会更改资源的位置。

## <a name="checklist-before-moving-resources"></a>移动资源前需查看的清单

移动资源之前需执行的一些重要步骤。 验证这些条件可以避免错误。

1. 要移动的资源必须支持移动操作。 有关支持移动的资源列表，请参阅[资源的移动操作支持](move-support-resources.md)。

1. 某些服务在移动资源时有特定的限制或要求。 如果要移动以下任何服务，请在移动之前查看该指南。

    * [应用程序服务移动指南](./move-limitations/app-service-move-limitations.md)
        <!--Not Available on * [Azure DevOps Services move guidance](https://docs.microsoft.com/zh-cn/azure/devops/organizations/billing/change-azure-subscription?toc=/azure/azure-resource-manager/toc.json)-->
    * [经典部署模型移动指南](./move-limitations/classic-model-move-limitations.md) - 经典计算、经典存储、经典虚拟网络和云服务  <!--Not Available on * [Recovery Services move guidance](../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json)-->
    * [虚拟机移动指南](./move-limitations/virtual-machines-move-limitations.md)
    * [虚拟网络移动指南](./move-limitations/virtual-network-move-limitations.md)

1. 源订阅和目标订阅必须处于活动状态。 如果在启用已禁用的帐户时遇到问题，请[创建 Azure 支持请求](https://support.azure.cn/zh-cn/support/support-azure/?l=zh-cn)。 选择“订阅管理”  作为问题类型。

1. 源订阅与目标订阅必须在同一个 [Azure Active Directory 租户](../active-directory/develop/quickstart-create-new-tenant.md)中。 若要检查这两个订阅是否具有相同的租户 ID，请使用 Azure PowerShell 或 Azure CLI。

    对于 Azure PowerShell，请使用：

    ```powershell
    (Get-AzSubscription -SubscriptionName <your-source-subscription>).TenantId
    (Get-AzSubscription -SubscriptionName <your-destination-subscription>).TenantId
    ```

    对于 Azure CLI，请使用：

    ```azurecli
    az account show --subscription <your-source-subscription> --query tenantId
    az account show --subscription <your-destination-subscription> --query tenantId
    ```

    如果源订阅和目标订阅的租户 ID 不相同，可使用以下方法协调租户 ID：

    * [将 Azure 订阅所有权转让给其他帐户](/billing/billing-subscription-transfer/)
        <!--MOONCAKE: URL CORRECT ON /billing/billing-subscription-transfer/ -->
    * 将 Azure 订阅关联或添加到 Azure Active Directory   <!--MOONCAKE: NOT Available on (../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)-->
1. 必须针对要移动的资源的资源提供程序注册目标订阅。 否则，会收到错误，指明 **未针对资源类型注册订阅**。 将资源移到新的订阅时，可能会看到此错误，但该订阅从未配合该资源类型使用。

    对于 PowerShell，请使用以下命令来获取注册状态：

    ```powershell
    Set-AzContext -Subscription <destination-subscription-name-or-id>
    Get-AzResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
    ```

    若要注册资源提供程序，请使用：

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
    ```

    对于 Azure CLI，请使用以下命令来获取注册状态：

    ```azurecli
    az account set -s <destination-subscription-name-or-id>
    az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
    ```

    若要注册资源提供程序，请使用：

    ```azurecli
    az provider register --namespace Microsoft.Batch
    ```

1. 移动资源的帐户至少需要具备下列权限：

    * 源资源组上的 Microsoft.Resources/subscriptions/resourceGroups/moveResources/action  权限。
    * 目标资源组上的 Microsoft.Resources/subscriptions/resourceGroups/write  权限。

1. 在移动资源之前，请检查要将资源移动到的订阅的订阅配额。 如果移动资源意味着订阅将超出其限制，则需要检查是否可以请求增加配额。 有关限制的列表及如何请求增加配额的信息，请参阅 [Azure 订阅和服务限制、配额与约束](../azure-subscription-service-limits.md)。

## <a name="validate-move"></a>验证移动

[验证移动操作](https://docs.microsoft.com/rest/api/resources/resources/validatemoveresources)可以测试你的移动方案而无需实际移动资源。 使用此操作检查移动是否会成功。 发送移动请求时会自动调用验证。 仅当需要预先确定结果时才使用此操作。 若要运行此操作，需要：

* 源资源组的名称
* 目标资源组的资源 ID
* 要移动的每个资源的资源 ID
* 你的帐户的[访问令牌](https://docs.microsoft.com/rest/api/azure/#acquire-an-access-token)

发送以下请求：

```HTTP
POST https://management.chinacloudapi.cn/subscriptions/<subscription-id>/resourceGroups/<source-group>/validateMoveResources?api-version=2019-05-10
Authorization: Bearer <access-token>
Content-type: application/json
```

包含请求正文：

```json
{
 "resources": ["<resource-id-1>", "<resource-id-2>"],
 "targetResourceGroup": "/subscriptions/<subscription-id>/resourceGroups/<target-group>"
}
```

如果请求格式正确，则操作将返回：

```HTTP
Response Code: 202
cache-control: no-cache
pragma: no-cache
expires: -1
location: https://management.chinacloudapi.cn/subscriptions/<subscription-id>/operationresults/<operation-id>?api-version=2018-02-01
retry-after: 15
...
```

202 状态代码指示已接受验证请求，但尚未确定移动操作是否会成功。 `location` 值包含用于检查长时间运行操作的状态的 URL。  

若要检查状态，请发送以下请求：

```HTTP
GET <location-url>
Authorization: Bearer <access-token>
```

操作仍在运行时，会继续收到 202 状态代码。 请等待 `retry-after` 值中所示的秒数，然后重试。 如果移动操作验证成功，则会收到 204 状态代码。 如果移动验证失败，则会收到错误消息，例如：

```json
{"error":{"code":"ResourceMoveProviderValidationFailed","message":"<message>"...}}
```

<a name="use-portal"></a>
## <a name="use-the-portal"></a>使用门户

若要移动资源，请选择包含这些资源的资源组，然后选择“移动”  按钮。

![移动资源](./media/resource-group-move-resources/select-move.png)

选择是要将资源移到新资源组还是新订阅。

选择要移动的资源和目标资源组。 确认需要更新这些资源的脚本，选择“确定”  。 如果在上一步中已选择“编辑订阅”图标，则还必须选择目标订阅。

![选择目标](./media/resource-group-move-resources/select-destination.png)

在“通知”  中，可以看到移动操作正在运行。

![显示移动状态](./media/resource-group-move-resources/show-status.png)

操作完成后，你会获得结果通知。

![显示移动结果](./media/resource-group-move-resources/show-result.png)

如果出现错误，请参阅[排查将 Azure 资源移到新的资源组或订阅时遇到的问题](troubleshoot-move.md)。

<a name="use-powershell"></a>
## <a name="use-azure-powershell"></a>使用 Azure PowerShell

要将现有资源移到另一个资源组或订阅，请使用 [Move-AzResource](https://docs.microsoft.com/powershell/module/az.resources/move-azresource) 命令。 下面的示例演示了如何将多个资源移动到新的资源组。

```powershell
$webapp = Get-AzResource -ResourceGroupName OldRG -ResourceName ExampleSite
$plan = Get-AzResource -ResourceGroupName OldRG -ResourceName ExamplePlan
Move-AzResource -DestinationResourceGroupName NewRG -ResourceId $webapp.ResourceId, $plan.ResourceId
```

若要移到新订阅，请包含 `DestinationSubscriptionId` 参数的值。

如果出现错误，请参阅[排查将 Azure 资源移到新的资源组或订阅时遇到的问题](troubleshoot-move.md)。

<a name="use-azure-cli"></a>
## <a name="use-azure-cli"></a>使用 Azure CLI

若要将现有资源移动到另一个资源组或订阅，请使用 [az resource move](https://docs.azure.cn/zh-cn/cli/resource?view=azure-cli-latest#az-resource-move) 命令。 提供要移动的资源的资源 ID。 下面的示例演示了如何将多个资源移动到新的资源组。 在 `--ids` 参数中，提供要移动的资源 ID 的空格分隔列表。

```azurecli
webapp=$(az resource show -g OldRG -n ExampleSite --resource-type "Microsoft.Web/sites" --query id --output tsv)
plan=$(az resource show -g OldRG -n ExamplePlan --resource-type "Microsoft.Web/serverfarms" --query id --output tsv)
az resource move --destination-group newgroup --ids $webapp $plan
```

若要移到新订阅，请提供 `--destination-subscription-id` 参数。

如果出现错误，请参阅[排查将 Azure 资源移到新的资源组或订阅时遇到的问题](troubleshoot-move.md)。

## <a name="use-rest-api"></a>使用 REST API

要将现有资源移到另一个资源组或订阅，请使用[移动资源](https://docs.microsoft.com/rest/api/resources/Resources/MoveResources)操作。

```HTTP
POST https://management.chinacloudapi.cn/subscriptions/{source-subscription-id}/resourcegroups/{source-resource-group-name}/moveResources?api-version={api-version}
```

在请求正文中，指定目标资源组和要移动的资源。

```json
{
 "resources": ["<resource-id-1>", "<resource-id-2>"],
 "targetResourceGroup": "/subscriptions/<subscription-id>/resourceGroups/<target-group>"
}
```

如果出现错误，请参阅[排查将 Azure 资源移到新的资源组或订阅时遇到的问题](troubleshoot-move.md)。

## <a name="next-steps"></a>后续步骤

有关支持移动的资源列表，请参阅[资源的移动操作支持](move-support-resources.md)。

<!--Update_Description: update meta properties, wording update, update link -->
