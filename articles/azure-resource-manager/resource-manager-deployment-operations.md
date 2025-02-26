---
title: 使用 Azure 资源管理器查看部署历史记录 | Azure
description: 介绍如何通过门户、PowerShell、Azure CLI 和 REST API 查看 Azure Resource Manager 部署操作。
tags: top-support-issue
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: conceptual
origin.date: 05/13/2019
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: 3a101f12b3ada2775c19ba5d3b34c3b369a3d4b1
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337418"
---
# <a name="view-deployment-history-with-azure-resource-manager"></a>使用 Azure 资源管理器查看部署历史记录

使用 Azure 资源管理器可以查看部署历史记录并检查过去部署中的特定操作。 你可以查看已部署的资源，并获取有关任何错误的信息。

如需帮助解决特定部署错误，请参阅[解决使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误](resource-manager-common-deployment-errors.md)。

## <a name="portal"></a>门户

从部署历史记录中获取有关部署的详细信息。

1. 对于部署中涉及的资源组，请注意最后一个部署的状态。 可以选择此状态以获取更多详细信息。

    ![部署状态](./media/resource-manager-deployment-operations/deployment-status.png)
2. 看到最近的部署历史记录。 选择失败的部署。

    ![部署状态](./media/resource-manager-deployment-operations/select-deployment.png)
3. 选择链接，查看部署失败的原因说明。 在下图中，DNS 记录不是唯一的。  

    ![查看失败的部署](./media/resource-manager-deployment-operations/view-error.png)

    此错误消息应足够让可以开始进行故障排除。 但是，如果需要有关完成了哪些任务的更多详细信息，可以查看操作，如下面的步骤所示。
4. 可以查看所有部署操作。 选择任何操作，以查看更多详细信息。

    ![查看操作](./media/resource-manager-deployment-operations/view-operations.png)

    在此示例中，会看到已成功创建存储帐户、虚拟网络和可用性集。 公共 IP 地址失败，未尝试其他资源。
5. 可以通过选择“事件”查看部署的事件  。

    ![查看事件](./media/resource-manager-deployment-operations/view-events.png)
6. 查看部署的所有事件，并选择任何事件以了解更多详细信息。 请注意相关 ID。 与技术支持人员合作排查部署问题时，此值非常有用。

    ![查看事件](./media/resource-manager-deployment-operations/see-all-events.png)

## <a name="powershell"></a>PowerShell
1. 若要获取部署的总体状态，请使用 **Get-AzResourceGroupDeployment** 命令。 

    ```powershell
    Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup
    ```

    也可以筛选结果，以便只获取失败的部署。

    ```powershell
    Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup | Where-Object ProvisioningState -eq Failed
    ```

2. 若要获取相关 ID，请使用：

    ```powershell
    (Get-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -DeploymentName azuredeploy).CorrelationId
    ```

3. 每个部署包括多个操作。 每个操作代表部署过程中的一个步骤。 为了查明部署何处出现问题，通常需要查看部署操作的详细信息。 可以使用 **Get-AzResourceGroupDeploymentOperation** 查看操作状态。

    ```powershell 
    Get-AzResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName vmDeployment
    ```

    它返回多个操作，其中每个操作采用以下格式：

    ```powershell
    Id             : /subscriptions/{guid}/resourceGroups/ExampleGroup/providers/Microsoft.Resources/deployments/Microsoft.Template/operations/A3EB2DA598E0A780
    OperationId    : A3EB2DA598E0A780
    Properties     : @{provisioningOperation=Create; provisioningState=Succeeded; timestamp=2016-06-14T21:55:15.0156208Z;
                   duration=PT23.0227078S; trackingId=11d376e8-5d6d-4da8-847e-6f23c6443fbf;
                   serviceRequestId=0196828d-8559-4bf6-b6b8-8b9057cb0e23; statusCode=OK; targetResource=}
    PropertiesText : {duration:PT23.0227078S, provisioningOperation:Create, provisioningState:Succeeded,
                   serviceRequestId:0196828d-8559-4bf6-b6b8-8b9057cb0e23...}
    ```

4. 若要获取有关失败操作的更多详细信息，请检索状态为“失败”的操作的属性  。

    ```powershell
    (Get-AzResourceGroupDeploymentOperation -DeploymentName Microsoft.Template -ResourceGroupName ExampleGroup).Properties | Where-Object ProvisioningState -eq Failed
    ```

    它返回所有失败的操作，其中每个操作采用以下格式：

    ```powershell
    provisioningOperation : Create
    provisioningState     : Failed
    timestamp             : 2016-06-14T21:54:55.1468068Z
    duration              : PT3.1449887S
    trackingId            : f4ed72f8-4203-43dc-958a-15d041e8c233
    serviceRequestId      : a426f689-5d5a-448d-a2f0-9784d14c900a
    statusCode            : BadRequest
    statusMessage         : @{error=}
    targetResource        : @{id=/subscriptions/{guid}/resourceGroups/ExampleGroup/providers/
                          Microsoft.Network/publicIPAddresses/myPublicIP;
                          resourceType=Microsoft.Network/publicIPAddresses; resourceName=myPublicIP}
    ```

    注意操作的 serviceRequestId 和 trackingId。 与技术支持人员合作排查部署问题时，serviceRequestId 非常有用。 会在下一步使用 trackingId 重点关注特定操作。
