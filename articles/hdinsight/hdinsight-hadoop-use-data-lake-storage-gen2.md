---
title: 配合使用 Azure Data Lake Storage Gen2 和 Azure HDInsight 群集
description: 了解如何将 Azure Data Lake Storage Gen2 与 Azure HDInsight 群集配合使用。
services: hdinsight
author: hrasheed-msft
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: howto
origin.date: 05/30/2019
ms.date: 06/24/2019
ms.author: v-yiso
ms.openlocfilehash: 7e106e4fd9252cdb8e18f8609b2e2d5a52825dc4
ms.sourcegitcommit: d15a1a8d21b27196b9097ac24e4e110af5436a99
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/21/2019
ms.locfileid: "67307573"
---
# <a name="use-azure-data-lake-storage-gen2-with-azure-hdinsight-clusters"></a>配合使用 Azure Data Lake Storage Gen2 和 Azure HDInsight 群集

Azure Data Lake Storage Gen2 是构建在 Azure Blob 存储基础之上的，专用于大数据分析的云存储服务。 Data Lake Storage Gen2 将 Azure Blob 存储和 Azure Data Lake Storage Gen1 的功能组合在一起。 组合后的服务既可提供 Azure Data Lake Storage Gen1 的功能（例如文件系统语义、目录级别和文件级别的安全性及可伸缩性），也可提供 Azure Blob 存储的高性价比、分层存储、高可用性和灾难恢复功能。

## <a name="data-lake-storage-gen2-availability"></a>Data Lake Storage Gen2 可用性

Data Lake Storage Gen2 能够以默认存储和附加存储帐户的形式用作几乎所有 Azure HDInsight 群集类型的存储选项。 但是，HBase 只能有一个 Data Lake Storage Gen2 帐户。

> [!Note] 
> 选择 Data Lake Storage Gen2 作为**主要存储类型**后，无法选择 Data Lake Storage Gen1 帐户作为附加存储。

## <a name="create-a-cluster-with-data-lake-storage-gen2-through-the-azure-portal"></a>通过 Azure 门户创建使用 Data Lake Storage Gen2 的群集

若要创建将 Data Lake Storage Gen2 用作存储的 HDInsight 群集，请遵循以下步骤配置 Data Lake Storage Gen2 帐户。

### <a name="create-a-user-assigned-managed-identity"></a>创建用户分配的托管标识

创建用户分配的托管标识（如果还没有）。 

![创建用户分配的托管标识](./media/hdinsight-hadoop-use-data-lake-storage-gen2/create-user-assigned-managed-identity-portal.png)

### <a name="create-a-data-lake-storage-gen2-account"></a>创建 Data Lake Storage Gen2 帐户

创建 Azure Data Lake Storage Gen2 存储帐户。 确保已启用“分层命名空间”选项  。 有关详细信息，请参阅[快速入门：创建 Azure Data Lake Storage Gen2 存储帐户](../storage/blobs/data-lake-storage-quickstart-create-account.md)。

![显示 Azure 门户中存储帐户创建情况的屏幕截图](./media/hdinsight-hadoop-data-lake-storage-gen2/azure-data-lake-storage-account-create-advanced.png)

### <a name="set-up-permissions-for-the-managed-identity-on-the-data-lake-storage-gen2-account"></a>在 Data Lake Storage Gen2 帐户中设置托管标识的权限

将托管标识分配到存储帐户上的“存储 Blob 数据所有者”角色  。 有关详细信息，请参阅[使用 RBAC 管理对 Azure Blob 和队列数据的访问权限（预览版）](../storage/common/storage-auth-aad-rbac.md)。

