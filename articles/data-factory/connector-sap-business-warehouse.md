---
title: 使用 Azure 数据工厂从 SAP BW 复制数据 | Microsoft Docs
description: 了解如何通过在 Azure 数据工厂管道中使用复制活动，将数据从 SAP Business Warehouse 复制到支持的接收器数据存储。
services: data-factory
documentationcenter: ''
author: WenJason
manager: digimobile
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
origin.date: 08/07/2018
ms.date: 07/08/2019
ms.author: v-jay
ms.openlocfilehash: e383b32808eb4884bafd995054d4f8dd4e260cf7
ms.sourcegitcommit: 5191c30e72cbbfc65a27af7b6251f7e076ba9c88
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67570393"
---
# <a name="copy-data-from-sap-business-warehouse-using-azure-data-factory"></a>使用 Azure 数据工厂从 SAP Business Warehouse 复制数据

本文概述了如何使用 Azure 数据工厂中的复制活动从 SAP Business Warehouse (BW) 复制数据。 它是基于概述复制活动总体的[复制活动概述](copy-activity-overview.md)一文。

## <a name="supported-capabilities"></a>支持的功能

可以将数据从 SAP Business Warehouse 复制到任何受支持的接收器数据存储。 有关复制活动支持作为源/接收器的数据存储列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)表。

具体而言，此 SAP Business Warehouse 连接器支持：

- SAP Business Warehouse 版本 7.x  。
- 使用 MDX 查询从 InfoCubes 和 QueryCubes  （包括 BEx 查询）复制数据。
- 使用基本身份验证复制数据。

## <a name="prerequisites"></a>先决条件

若要使用此 SAP Business Warehouse 连接器，需要：

- 设置自承载集成运行时。 有关详细信息，请参阅[自我托管集成运行时](create-self-hosted-integration-runtime.md)一文。
- 在集成运行时计算机上安装 SAP NetWeaver 库  。 可以从 SAP 管理员处或直接从 [SAP 软件下载中心](https://support.sap.com/swdc)获取 SAP Netweaver 库。 搜索“SAP Note #1025361”  获取最新版本的下载位置。 请确保选取与集成运行时安装匹配的 64 位 SAP NetWeaver 库  。 然后，按照 SAP 说明安装 SAP NetWeaver RFC SDK 中包含的所有文件。 SAP NetWeaver 库也包括在 SAP 客户端工具安装中。

>[!TIP]
>要解决 SAP BW 的连接问题，请确保：
>- 从 NetWeaver RFC SDK 中提取的所有依赖项库都位于 %windir%\system32 文件夹中。 通常它有 icudt34.dll、icuin34.dll、icuuc34.dll、libicudecnumber.dll、librfc32.dll、libsapucum.dll、sapcrypto.dll、sapnwrfc.dll。
>- 用于连接 SAP 服务器的所需端口在自承载 IR 计算机上启用，通常是端口 3300 和 3201。

## <a name="getting-started"></a>入门

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

对于特定于 SAP Business Warehouse 连接器的数据工厂实体，以下部分提供了有关用于定义这些实体的属性的详细信息。

## <a name="linked-service-properties"></a>链接服务属性

SAP Business Warehouse (BW) 链接服务支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | type 属性必须设置为：**SapBw** | 是 |
| server | SAP BW 实例所驻留的服务器的名称。 | 是 |
| systemNumber | SAP BW 系统的系统编号。<br/>允许值：用字符串表示的两位十进制数。 | 是 |
| clientId | SAP W 系统中的客户端的客户端 ID。<br/>允许值：用字符串表示的三位十进制数。 | 是 |
| userName | 有权访问 SAP 服务器的用户名。 | 是 |
| password | 用户密码。 将此字段标记为 SecureString 以安全地将其存储在数据工厂中或[引用存储在 Azure Key Vault 中的机密](store-credentials-in-key-vault.md)。 | 是 |
| connectVia | 用于连接到数据存储的[集成运行时](concepts-integration-runtime.md)。 如[先决条件](#prerequisites)中所述，需要自承载集成运行时。 |是 |

**示例：**

```json
{
    "name": "SapBwLinkedService",
    "properties": {
        "type": "SapBw",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client id>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
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

有关可用于定义数据集的各部分和属性的完整列表，请参阅数据集一文。 本部分提供 SAP BW 数据集支持的属性列表。

要从 SAP BW 复制数据，请将数据集的 type 属性设置为“RelationalTable”  。 RelationalTable 类型的 SAP BW 数据集不支持任何类型特定的属性。

**示例：**

```json
{
    "name": "SAPBWDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": {
            "referenceName": "<SAP BW linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供 SAP BW 源支持的属性列表。

### <a name="sap-bw-as-source"></a>SAP BW 作为源

要从 SAP BW 复制数据，请将复制活动中的源类型设置为“RelationalSource”  。 复制活动源  部分支持以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | 复制活动源的 type 属性必须设置为：**RelationalSource** | 是 |
| 查询 | 指定要从 SAP BW 实例读取数据的 MDX 查询。 | 是 |

**示例：**

```json
"activities":[
    {
        "name": "CopyFromSAPBW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP BW input dataset name>",
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
                "query": "<MDX query for SAP BW>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-sap-bw"></a>SAP BW 的数据类型映射

从 SAP BW 复制数据时，以下映射用于从 SAP BW 数据类型映射到 Azure 数据工厂临时数据类型。 若要了解复制活动如何将源架构和数据类型映射到接收器，请参阅[架构和数据类型映射](copy-activity-schema-and-type-mapping.md)。

| SAP BW 数据类型 | 数据工厂临时数据类型 |
|:--- |:--- |
| ACCP | Int |
| CHAR | String |
| CLNT | String |
| CURR | Decimal |
| CUKY | String |
| DEC | Decimal |
| FLTP | Double |
| INT1 | Byte |
| INT2 | Int16 |
| INT4 | Int |
| LANG | String |
| LCHR | String |
| LRAW | Byte[] |
| PREC | Int16 |
| QUAN | Decimal |
| RAW | Byte[] |
| RAWSTRING | Byte[] |
| STRING | String |
| UNIT | String |
| DATS | String |
| NUMC | String |
| TIMS | String |


## <a name="next-steps"></a>后续步骤
有关 Azure 数据工厂中复制活动支持作为源和接收器的数据存储的列表，请参阅[支持的数据存储](copy-activity-overview.md#supported-data-stores-and-formats)。
