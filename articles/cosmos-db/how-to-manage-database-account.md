---
title: 了解如何在 Azure Cosmos DB 中管理数据库帐户
description: 了解如何在 Azure Cosmos DB 中管理数据库帐户
author: rockboyfor
ms.service: cosmos-db
ms.topic: sample
origin.date: 05/23/2019
ms.date: 07/29/2019
ms.author: v-yeche
ms.openlocfilehash: 928e8960fcbe1758a5eb96d83980e615cea220a6
ms.sourcegitcommit: 021dbf0003a25310a4c8582a998c17729f78ce42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68514172"
---
<!-- Verify Successfully-->
# <a name="manage-an-azure-cosmos-account"></a>管理 Azure Cosmos 帐户

本文介绍如何使用 Azure 门户、Azure PowerShell、Azure CLI 和 Azure 资源管理器模板管理 Azure Cosmos 帐户中的各种任务。

## <a name="create-an-account"></a>创建帐户

<a name="create-database-account-via-portal"></a>
### <a name="azure-portal"></a>Azure 门户

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

<a name="create-database-account-via-cli"></a>
### <a name="azure-cli"></a>Azure CLI

```azurecli
# Sign in the Azure China Cloud
az cloud set -n AzureChinaCloud
az login

# Create an account
$resourceGroupName = 'myResourceGroup'
$accountName = 'myaccountname' # must be lower case.

az cosmosdb create \
   --name $accountName \
   --resource-group $resourceGroupName \
   --kind GlobalDocumentDB \
   --default-consistency-level Session \
   --locations chinanorth=0 chinaeast=1 \
   --enable-multiple-write-locations true
```

<a name="create-database-account-via-ps"></a>
### <a name="azure-powershell"></a>Azure PowerShell
```powershell
# Sign in the Azure China Cloud
Connect-AzAccount -Environment AzureChinaCloud

# Create an Azure Cosmos account for Core (SQL) API
$resourceGroupName = "myResourceGroup"
$location = "China North"
$accountName = "mycosmosaccount" # must be lower case.

$locations = @(
    @{ "locationName"="China North"; "failoverPriority"=0 },
    @{ "locationName"="China East"; "failoverPriority"=1 }
)

$consistencyPolicy = @{
    "defaultConsistencyLevel"="BoundedStaleness";
    "maxIntervalInSeconds"=300;
    "maxStalenessPrefix"=100000
}

$CosmosDBProperties = @{
    "databaseAccountOfferType"="Standard";
    "locations"=$locations;
    "consistencyPolicy"=$consistencyPolicy;
    "enableMultipleWriteLocations"="true"
}

New-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName -Location $location `
    -Name $accountName -PropertyObject $CosmosDBProperties
```

<a name="create-database-account-via-arm-template"></a>
### <a name="azure-resource-manager-template"></a>Azure Resource Manager 模板

此 Azure 资源管理器模板将为任何受支持的 API（配置有两个区域以及用于选择一致性级别、自动故障转移和多主数据库的选项）创建 Azure Cosmos 帐户。 若要部署此模板，请在自述文件页[创建 Azure Cosmos 帐户](https://github.com/Azure/azure-quickstart-templates/tree/master/101-cosmosdb-create-multi-region-account)上，单击“部署到 Azure”

<!--MOONCAKE: CUSTOMIZE-->

[![“部署到 Azure”](http://azuredeploy.net/deploybutton.png)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-create-multi-region-account%2Fazuredeploy.json)

<!--MOONCAKE: CUSTOMIZE-->

## <a name="addremove-regions-from-your-database-account"></a>在数据库帐户中添加/删除区域

<a name="add-remove-regions-via-portal"></a>
### <a name="azure-portal"></a>Azure 门户

1. 登录到 [Azure 门户](https://portal.azure.cn)。 

1. 导航到 Azure Cosmos 帐户，打开“全局复制数据”菜单  。

    <!--MOONCAKE: submene correct on **Replicate data globally**-->
1. 要添加区域，请在地图上选择包含与所需区域对应的 +  标签的六边形。 另外，若要添加某个区域，请选择“+ 添加区域”选项，然后从下拉菜单中选择一个区域。 

1. 若要删除区域，请选择带对号的蓝色六边形以从地图中清除一个或多个区域。 或者选择右侧位于区域旁边的“废纸篓”(🗑) 图标。

1. 若要保存更改，请选择“确定”。 

    ![添加或删除区域菜单](./media/how-to-manage-database-account/add-region.png)

在单区域写入模式下，不能删除写入区域。 必须先故障转移到另一区域，然后才能删除当前的写入区域。

在多区域写入模式下，如果你至少具有一个区域，则可以添加或删除任何区域。

<a name="add-remove-regions-via-cli"></a>
### <a name="azure-cli"></a>Azure CLI

```azurecli
$resourceGroupName = 'myResourceGroup'
$accountName = 'myaccountname'

