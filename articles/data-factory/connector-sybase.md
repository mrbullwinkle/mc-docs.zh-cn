---
title: 使用 Azure 数据工厂从 Sybase 复制数据 | Microsoft Docs
description: 了解如何通过在 Azure 数据工厂管道中使用复制活动，将数据从 Sybase 复制到支持的接收器数据存储。
services: data-factory
documentationcenter: ''
author: WenJason
manager: digimobile
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
origin.date: 02/07/2018
ms.date: 07/08/2019
ms.author: v-jay
ms.openlocfilehash: 76a9c0f5db3032941a2ddad21430a791edd95056
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67570380"
---
# <a name="copy-data-from-sybase-using-azure-data-factory"></a>使用 Azure 数据工厂从 Sybase 复制数据

本文概述了如何使用 Azure 数据工厂中的复制活动从 Sybase 数据库复制数据。 它是基于概述复制活动总体的[复制活动概述](copy-activity-overview.md)一文。

## <a name="supported-capabilities"></a>支持的功能

可以将数据从 Sybase 数据库复制到任何支持的接收器数据存储。 有关复制活动支持作为源/接收器的数据存储列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)表。

具体而言，此 Sybase 连接器支持：

- SAP Sybase SQL 随处 (ASA) **版本 16 和更高版本**；不支持智能和 ASE。
- 使用**基本**或 **Windows** 身份验证复制数据。

## <a name="prerequisites"></a>先决条件

要使用此 Sybase 连接器，需要：

- 设置自承载集成运行时。 有关详细信息，请参阅[自承载集成运行时](create-self-hosted-integration-runtime.md)一文。
- 在集成运行时计算机上安装 16 或更高版本的[适用于 Sybase iAnywhere.Data.SQLAnywhere 的数据提供程序](https://go.microsoft.com/fwlink/?linkid=324846)。

## <a name="getting-started"></a>入门

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

对于特定于 Sybase 连接器的数据工厂实体，以下部分提供有关用于定义这些实体的属性的详细信息。

## <a name="linked-service-properties"></a>链接服务属性

Sybase 链接的服务支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | type 属性必须设置为：**Sybase** | 是 |
| server | Sybase 服务器的名称。 |是 |
| database | Sybase 数据库的名称。 |是 |
| authenticationType | 用于连接 Sybase 数据库的身份验证类型。<br/>允许值包括：**Basic** 和 **Windows**。 |是 |
| username | 指定用于连接到 Sybase 数据库的用户名。 |是 |
| password | 指定为用户名指定的用户帐户的密码。 将此字段标记为 SecureString 以安全地将其存储在数据工厂中或[引用存储在 Azure Key Vault 中的机密](store-credentials-in-key-vault.md)。 |是 |
| connectVia | 用于连接到数据存储的[集成运行时](concepts-integration-runtime.md)。 如[先决条件](#prerequisites)中所述，需要自承载集成运行时。 |是 |

**示例：**

```json
{
    "name": "SybaseLinkedService",
    "properties": {
        "type": "Sybase",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "authenticationType": "Basic",
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各部分和属性的完整列表，请参阅数据集一文。 本部分提供 Sybase 数据集支持的属性列表。

要从 Sybase 复制数据，请将数据集的 type 属性设置为“RelationalTable”  。 支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 数据集的 type 属性必须设置为：**RelationalTable** | 是 |
| tableName | Sybase 数据库中的表名。 | 否（如果指定了活动源中的“query”） |

**示例**

```json
{
    "name": "SybaseDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<Sybase linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供 Sybase 源支持的属性列表。

### <a name="sybase-as-source"></a>以 Sybase 作为源

要从 Sybase 复制数据，请将复制活动中的源类型设置为“RelationalSource”  。 复制活动源  部分支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 复制活动源的 type 属性必须设置为：**RelationalSource** | 是 |
| query | 使用自定义 SQL 查询读取数据。 例如：`"SELECT * FROM MyTable"`。 | 否（如果指定了数据集中的“tableName”） |

**示例：**

```json
"activities":[
    {
        "name": "CopyFromSybase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Sybase input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "RelationalSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-sybase"></a>Sybase 的数据类型映射

从 Sybase 复制数据时，以下映射用于从 Sybase 数据类型映射到 Azure 数据工厂临时数据类型。 若要了解复制活动如何将源架构和数据类型映射到接收器，请参阅[架构和数据类型映射](copy-activity-schema-and-type-mapping.md)。

Sybase 支持 T-SQL 类型。 有关从 SQL 类型到 Azure 数据工厂临时数据类型的映射表，请参阅 [Azure SQL 数据库连接器 - 数据类型映射](connector-azure-sql-database.md#data-type-mapping-for-azure-sql-database)一节。


## <a name="next-steps"></a>后续步骤
有关 Azure 数据工厂中复制活动支持作为源和接收器的数据存储的列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)。
