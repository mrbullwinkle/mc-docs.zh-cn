---
title: Key Vault 机密与 Azure 资源管理器模板 | Azure
description: 说明在部署期间如何以参数形式从密钥保管库传递机密。
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: conceptual
origin.date: 05/09/2019
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: 58267b3492e050bb3f3c97550a9fa35d9b31a5ba
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337404"
---
# <a name="use-azure-key-vault-to-pass-secure-parameter-value-during-deployment"></a>在部署过程中使用 Azure Key Vault 传递安全参数值

在部署过程中，可以从 [Azure Key Vault](../key-vault/key-vault-whatis.md) 中检索一个安全值，而不是直接在模板或参数文件中放置该值（如密码）。 通过引用参数文件中的密钥保管库和密钥来检索值。 值永远不会公开，因为仅引用其密钥保管库 ID。 密钥保管库可以与要部署到的资源组位于不同的订阅中。

## <a name="deploy-key-vaults-and-secrets"></a>部署密钥保管库和机密

若要在模板部署期间访问密钥保管库，请将密钥保管库上的 `enabledForTemplateDeployment` 设置为 `true`。

以下 Azure CLI 和 Azure PowerShell 示例显示如何创建密钥保管库和添加机密。

```azurecli
location='chinaeast'

az group create --name $resourceGroupName --location $location

az keyvault create \
  --name $keyVaultName \
  --resource-group $resourceGroupName \
  --location $location \
  --enabled-for-template-deployment true
az keyvault secret set --vault-name $keyVaultName --name "ExamplePassword" --value "hVFkk965BuUv"
```

```azurepowershell
$location='chinaeast'

New-AzResourceGroup -Name $resourceGroupName -Location $location

New-AzKeyVault `
  -VaultName $keyVaultName `
  -resourceGroupName $resourceGroupName `
  -Location $location `
  -EnabledForTemplateDeployment
$secretvalue = ConvertTo-SecureString 'hVFkk965BuUv' -AsPlainText -Force
$secret = Set-AzKeyVaultSecret -VaultName $keyVaultName -Name 'ExamplePassword' -SecretValue $secretvalue
```

作为密钥保管库的所有者，你可以自动获得创建机密的权限。 如果使用机密的用户不是密钥保管库的所有者，请使用以下命令授予访问权限：

```azurecli
userPrincipalName='<your-email-address-associated-with-your-subscription>'

az keyvault set-policy \
  --upn $userPrincipalName \
  --name $keyVaultName \
  --secret-permissions set delete get list
```

```azurepowershell
$userPrincipalName='<your-email-address-associated-with-your-subscription>'

Set-AzKeyVaultAccessPolicy `
  -VaultName $keyVaultName `
  -UserPrincipalName $userPrincipalName `
  -PermissionsToSecrets set,delete,get,list
```

若要详细了解如何创建密钥保管库和添加机密，请参阅：

- [使用 CLI 设置和检索机密](../key-vault/quick-create-cli.md)
- [使用 Powershell 设置和检索机密](../key-vault/quick-create-powershell.md)
- [使用门户设置和检索机密](../key-vault/quick-create-portal.md)
- [使用 .NET 设置和检索机密](../key-vault/quick-create-net.md)
- [使用 Node.js 设置和检索机密](../key-vault/quick-create-node.md)

## <a name="grant-access-to-the-secrets"></a>授予对机密的访问权限

部署模板的用户必须在资源组和密钥保管库范围内具有 `Microsoft.KeyVault/vaults/deploy/action` 权限。 [所有者](../role-based-access-control/built-in-roles.md#owner)和[参与者](../role-based-access-control/built-in-roles.md#contributor)角色均授予该访问权限。 如果你创建了密钥保管库，则你是所有者，因此具有权限。

以下过程展示了如何创建具有最小权限的角色，以及如何分配用户

1. 创建自定义角色定义 JSON 文件：

    ```json
    {
      "Name": "Key Vault resource manager template deployment operator",
      "IsCustom": true,
      "Description": "Lets you deploy a resource manager template with the access to the secrets in the Key Vault.",
      "Actions": [
        "Microsoft.KeyVault/vaults/deploy/action"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ]
    }
    ```
    将“00000000-0000-0000-0000-000000000000”替换为订阅 ID。

2. 使用 JSON 文件创建新角色：

    ```azurecli
    az role definition create --role-definition "<PathToRoleFile>"
    az role assignment create \
      --role "Key Vault resource manager template deployment operator" \
      --assignee $userPrincipalName \
      --resource-group $resourceGroupName
    ```

    ```azurepowershell
    New-AzRoleDefinition -InputFile "<PathToRoleFile>" 
    New-AzRoleAssignment `
      -ResourceGroupName $resourceGroupName `
      -RoleDefinitionName "Key Vault resource manager template deployment operator" `
      -SignInName $userPrincipalName
    ```

    此示例在资源组级别为用户分配自定义角色。  
    
  <!-- Not Available on [Managed Application](../managed-applications/overview.md)-->