# Create an account with 1 region
az cosmosdb create --name $accountName --resource-group $resourceGroupName --locations chinanorth=0

# Add a region
az cosmosdb update --name $accountName --resource-group $resourceGroupName --locations chinanorth=0 chinaeast=1

# Remove a region
az cosmosdb update --name $accountName --resource-group $resourceGroupName --locations chinanorth=0
```

<a name="add-remove-regions-via-ps"></a>
### <a name="azure-powershell"></a>Azure PowerShell

```powershell
# Create an account with 1 region
$resourceGroupName = "myResourceGroup"
$location = "China North"
$accountName = "mycosmosaccount" # must be lower case.

$locations = @( @{ "locationName"="China North"; "failoverPriority"=0 } )
$consistencyPolicy = @{ "defaultConsistencyLevel"="Session" }
$CosmosDBProperties = @{
    "databaseAccountOfferType"="Standard";
    "locations"=$locations;
    "consistencyPolicy"=$consistencyPolicy
}
New-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName -Location $location `
    -Name $accountName -PropertyObject $CosmosDBProperties

# Add a region
$account = Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName -Name $accountName

$locations = @( 
    @{ "locationName"="China North"; "failoverPriority"=0 },
    @{ "locationName"="China East"; "failoverPriority"=1 } 
)

$account.Properties.locations = $locations
$CosmosDBProperties = $account.Properties

Set-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $accountName -PropertyObject $CosmosDBProperties

# Azure Resource Manager does not wait on the resource update
Write-Host "Confirm region added before continuing..."

# Remove a region
$account = Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName -Name $accountName

$locations = @( @{ "locationName"="China North"; "failoverPriority"=0 } )

$account.Properties.locations = $locations
$CosmosDBProperties = $account.Properties

Set-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $accountName -PropertyObject $CosmosDBProperties
```

<a name="configure-multiple-write-regions"></a>
## <a name="configure-multiple-write-regions"></a>配置多个写入区域

<a name="configure-multiple-write-regions-portal"></a>
### <a name="azure-portal"></a>Azure 门户

打开“全局复制数据”选项卡，选择“启用”以启用多区域写入   。 启用多区域写入后，你的帐户当前拥有的所有读取区域将变为读取和写入区域。 

> [!NOTE]
> 启用多区域写入后，无法禁用它。 

![Azure Cosmos 帐户配置多主数据库屏幕快照](./media/how-to-manage-database-account/single-to-multi-master.png)

<!--Not Available on Please reach out to the askcosmosdb@microsoft.com alias for additional questions about this feature.-->

<a name="configure-multiple-write-regions-cli"></a>
### <a name="azure-cli"></a>Azure CLI

```azurecli
$resourceGroupName = 'myResourceGroup'
$accountName = 'myaccountname'
az cosmosdb update --name $accountName --resource-group $resourceGroupName --enable-multiple-write-locations true
```

<a name="configure-multiple-write-regions-ps"></a>
### <a name="azure-powershell"></a>Azure PowerShell

```powershell
# Update an Azure Cosmos account from single to multi-master

$account = Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName -Name $accountName

$account.Properties.enableMultipleWriteLocations = "true"
$CosmosDBProperties = $account.Properties

Set-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $accountName -PropertyObject $CosmosDBProperties
```

