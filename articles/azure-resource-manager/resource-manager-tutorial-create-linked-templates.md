---
title: 创建 Azure 资源管理器链接模板 | Azure
description: 了解如何创建 Azure 资源管理器链接模板，以便创建虚拟机。
services: azure-resource-manager
documentationcenter: ''
author: rockboyfor
manager: digimobile
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
origin.date: 03/18/2019
ms.date: 07/22/2019
ms.topic: tutorial
ms.author: v-yeche
ms.openlocfilehash: 2233cef37cc3ed874ea9e5cc05bbe177d82ea3b5
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337389"
---
<!--Verify successfully-->
# <a name="tutorial-create-linked-azure-resource-manager-templates"></a>教程：创建 Azure 资源管理器链接模板

了解如何创建 Azure 资源管理器链接模板。 使用链接模板时，可以通过一个模板调用另一个模板。 它非常适用于模板的模块化。 在本教程中使用的模板与在[教程：使用依赖资源创建 Azure 资源管理器模板](./resource-manager-tutorial-create-templates-with-dependent-resources.md)中使用的模板相同，该模板用于创建虚拟机、虚拟网络以及其他依赖资源（包括存储帐户）。 请将存储帐户资源创建功能分隔到链接的模板。

调用链接的模板就像执行函数调用一样。  你还将了解如何将参数值传递给链接的模板，以及如何从链接的模板中获取“返回值”。

本教程涵盖以下任务：

> [!div class="checklist"]
> * 打开快速入门模板
> * 创建链接模板
> * 上传链接模板
> * 链接到链接模板
> * 配置依赖项
> * 部署模板
> * 其他做法

有关详细信息，请参阅[部署 Azure 资源时使用链接的和嵌套的模板](./resource-group-linked-templates.md)。