## <a name="reference-secrets-with-static-id"></a>通过静态 ID 引用机密

使用此方法，可以在参数文件（而不是在模板）中引用密钥保管库。 下图显示了参数文件如何引用机密并将该值传递到模板。

![资源管理器密钥保管库集成静态 ID 图](./media/resource-manager-keyvault-parameter/statickeyvault.png)

[教程：在资源管理器模板部署中集成 Azure Key Vault](./resource-manager-tutorial-use-key-vault.md) 使用了此方法。

以下模板部署包含管理员密码的 SQL Server。 密码参数设置为安全字符串。 但是，此模板未指定该值的来源。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "adminLogin": {
      "type": "string"
    },
    "adminPassword": {
      "type": "securestring"
    },
    "sqlServerName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "name": "[parameters('sqlServerName')]",
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2015-05-01-preview",
      "location": "[resourceGroup().location]",
      "tags": {},
      "properties": {
        "administratorLogin": "[parameters('adminLogin')]",
        "administratorLoginPassword": "[parameters('adminPassword')]",
        "version": "12.0"
      }
    }
  ],
  "outputs": {
  }
}
```

现在，为上述模板创建参数文件。 在参数文件中，指定与模板中的参数名称匹配的参数。 对于参数值，请从 key vault 中引用机密。 可以通过传递密钥保管库的资源标识符和机密的名称来引用机密：

在以下参数文件中，密钥保管库机密必须已存在，而且你为其资源 ID 提供了静态值。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "adminLogin": {
            "value": "exampleadmin"
        },
        "adminPassword": {
            "reference": {
              "keyVault": {
                "id": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"
              },
              "secretName": "ExamplePassword"
            }
        },
        "sqlServerName": {
            "value": "<your-server-name>"
        }
    }
}
```

如果需要使用当前版本以外的机密版本，请使用 `secretVersion` 属性。

```json
"secretName": "ExamplePassword",
"secretVersion": "cd91b2b7e10e492ebb870a6ee0591b68"
```

部署模板并传入参数文件：

<!-- Verify successful with --template-uri parameter-->

对于 Azure CLI，请使用：

```azurecli
deploymentName="<your-deployment-name>"
az group create --name $resourceGroupName --location $location
az group deployment create \
    --name $deploymentName \
    --resource-group $resourceGroupName \
    --template-uri <The Template File URI> \
    --parameters <The Parameter File>
```

对于 PowerShell，请使用：

```powershell
$deploymentName="<your-deployment-name>"
New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
  -Name $deploymentName `
  -ResourceGroupName $resourceGroupName `
  -TemplateUri <The Template File URI> `
  -TemplateParameterFile <The Parameter File>