<a name="configure-multiple-write-regions-arm"></a>
### <a name="resource-manager-template"></a>Resource Manager 模板

可通过部署用于创建帐户的资源管理器模板和设置 `enableMultipleWriteLocations: true` 来将一个帐户从单主数据库迁移到多主数据库。 以下 Azure 资源管理器模板是一个极简模板，该模板会在启用单区域和多主数据库的情况下为 SQL API 部署 Azure Cosmos 帐户。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "String"
        },
        "location": {
            "type": "String",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "type": "Microsoft.DocumentDb/databaseAccounts",
            "kind": "GlobalDocumentDB",
            "name": "[parameters('name')]",
            "apiVersion": "2015-04-08",
            "location": "[parameters('location')]",
            "tags": {},
            "properties": {
                "databaseAccountOfferType": "Standard",
                "consistencyPolicy": { "defaultConsistencyLevel": "Session" },
                "locations": [
                    {
                        "locationName": "[parameters('location')]",
                        "failoverPriority": 0
                    }
                ],
                "enableMultipleWriteLocations": true
            }
        }
    ]
}
```

<a name="automatic-failover"></a>
## <a name="enable-automatic-failover-for-your-azure-cosmos-account"></a>为 Azure Cosmos 帐户启用自动故障转移

借助自动故障转移选项，在某个区域不可用时，Azure Cosmos DB 可以故障转移到具有最高故障转移优先级的区域，无需用户操作。 如果启用自动故障转移，则可修改区域优先级。 帐户必须具有两个或更多区域以启用自动故障转移。

<a name="enable-automatic-failover-via-portal"></a>
### <a name="azure-portal"></a>Azure 门户

1. 在 Azure Cosmos 帐户中，打开“全局复制数据”窗格  。
    
    <!--MOONCAKE: CORRECT ON Replicate data globally-->
    
2. 在窗格顶部选择“自动故障转移”。 

    ![“多区域复制数据”菜单](./media/how-to-manage-database-account/replicate-data-globally.png)

3. 在“自动故障转移”窗格中，确保将“启用自动故障转移”设置为“开”。    

4. 选择“保存”。 

    ![自动故障转移门户菜单](./media/how-to-manage-database-account/automatic-failover.png)

<a name="enable-automatic-failover-via-cli"></a>
### <a name="azure-cli"></a>Azure CLI

```azurecli
# Enable automatic failover on an existing account
$resourceGroupName = 'myResourceGroup'
$accountName = 'myaccountname'

az cosmosdb update --name $accountName --resource-group $resourceGroupName --enable-automatic-failover true
```

<a name="enable-automatic-failover-via-ps"></a>
### <a name="azure-powershell"></a>Azure PowerShell

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"

$account = Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $accountName

$account.Properties.enableAutomaticFailover="true";
$CosmosDBProperties = $account.Properties;

Set-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $accountName -PropertyObject $CosmosDBProperties
```

## <a name="set-failover-priorities-for-your-azure-cosmos-account"></a>为 Azure Cosmos 帐户设置故障转移优先级

Cosmos 帐户配置为自动故障转移后，可以更改区域的故障转移优先级。

> [!IMPORTANT]
> 帐户配置为自动故障转移后，不能修改写入区域（故障转移优先级为零）。 要更改写入区域，必须禁用自动故障转移并执行手动故障转移。

<a name="set-failover-priorities-via-portal"></a>
### <a name="azure-portal"></a>Azure 门户

1. 在 Azure Cosmos 帐户中，打开“多区域复制数据”窗格  。

2. 在窗格顶部选择“自动故障转移”。 

    ![“多区域复制数据”菜单](./media/how-to-manage-database-account/replicate-data-globally.png)

3. 在“自动故障转移”窗格中，确保将“启用自动故障转移”设置为“开”。   

4. 若要修改故障转移优先级，请将鼠标指针悬停在读取区域上，并通过在行左侧出现的三个点拖动读取区域。

5. 选择“保存”。 

    ![自动故障转移门户菜单](./media/how-to-manage-database-account/automatic-failover.png)

<a name="set-failover-priorities-via-cli"></a>
### <a name="azure-cli"></a>Azure CLI