如果没有 Azure 订阅，请在开始前[创建一个试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>先决条件

若要完成本文，需要做好以下准备：

* 包含[资源管理器工具扩展](./resource-manager-quickstart-create-templates-use-visual-studio-code.md#prerequisites)的 [Visual Studio Code](https://code.visualstudio.com/)。
* 若要提高安全性，请使用为虚拟机管理员帐户生成的密码。 以下是密码生成示例：

    ```azurecli
    openssl rand -base64 32
    ```
    Azure Key Vault 旨在保护加密密钥和其他机密。 有关详细信息，请参阅[教程：在资源管理器模板部署中集成 Azure Key Vault](./resource-manager-tutorial-use-key-vault.md)。 我们还建议你每三个月更新一次密码。

## <a name="open-a-quickstart-template"></a>打开快速入门模板

Azure 快速入门模板是资源管理器模板的存储库。 无需从头开始创建模板，只需找到一个示例模板并对其自定义即可。 本教程中使用的模板称为[部署简单的 Windows VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows/)。 这是在[教程：使用依赖的资源创建 Azure 资源管理器模板](./resource-manager-tutorial-create-templates-with-dependent-resources.md)中使用的同一模板。 请保存同一模板的两个副本，用作：

* **主模板**：创建除存储帐户之外的所有资源。
* **链接模板**：创建存储帐户。

1. 在 Visual Studio Code 中，选择“文件”>“打开文件”。  
2. 在“文件名”中粘贴以下 URL： 

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json
    ```
3. 选择“打开”以打开该文件。 
4. 有五个通过此模板定义的资源：

    * `Microsoft.Storage/storageAccounts`。
    * `Microsoft.Network/publicIPAddresses`。
    * `Microsoft.Network/virtualNetworks`。
    * `Microsoft.Network/networkInterfaces`。
    * `Microsoft.Compute/virtualMachines`。
    
    <!-- Not Available on templates reference-->
    
     在自定义模板之前，有必要对模板架构进行一些基本的了解。
5. 选择“文件”>“另存为”，将该文件的副本保存到名为 **azuredeploy.json** 的本地计算机。  
6. 选择“文件”  >  “另存为”，创建名为 **linkedTemplate.json** 的另一文件副本。

## <a name="create-the-linked-template"></a>创建链接模板

链接模板可创建存储帐户。 链接的模板可以用作独立模板来创建存储帐户。 在本教程中，链接的模板采用两个参数，并将值传递给主模板。 此“返回”值在 `outputs` 元素中定义。

1. 在 Visual Studio Code 中打开 linkedTemplate.json（如果此文件尚未打开）  。
2. 进行以下更改：

    * 删除除 location 之外的所有参数  。
    * 添加名为 **storageAccountName** 的参数。
        ```json
        "storageAccountName":{
          "type": "string",
          "metadata": {
              "description": "Azure Storage account name."
          }
        },
        ```
        存储帐户名称和位置作为参数从主模板传递给链接的模板。

    * 删除 **variables** 元素以及所有变量定义。
    * 删除除存储帐户之外的所有资源。 删除总共四项资源。
    * 将存储帐户资源的 **name** 元素的值更新为：

        ```json
          "name": "[parameters('storageAccountName')]",
        ```

    * 更新 **outputs** 元素，使之如下所示：

        ```json
        "outputs": {
          "storageUri": {
              "type": "string",
              "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
            }
        }
        ```
        **storageUri** 在主模板中是虚拟机资源定义所需要的。  请将值作为输出值传回主模板。

        完成后，模板应如下所示：

        ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "storageAccountName": {
              "type": "string",
              "metadata": {
                "description": "Azure Storage account name."
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
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "name": "[parameters('storageAccountName')]",
              "location": "[parameters('location')]",
              "apiVersion": "2018-07-01",
              "sku": {
                "name": "Standard_LRS"
              },
              "kind": "Storage",
              "properties": {}
            }
          ],
          "outputs": {
            "storageUri": {
              "type": "string",
              "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
            }
          }
        }
        ```
3. 保存更改。

## <a name="upload-the-linked-template"></a>上传链接模板

<!--NOTICE: CLOUD SHELL IS INVALID ON MOONCAKE-->

主模板和链接的模板必须能够从运行部署时所在的位置进行访问。 在本教程中使用的本地 Shell 部署方法就是在[教程：使用依赖的资源创建 Azure 资源管理器模板](./resource-manager-tutorial-create-templates-with-dependent-resources.md)中使用的。 主模板 (azuredeploy.json) 保存到本地电脑。 链接的模板 (linkedTemplate.json) 必须在某个位置安全地共享。 以下 PowerShell 脚本创建一个 Azure 存储帐户，将模板上传到该存储帐户，然后生成一个 SAS 令牌，以便授予对模板文件的受限访问权限。 为了简化本教程，该脚本会从共享位置下载一个完成的链接模板。 

<!--NOTICE: CLOUD SHELL IS INVALID ON MOONCAKE-->

> [!NOTE]
> 脚本将 SAS 令牌限制为在八小时内使用。 如果需要更多时间来完成本教程，请将到期时间推后。

```powershell
$projectNamePrefix = Read-Host -Prompt "Enter a project name:"   # This name is used to generate names for Azure resources, such as storage account name.
$location = Read-Host -Prompt "Enter a location (i.e. chinaeast)"

$resourceGroupName = $projectNamePrefix + "rg"
$storageAccountName = $projectNamePrefix + "store"
$containerName = "linkedtemplates" # The name of the Blob container to be created.

$linkedTemplateURL = "https://armtutorials.blob.core.windows.net/linkedtemplates/linkedStorageAccount.json" # A completed linked template used in this tutorial.
$fileName = "linkedStorageAccount.json" # A file name used for downloading and uploading the linked template.

# Download the tutorial linked template
Invoke-WebRequest -Uri $linkedTemplateURL -OutFile "$home/$fileName"

# Create a resource group
New-AzResourceGroup -Name $resourceGroupName -Location $location

# Create a storage account
$storageAccount = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -Location $location `
    -SkuName "Standard_LRS"

$context = $storageAccount.Context

# Create a container
New-AzStorageContainer -Name $containerName -Context $context

# Upload the linked template
Set-AzStorageBlobContent `
    -Container $containerName `
    -File "$home/$fileName" `
    -Blob $fileName `
    -Context $context

# Generate a SAS token
$templateURI = New-AzStorageBlobSASToken `
    -Context $context `
    -Container $containerName `
    -Blob $fileName `
    -Permission r `
    -ExpiryTime (Get-Date).AddHours(8.0) `
    -FullUri

echo "You need the following values later in the tutorial:"
echo "Resource Group Name: $resourceGroupName"
echo "Linked template URI with SAS token: $templateURI"
```

<!--Not Available on Azure cloud shell-->

在实践中，请在部署主模板时生成一个 SAS 令牌，让该 SAS 令牌在更短的时间范围内到期，以增强安全性。 有关详细信息，请参阅[在部署期间提供 SAS 令牌](./resource-manager-powershell-sas-token.md#provide-sas-token-during-deployment)。

## <a name="call-the-linked-template"></a>调用链接模板

主模板称为 azuredeploy.json。

1. 在 Visual Studio Code 中打开 azuredeploy.json（如果尚未打开）  。
2. 从模板中删除存储帐户资源定义：

    ```json
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "apiVersion": "2018-07-01",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {}
    },
    ```
3. 将以下 json 代码片段添加到存储帐户定义所在的位置：

    ```json
    {
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2018-05-01",
      "properties": {
          "mode": "Incremental",
          "templateLink": {
              "uri":"https://armtutorials.blob.core.windows.net/linkedtemplates/linkedStorageAccount.json"
          },
          "parameters": {
              "storageAccountName":{"value": "[variables('storageAccountName')]"},
              "location":{"value": "[parameters('location')]"}
          }
      }
    },
    ```

    请注意以下详细信息：

    * 主模板中的 `Microsoft.Resources/deployments` 资源用于链接到另一模板。
    * `deployments` 资源的名称为 `linkedTemplate`。 该名称用于[配置依赖项](#configure-dependency)。
    * 在调用链接模板时，只能使用[增量](./deployment-modes.md)部署模式。
    * `templateLink/uri` 包含链接模板 URI。 将值更新为在上传链接模板（与 SAS 令牌配合使用的模板）时获取的 URI。
    * 请使用 `parameters` 将值从主模板传递到链接模板。
4. 确保已将 `uri` 元素的值更新为在上传链接模板（与 SAS 令牌配合使用的模板）时获取的值。 在实践中，需为 URI 提供一个参数。
5. 保存修订的模板

## <a name="configure-dependency"></a>配置依赖项

回想一下，在[教程：使用依赖资源创建 Azure 资源管理器模板](./resource-manager-tutorial-create-templates-with-dependent-resources.md)中，虚拟机资源依赖于存储帐户：

![Azure 资源管理器模板依赖项图](./media/resource-manager-tutorial-create-linked-templates/resource-manager-template-visual-studio-code-dependency-diagram.png)

由于存储帐户现在已在链接模板中定义，因此必须更新 `Microsoft.Compute/virtualMachines` 资源的下述两个元素。

* 重新配置 `dependOn` 元素。 存储帐户定义移到链接模板。
* 重新配置 `properties/diagnosticsProfile/bootDiagnostics/storageUri` 元素。 在[创建链接模板](#create-the-linked-template)中，已添加输出值：

    ```json
    "outputs": {
        "storageUri": {
            "type": "string",
            "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
            }
    }
    ```
    该值是主模板需要的。

1. 在 Visual Studio Code 中打开 azuredeploy.json（如果尚未打开）。
2. 扩展虚拟机资源定义，更新 **dependsOn**，如以下屏幕截图所示：

    ![Azure 资源管理器链接模板可配置依赖项 ](./media/resource-manager-tutorial-create-linked-templates/resource-manager-template-linked-templates-configure-dependency.png)

    *linkedTemplate* 是部署资源的名称。  
3. 更新 **properties/diagnosticsProfile/bootDiagnostics/storageUri**，如上一屏幕截图所示。
4. 保存修订的模板。

## <a name="deploy-the-template"></a>部署模板

有关部署过程，请参阅[部署模板](./resource-manager-tutorial-create-templates-with-dependent-resources.md#deploy-the-template)部分。 使用与存储帐户相同的资源组名称来存储链接模板。 这样可以更方便地在下一部分清理资源。 若要提高安全性，请使用为虚拟机管理员帐户生成的密码。 请参阅[先决条件](#prerequisites)。

## <a name="clean-up-resources"></a>清理资源

不再需要 Azure 资源时，请通过删除资源组来清理部署的资源。

1. 在 Azure 门户上的左侧菜单中选择“资源组”  。
2. 在“按名称筛选”字段中输入资源组名称。 
3. 选择资源组名称。  应会看到，该资源组中总共有六个资源。
4. 在顶部菜单中选择“删除资源组”。 

## <a name="additional-practice"></a>其他做法

若要改进项目，请对已完成的项目进行下述其他更改：

1. 修改主模板 (azuredeploy.json)，使之通过参数获取链接模板 URI 值。
2. 请在部署主模板时生成 SAS 令牌，而不是在上传链接模板时生成该令牌。 有关详细信息，请参阅[在部署期间提供 SAS 令牌](./resource-manager-powershell-sas-token.md#provide-sas-token-during-deployment)。

## <a name="next-steps"></a>后续步骤

本教程介绍了如何将模板模块化为主模板和链接模板。 若要了解如何使用虚拟机扩展执行部署后任务，请参阅：

> [!div class="nextstepaction"]
> [部署虚拟机扩展](./resource-manager-tutorial-deploy-vm-extensions.md)

<!-- Update_Description: update meta properties, wording update -->