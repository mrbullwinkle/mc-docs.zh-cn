---
title: 使用 REST API 和模板部署资源 | Azure
description: 使用 Azure Resource Manager 和 Resource Manager REST API 将资源部署到 Azure。 资源在 Resource Manager 模板中定义。
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: conceptual
origin.date: 06/04/2019
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: 9cbb6c00f465abba2454be61ee65ed06ad7caefd
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337435"
---
# <a name="deploy-resources-with-resource-manager-templates-and-resource-manager-rest-api"></a>使用 Resource Manager 模板和 Resource Manager REST API 部署资源

本文介绍如何将 Resource Manager REST API 与 Resource Manager 模板配合使用向 Azure 部署资源。  

可以在请求正文中包含模板或链接到文件。 使用文件时，它可以是本地文件，也可以是通过 URI 提供的外部文件。 如果模板位于存储帐户中，可以限制对该模板的访问，并在部署过程中提供共享访问签名 (SAS) 令牌。

## <a name="deployment-scope"></a>部署范围

可将部署目标设定为管理组、Azure 订阅或资源组。 大多数情况下，我们会将部署目标设定为资源组。 使用管理组或订阅部署在指定范围内应用策略和角色分配。 还可以使用订阅部署创建资源组并向其部署资源。 你将根据部署范围使用不同的命令。

