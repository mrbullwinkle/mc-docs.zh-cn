---
title: Azure PowerShell 脚本 - Azure Cosmos DB Gremlin API 的列出和获取操作
description: Azure PowerShell 脚本 - Azure Cosmos DB Gremlin API 的列出和获取操作
author: rockboyfor
ms.service: cosmos-db
ms.topic: sample
origin.date: 05/18/2019
ms.date: 07/29/2019
ms.author: v-yeche
ms.openlocfilehash: c9db13ace8dcdb7c1ecd0cbfd40759446828c75a
ms.sourcegitcommit: 021dbf0003a25310a4c8582a998c17729f78ce42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68514408"
---
# <a name="list-and-get-databases-and-graphs-for-azure-cosmos-db---gremlin-api"></a>列出和获取 Azure Cosmos DB 的数据库和图 - Gremlin API

[!INCLUDE [updated-for-az](../../../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="sample-script"></a>示例脚本

```powershell
# List and Get operations for Azure Cosmos Gremlin API
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$databaseName = "database1"
$graphName = "graph1"
$accountResourceName = $accountName + "/gremlin/"
$databaseResourceName = $accountName + "/gremlin/" + $databaseName
$graphResourceName = $accountName + "/gremlin/" + $databaseName + "/" + $graphName

Read-Host -Prompt "List all databases in an account. Press Enter to continue"

Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts/apis/databases" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $accountResourceName  | Select-Object Properties

Read-Host -Prompt "Get a database in an account. Press Enter to continue"

Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts/apis/databases" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $databaseResourceName | Select-Object Properties

Read-Host -Prompt "List all graphs in a database. Press Enter to continue"

Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts/apis/databases/graphs" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $databaseResourceName | Select-Object Properties

Read-Host -Prompt "Get a graph in a database. Press Enter to continue"

Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts/apis/databases/graphs" `
    -ApiVersion "2015-04-08" -ResourceGroupName $resourceGroupName `
    -Name $graphResourceName | Select-Object Properties

```

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

```powershell
Remove-AzResourceGroup -ResourceGroupName "myResourceGroup"
```

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 注释 |
|---|---|
|**Azure 资源**| |
| [New-AzResource](https://docs.microsoft.com/powershell/module/az.resources/new-azresource) | 创建资源。 |
|**Azure 资源组**| |
| [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/powershell/)。

可以在 [Azure Cosmos DB PowerShell 脚本](../../../powershell-samples.md)中找到其他 Azure Cosmos DB PowerShell 脚本示例。

<!--Update_Description: update meta properties -->