5. 若要获取特定失败操作的状态消息，请使用以下命令：

    ```powershell
    ((Get-AzResourceGroupDeploymentOperation -DeploymentName Microsoft.Template -ResourceGroupName ExampleGroup).Properties | Where-Object trackingId -eq f4ed72f8-4203-43dc-958a-15d041e8c233).StatusMessage.error
    ```

    返回：

    ```powershell
    code           message                                                                        details
    ----           -------                                                                        -------
    DnsRecordInUse DNS record dns.chinanorth.cloudapp.chinacloudapi.cn is already used by another public IP. {}
    ```

6. Azure 中的每个部署操作均包括请求和响应内容。 请求内容是在部署过程中发送到 Azure 的内容（例如，创建 VM、OS 磁盘和其他资源）。 响应内容是 Azure 从部署请求发送回的内容。 在部署期间，可以使用 **DeploymentDebugLogLevel** 参数指定将请求和/或响应保留在日志中。 

    使用以下 PowerShell 命令从日志中获取该信息，并将其保存在本地：

    ```powershell
    (Get-AzResourceGroupDeploymentOperation -DeploymentName "TestDeployment" -ResourceGroupName "Test-RG").Properties.request | ConvertTo-Json |  Out-File -FilePath <PathToFile>

    (Get-AzResourceGroupDeploymentOperation -DeploymentName "TestDeployment" -ResourceGroupName "Test-RG").Properties.response | ConvertTo-Json |  Out-File -FilePath <PathToFile>
    ```

## <a name="azure-cli"></a>Azure CLI

1. 使用 **azure group deployment show** 命令获取部署的总体状态。

    ```azurecli
    az group deployment show -g ExampleGroup -n ExampleDeployment
    ```

2. 返回的值之一是 **correlationId**。 此值可用于跟踪相关事件，在与技术支持人员合作排查部署问题时非常有用。

    ```azurecli
    az group deployment show -g ExampleGroup -n ExampleDeployment --query properties.correlationId
    ```

3. 若要查看部署操作，请使用：

    ```azurecli
    az group deployment operation list -g ExampleGroup -n ExampleDeployment
    ```

## <a name="rest"></a>REST

1. 使用 [获取有关模板部署的信息](https://docs.microsoft.com/rest/api/resources/deployments) 操作来获取有关部署的信息。

    ```http
    GET https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}
    ```

    在响应中，请特别注意 **provisioningState**、**correlationId** 和 **error** 元素。 **correlationId** 可用于跟踪相关事件，在与技术支持人员合作排查部署问题时非常有用。

    ```json
    { 
      ...
      "properties": {
        "provisioningState":"Failed",
        "correlationId":"d5062e45-6e9f-4fd3-a0a0-6b2c56b15757",
        ...
        "error":{
          "code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.",
          "details":[{"code":"Conflict","message":"{\r\n  \"error\": {\r\n    \"message\": \"Conflict\",\r\n    \"code\": \"Conflict\"\r\n  }\r\n}"}]
        }  
      }
    }
    ```

2. 使用[列出所有模板部署操作](https://docs.microsoft.com/rest/api/resources/deployments)获取有关部署的信息。 

    ```http
    GET https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}/operations?$skiptoken={skiptoken}&api-version={api-version}
    ```

    响应包含请求和/或响应信息，具体取决于部署期间在 **debugSetting** 属性中指定的内容。

    ```json
    {
      ...
      "properties": 
      {
        ...
        "request":{
          "content":{
            "location":"China North",
            "properties":{
              "accountType": "Standard_LRS"
            }
          }
        },
        "response":{
          "content":{
            "error":{
              "message":"Conflict","code":"Conflict"
            }
          }
        }
      }
    }
    ```

## <a name="next-steps"></a>后续步骤
* 如需帮助解决特定部署错误，请参阅[解决使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误](resource-manager-common-deployment-errors.md)。
* 若要了解如何使用活动日志监视其他类型的操作，请参阅[通过查看活动日志管理 Azure 资源](resource-group-audit.md)。
* 若要在执行部署之前验证部署，请参阅[使用 Azure Resource Manager 模板部署资源组](resource-group-template-deploy.md)。

<!-- Update_Description: update meta properties, wording update, update link -->