```azurecli
# Assume region order is initially chinaeast=0 chinanorth=1 chinaeast2=2 on account creation
$resourceGroupName = 'myResourceGroup'
$accountName = 'myaccountname'

az cosmosdb failover-priority-change --name $accountName --resource-group $resourceGroupName --failover-policies chinaeast=0 chinaeast2=1 chinanorth=2
```

<a name="set-failover-priorities-via-ps"></a>
### <a name="azure-powershell"></a>Azure PowerShell

<!--MOONCAKE: China East and China East 2 change each other.-->
```powershell
# Assume account currently has regions with priority: China North = 0, China East = 1, China East 2 = 2
$resourceGroupName = "myResourceGroup"
$accountName = "myaccountname"

$failoverPolicies = @(
    @{ "locationName"="China North"; "failoverPriority"=0 },
    @{ "locationName"="China East 2"; "failoverPriority"=1 },
    @{ "locationName"="China East"; "failoverPriority"=2 }
)

Invoke-AzResourceAction -Action failoverPriorityChange `
    -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" `
    -ResourceGroupName $resourceGroupName -Name $accountName -Parameters $failoverPolicies
```

<a name="manual-failover"></a>
## <a name="perform-manual-failover-on-an-azure-cosmos-account"></a>在 Azure Cosmos 帐户上执行手动故障转移

> [!IMPORTANT]
> Azure Cosmos 帐户必须配置为手动故障转移，才能成功执行此操作。

执行手动故障转移的过程涉及将帐户的写入区域（故障转移优先级 = 0）更改为已为该帐户配置的其他区域。

> [!NOTE]
> 多主数据库帐户不能进行手动故障转移。 对于使用 Azure Cosmos SDK 的应用程序，SDK 会检测某个区域何时变为不可用，然后自动重定向到下一个最近的区域（如果在 SDK 中使用多宿主 API）。

<a name="enable-manual-failover-via-portal"></a>
### <a name="azure-portal"></a>Azure 门户

1. 导航到 Azure Cosmos 帐户，打开“全局复制数据”菜单  。
    
    <!--MOONCAKE: submene correct on **Replicate data globally**-->
    
2. 在菜单顶部，选择“手动故障转移”。 

    ![“多区域复制数据”菜单](./media/how-to-manage-database-account/replicate-data-globally.png)

3. 在“手动故障转移”  菜单上，选择你的新写入区域。 选中相应的复选框，以指示你了解此选项会更改你的写入区域。

4. 若要触发故障转移，请选择“确定”。 

    ![手动故障转移门户菜单](./media/how-to-manage-database-account/manual-failover.png)

<a name="enable-manual-failover-via-cli"></a>
### <a name="azure-cli"></a>Azure CLI

```azurecli
# Assume account currently has regions with priority: chinaeast=0 chinanorth=1
# Change the priority order to trigger a failover of the write region
$resourceGroupName = 'myResourceGroup'
$accountName = 'myaccountname'

az cosmosdb update --name $accountName --resource-group $resourceGroupName --locations chinanorth=0 chinaeast=1
```

<a name="enable-manual-failover-via-ps"></a>
### <a name="azure-powershell"></a>Azure PowerShell

```powershell
# Assume account currently has regions with priority: China North = 0, China East = 1
# Change the priority order to trigger a failover of the write region
$resourceGroupName = "myResourceGroup"
$accountName = "myaccountname"

$account = Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $accountName

$locations = @(
    @{ "locationName"="China East"; "failoverPriority"=0 },
    @{ "locationName"="China North"; "failoverPriority"=1 }
)

$account.Properties.locations=$locations;
$CosmosDBProperties = $account.Properties;

Set-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $accountName -PropertyObject $CosmosDBProperties
```

## <a name="next-steps"></a>后续步骤

有关如何管理 Azure Cosmos 帐户以及数据库和容器的详细信息和示例，请阅读以下文章：

* [使用 Azure PowerShell 管理 Azure Cosmos DB](manage-with-powershell.md)
* [使用 Azure CLI 管理 Azure Cosmos DB](manage-with-cli.md)

<!-- Update_Description: update meta properties, wording update-->