1. 在 [Azure 门户](https://portal.azure.com)中转到自己的存储帐户。
1. 选择存储帐户，然后选择“访问控制(IAM)”以显示该帐户的访问控制设置  。 选择“角色分配”  选项卡以查看角色分配列表。
    
![显示存储访问控制设置的屏幕截图](./media/hdinsight-hadoop-data-lake-storage-gen2/portal-access-control.png)
    
1. 选择“+ 添加角色分配”按钮以添加一个新角色  。
1. 在“添加角色分配”窗口中，选择“存储 Blob 数据所有者”角色   。 然后，选择具有托管标识和存储帐户的订阅。 接下来，搜索并找到之前创建的用户分配托管标识。 最后，选择托管标识，它将在“选定成员”下列出  。
    
![显示如何分配 RBAC 角色的屏幕截图](./media/hdinsight-hadoop-data-lake-storage-gen2/add-rbac-role3.png)
    
1. 选择“其他安全性验证”  。 现在，选定的用户分配的标识会列在选定的角色下。
1. 此初始设置完成后，可通过门户创建群集。 群集必须与存储帐户位于同一 Azure 区域中。 在群集创建菜单的“存储”部分，选择以下选项  ：
        
* 对于“主要存储类型”，请选择“Azure Data Lake Storage Gen2”   。
* 在“选择存储帐户”下，搜索并选择新建的 Data Lake Storage Gen2 存储帐户  。
        
  ![用于配合使用 Data Lake Storage Gen2 和 Azure HDInsight 的存储设置](./media/hdinsight-hadoop-data-lake-storage-gen2/primary-storage-type-adls-gen2.png)
    
* 在“标识”下，选择正确的订阅和新建的用户分配的托管标识  。
        
  ![用于配合使用 Data Lake Storage Gen2 和 Azure HDInsight 的标识设置](./media/hdinsight-hadoop-data-lake-storage-gen2/managed-identity-cluster-creation.png)
        
> [!Note]
> 若要添加辅助 Data Lake Storage Gen2 帐户，请在存储帐户级别直接将此前创建的托管标识分配给要添加的新 Data Lake Storage Gen2 存储帐户。请注意，不支持在 HDInsight 上通过“其他存储帐户”边栏选项卡添加辅助 Data Lake Storage Gen2 帐户。 

## <a name="create-a-cluster-with-data-lake-storage-gen2-through-the-azure-cli"></a>通过 Azure CLI 创建使用 Data Lake Storage Gen2 的群集

可以[下载示例模板文件](https://github.com/Azure-Samples/hdinsight-data-lake-storage-gen2-templates/blob/master/hdinsight-adls-gen2-template.json)并[下载示例参数文件](https://github.com/Azure-Samples/hdinsight-data-lake-storage-gen2-templates/blob/master/parameters.json)。 使用模板之前，请将字符串 `<SUBSCRIPTION_ID>` 替换为实际的 Azure 订阅 ID。 此外，请将字符串 `<PASSWORD>` 替换为所选的密码，以设置用于登录到群集的密码以及 SSH 密码。

以下代码片段将会执行下述初始步骤：

1. 登录到 Azure 帐户。
1. 设置要在其中执行创建操作的活动订阅。
1. 为新的部署活动创建名为 `hdinsight-deployment-rg` 的新资源组。
1. 创建名为 `test-hdinsight-msi` 的用户分配的托管标识。
1. 将一个扩展添加到 Azure CLI，以使用 Data Lake Storage Gen2 的功能。
1. 使用 `--hierarchical-namespace true` 标志创建名为 `hdinsightadlsgen2` 的新 Data Lake Storage Gen2 帐户。

```azurecli
az login
az account set --subscription <subscription_id>

# Create resource group
az group create --name hdinsight-deployment-rg --location chinaeast

# Create managed identity
az identity create -g hdinsight-deployment-rg -n test-hdinsight-msi

az extension add --name storage-preview

az storage account create --name hdinsightadlsgen2 \
    --resource-group hdinsight-deployment-rg \
    --location eastus --sku Standard_LRS \
    --kind StorageV2 --hierarchical-namespace true
```

接下来，登录到门户。 根据[使用 Azure 门户](hdinsight-hadoop-use-data-lake-storage-gen2.md)中的步骤 3 所述，将新的用户分配的托管标识添加到存储帐户中的“存储 Blob 数据参与者”角色。 

为用户分配的托管标识分配角色后，使用以下代码片段部署模板。

```azurecli
az group deployment create --name HDInsightADLSGen2Deployment \
    --resource-group hdinsight-deployment-rg \
    --template-file hdinsight-adls-gen2-template.json \
    --parameters parameters.json
```

## <a name="access-control-for-data-lake-storage-gen2-in-hdinsight"></a>HDInsight 中 Data Lake Storage Gen2 的访问控制

### <a name="what-kinds-of-permissions-does-data-lake-storage-gen2-support"></a>Data Lake Storage Gen2 支持哪些类型的权限？

Data Lake Storage Gen2 使用一个支持基于角色的访问控制 (RBAC) 和类似于 POSIX 的访问控制列表 (ACL) 的访问控制模型。 Data Lake Storage Gen1 仅支持用于控制数据访问的访问控制列表。

RBAC 使用角色分配有效地将权限集应用到 Azure 资源的用户、组和服务主体。 一般情况下，这些 Azure 资源限制为顶级资源（例如 Azure 存储帐户）。 对于 Azure 存储以及 Azure Data Lake Storage Gen2，此机制已扩展到文件系统资源。

 有关使用 RBAC 分配文件权限的详细信息，请参阅 [Azure 基于角色的访问控制 (RBAC)](../storage/blobs/data-lake-storage-access-control.md#azure-role-based-access-control-rbac)。

有关使用 ACL 分配文件权限的详细信息，请参阅[对文件和目录应用访问控制列表](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories)。

### <a name="how-do-i-control-access-to-my-data-in-data-lake-storage-gen2"></a>如何在 Data Lake Storage Gen2 中控制对数据的访问？

HDInsight 群集在 Data Lake Storage Gen2 中访问文件的能力通过托管标识进行控制。 托管标识是在 Azure Active Directory (Azure AD) 中注册的标识，其凭据由 Azure 管理。 使用托管标识，无需在 Azure AD 中注册服务主体，也无需维护证书等凭据。

Azure 服务有两种类型的托管标识：系统分配的托管标识和用户分配的托管标识。 HDInsight 使用用户分配的托管标识来访问 Data Lake Storage Gen2。 用户分配的托管标识作为独立的 Azure 资源创建。 在创建过程中，Azure 会在由所用订阅信任的 Azure AD 租户中创建一个标识。 在创建标识后，可以将标识分配到一个或多个 Azure 服务实例。

用户分配标识的生命周期与它所分配到的 Azure 服务实例的生命周期是分开管理的。 

### <a name="how-do-i-set-permissions-for-azure-ad-users-to-query-data-in-data-lake-storage-gen2-by-using-hive-or-other-services"></a>如何设置 Azure AD 用户的权限，以使用 Hive 或其他服务在 Data Lake Storage Gen2 中查询数据？

若要为用户设置权限以查询数据，请将 Azure AD 安全组用作 ACL 中分配的主体。 不要直接向单个用户或服务主体分配文件访问权限。 使用 Azure AD 安全组控制权限流时，可以添加和删除用户或服务主体，而无需将 ACL 重新应用到整个目录结构。 只需要从相应的 Azure AD 安全组添加或删除用户。 ACL 不可继承，因此，重新应用 ACL 需要更新针对每个文件和子目录应用的 ACL。

## <a name="next-steps"></a>后续步骤

* [Azure HDInsight 与 Data Lake Storage Gen2 预览版集成 - ACL 和安全更新](https://azure.microsoft.com/blog/azure-hdinsight-integration-with-data-lake-storage-gen-2-preview-acl-and-security-update/)
* [Azure Data Lake Storage Gen2 简介](../storage/blobs/data-lake-storage-introduction.md)