```

## <a name="reference-secrets-with-dynamic-id"></a>通过动态 ID 引用机密

<!--Verify sucessfully-->

上一部分介绍了如何通过参数传递密钥保管库机密的静态资源 ID。 但是，在某些情况下，需要引用随当前部署而变的密钥保管库机密。 或者，你需要将参数值传递到模板而不在参数文件中创建引用参数。 在任何一种情况下，均可使用链接模板为密钥保管库机密动态生成资源 ID。

不能在参数文件中动态生成资源 ID，因为参数文件中不允许使用模板表达式。 

在父模板中添加链接模板，然后传入包含动态生成的资源 ID 的参数。 下图显示链接模板中的参数如何引用机密。

![动态 ID](./media/resource-manager-keyvault-parameter/dynamickeyvault.png)

[以下模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-key-vault-use-dynamic-id)动态创建 Key Vault ID 并将其作为参数传递。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "The location where the resources will be deployed."
            }
        },
        "vaultName": {
            "type": "string",
            "metadata": {
                "description": "The name of the keyvault that contains the secret."
            }
        },
        "secretName": {
            "type": "string",
            "metadata": {
                "description": "The name of the secret."
            }
        },
        "vaultResourceGroupName": {
            "type": "string",
            "metadata": {
                "description": "The name of the resource group that contains the keyvault."
            }
        },
        "vaultSubscription": {
            "type": "string",
            "defaultValue": "[subscription().subscriptionId]",
            "metadata": {
                "description": "The name of the subscription that contains the keyvault."
            }
        },
        "_artifactsLocation": {
            "type": "string",
            "metadata": {
                "description": "The base URI where artifacts required by this template are located. When the template is deployed using the accompanying scripts, a private location in the subscription will be used and this value will be automatically generated."
            },
            "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-key-vault-use-dynamic-id/"
        },
        "_artifactsLocationSasToken": {
            "type": "securestring",
            "metadata": {
                "description": "The sasToken required to access _artifactsLocation.  When the template is deployed using the accompanying scripts, a sasToken will be automatically generated."
            },
            "defaultValue": ""
        }
    },
    "resources": [
        {
            "apiVersion": "2018-05-01",
            "name": "dynamicSecret",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                    "contentVersion": "1.0.0.0",
                    "uri": "[uri(parameters('_artifactsLocation'), concat('./nested/sqlserver.json', parameters('_artifactsLocationSasToken')))]"
                },
                "parameters": {
                    "location": {
                        "value": "[parameters('location')]"
                    },
                    "adminLogin": {
                        "value": "ghuser"
                    },
                    "adminPassword": {
                        "reference": {
                            "keyVault": {
                                "id": "[resourceId(parameters('vaultSubscription'), parameters('vaultResourceGroupName'), 'Microsoft.KeyVault/vaults', parameters('vaultName'))]"
                            },
                            "secretName": "[parameters('secretName')]"
                        }
                    }
                }
            }
        }
    ],
    "outputs": {
        "sqlFQDN": {
            "type": "string",
            "value": "[reference('dynamicSecret').outputs.sqlFQDN.value]"
        }
    }
}
```

部署前面的模板，并为参数提供值。 可以使用 GitHub 中的示例模板，但必须提供环境的参数值。

对于 Azure CLI，请使用：

```azurecli
secretName="<your-secret-name>"
keyVaultResourceGroupName="<your-keyvault-resource-group-name>"
az group create --name $resourceGroupName --location $location
az group deployment create \
    --name $deploymentName \
    --resource-group $resourceGroupName \
    --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-key-vault-use-dynamic-id/azuredeploy.json \
    --parameters vaultName=$keyVaultName vaultResourceGroupName=$keyVaultResourceGroupName secretName=$secretName
```

对于 PowerShell，请使用：

```powershell
$secretName="<your-secret-name>"
$keyVaultResourceGroupName="<your-keyvault-resource-group-name>"
New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment `
  -Name $deploymentName `
  -ResourceGroupName $resourceGroupName `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-key-vault-use-dynamic-id/azuredeploy.json `
  -vaultName $keyVaultName -vaultResourceGroupName $keyVaultResourceGroupName -secretName $secretName
```

## <a name="next-steps"></a>后续步骤

- 有关密钥保管库的一般信息，请参阅[什么是 Azure 密钥保管库？](../key-vault/key-vault-overview.md)。
- 有关引用密钥机密的完整示例，请参阅 [密钥保管库示例](https://github.com/rjmax/ArmExamples/tree/master/keyvaultexamples)。

<!--Update_Description: update meta properties, wording update, update cmdlet sample-->