若要部署到**资源组**，请使用“[部署 - 创建](https://docs.microsoft.com/rest/api/resources/deployments/createorupdate)”。 请求将发送到：

```HTTP
PUT https://management.chinacloudapi.cn/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2019-05-01
```

若要部署到**订阅**，请使用“[部署 - 在订阅范围创建](https://docs.microsoft.com/rest/api/resources/deployments/createorupdateatsubscriptionscope)”。 请求将发送到：

```HTTP
PUT https://management.chinacloudapi.cn/subscriptions/{subscriptionId}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2019-05-01
```

若要部署到**管理组**，请使用[部署 - 在管理组范围内创建](https://docs.microsoft.com/rest/api/resources/deployments/createorupdateatmanagementgroupscope)。 请求将发送到：

```HTTP
PUT https://management.chinacloudapi.cn/providers/Microsoft.Management/managementGroups/{groupId}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2019-05-01
```

本文中的示例使用资源组部署。 有关订阅部署的详细信息，请参阅[在订阅级别创建资源组和资源](deploy-to-subscription.md)。

## <a name="deploy-with-the-rest-api"></a>使用 REST API 进行部署

1. 设置 [常见参数和标头](https://docs.microsoft.com/rest/api/azure/)，包括身份验证令牌。

1. 如果目前没有资源组，请创建资源组。 提供订阅 ID、新资源组的名称，以及解决方案所需的位置。 有关详细信息，请参阅 [创建资源组](https://docs.microsoft.com/rest/api/resources/resourcegroups/createorupdate)。

    ```HTTP
    PUT https://management.chinacloudapi.cn/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>?api-version=2019-05-01
    ```

    使用如下所示请求正文：

    ```json
    {
    "location": "China North",
    "tags": {
      "tagname1": "tagvalue1"
    }
    }
    ```

1. 在执行部署之前，通过运行[验证模板部署](https://docs.microsoft.com/rest/api/resources/deployments/validate)操作来验证部署。 测试部署时，请提供与执行部署时所提供的完全相同的参数（如下一步中所示）。

1. 若要部署模板，请在请求 URI 中提供订阅 ID、资源组名称和部署名称。 

    ```HTTP
    PUT https://management.chinacloudapi.cn/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2019-05-01
    ```

    在请求正文中，提供指向模板和参数文件的链接。 请注意，**mode** 设置为 **Incremental**。 要运行完整部署，请将 **mode** 设置为 **Complete**。 使用完整模式时要小心，因为可能会无意中删除不在模板中的资源。

    ```json
    {
    "properties": {
      "templateLink": {
        "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/template.json",
        "contentVersion": "1.0.0.0"
      },
      "parametersLink": {
        "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/parameters.json",
        "contentVersion": "1.0.0.0"
      },
      "mode": "Incremental"
    }
    }
    ```

    如果想要记录响应内容或/和请求内容，请在请求中包括 **debugSetting**。

    ```json
    {
    "properties": {
      "templateLink": {
        "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/template.json",
        "contentVersion": "1.0.0.0"
      },
      "parametersLink": {
        "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/parameters.json",
        "contentVersion": "1.0.0.0"
      },
      "mode": "Incremental",
      "debugSetting": {
        "detailLevel": "requestContent, responseContent"
      }
    }
    }
    ```

    可以将存储帐户设置为使用共享访问签名 (SAS) 令牌。 有关详细信息，请参阅[使用共享访问签名委托访问权限](https://docs.microsoft.com/rest/api/storageservices/delegating-access-with-a-shared-access-signature)。

1. 可以将模板和参数包含在请求正文中，而不是链接到模板和参数的文件。 以下示例显示了内联模板和参数的请求正文：

    ```json
    {
      "properties": {
      "mode": "Incremental",
      "template": {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "storageAccountType": {
            "type": "string",
            "defaultValue": "Standard_LRS",
            "allowedValues": [
              "Standard_LRS",
              "Standard_GRS",
              "Standard_ZRS",
              "Premium_LRS"
            ],
            "metadata": {
              "description": "Storage Account type"
            }
          },
          "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
              "description": "Location for all resources."
            }
          }
        },
        "variables": {
          "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
        },
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[variables('storageAccountName')]",
            "apiVersion": "2018-02-01",
            "location": "[parameters('location')]",
            "sku": {
              "name": "[parameters('storageAccountType')]"
            },
            "kind": "StorageV2",
            "properties": {}
          }
        ],
        "outputs": {
          "storageAccountName": {
            "type": "string",
            "value": "[variables('storageAccountName')]"
          }
        }
      },
      "parameters": {
        "location": {
          "value": "chinaeast2"
        }
      }
    }
    }
    ```

1. 若要获取模板部署的状态，请使用[部署 - 获取](https://docs.microsoft.com/rest/api/resources/deployments/get)。

    ```HTTP
    GET https://management.chinacloudapi.cn/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2018-05-01
    ```

## <a name="redeploy-when-deployment-fails"></a>部署失败时，重新部署

此功能也称为“出错时回滚”  。 部署失败时，可以自动重新部署部署历史记录中先前成功的部署。 要指定重新部署，请在请求正文中使用 `onErrorDeployment` 属性。 如果基础结构部署存在一个已知良好的状态，并且你希望还原到此状态，则此功能非常有用。 有许多需要注意的问题和限制：

- 重新部署使用与以前运行它时相同的参数以相同的方式运行。 无法更改参数。
- 以前的部署是使用[完整模式](./deployment-modes.md#complete-mode)运行的。 以前的部署中未包括的任何资源都将被删除，任何资源配置都将设置为以前的状态。 请确保你完全理解[部署模式](./deployment-modes.md)。
- 重新部署只会影响资源，而不会影响任何数据更改。
- 只有资源组部署支持此功能，订阅级部署不支持此功能。 有关订阅级部署的详细信息，请参阅[在订阅级别创建资源组和资源](./deploy-to-subscription.md)。

若要使用此选项，部署必须具有唯一的名称，以便可以在历史记录中标识它们。 如果没有唯一名称，则当前失败的部署可能会覆盖历史记录中以前成功的部署。 只能将此选项用于根级别部署。 从嵌套模板进行的部署不可用于重新部署。

若要在当前部署失败的情况下，重新部署上一个成功部署，请使用：

```json
{
  "properties": {
    "templateLink": {
      "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/template.json",
      "contentVersion": "1.0.0.0"
    },
    "mode": "Incremental",
    "parametersLink": {
      "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/parameters.json",
      "contentVersion": "1.0.0.0"
    },
    "onErrorDeployment": {
      "type": "LastSuccessful",
    }
  }
}
```

若要在当前部署失败的情况下，重新部署特定部署，请使用：

```json
{
  "properties": {
    "templateLink": {
      "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/template.json",
      "contentVersion": "1.0.0.0"
    },
    "mode": "Incremental",
    "parametersLink": {
      "uri": "http://mystorageaccount.blob.core.chinacloudapi.cn/templates/parameters.json",
      "contentVersion": "1.0.0.0"
    },
    "onErrorDeployment": {
      "type": "SpecificDeployment",
      "deploymentName": "<deploymentname>"
    }
  }
}
```

指定的部署必须已成功。

## <a name="parameter-file"></a>参数文件

如果要在部署期间使用参数文件传递参数值，需要使用类似于以下示例的格式创建一个 JSON 文件：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "webSiteName": {
            "value": "ExampleSite"
        },
        "webSiteHostingPlanName": {
            "value": "DefaultPlan"
        },
        "webSiteLocation": {
            "value": "China North"
        },
        "adminPassword": {
            "reference": {
               "keyVault": {
                  "id": "/subscriptions/{guid}/resourceGroups/{group-name}/providers/Microsoft.KeyVault/vaults/{vault-name}"
               },
               "secretName": "sqlAdminPassword"
            }
        }
   }
}
```

参数文件的大小不能超过 64 KB。

如果需要为参数（如密码）提供敏感值，请将该值添加到密钥保管库。 在部署过程中检索密钥保管库，如前面的示例所示。 有关详细信息，请参阅[在部署期间传递安全值](resource-manager-keyvault-parameter.md)。 

## <a name="next-steps"></a>后续步骤

- 若要指定如何处理存在于资源组中但未在模板中定义的资源，请参阅 [Azure 资源管理器部署模式](deployment-modes.md)。
- 若要了解如何处理异步 REST 操作，请参阅[跟踪异步 Azure 操作](resource-manager-async-operations.md)。
- 若要详细了解模板，请参阅[了解 Azure 资源管理器模板的结构和语法](resource-group-authoring-templates.md)。

<!--Update_Description: update meta properties, wording update -->