---
title: 将块 Blob 存储在设备上 - Azure IoT Edge
description: 了解分层和生存时间功能、查看支持的 Blob 存储操作，以及连接到 Blob 存储帐户。
author: arduppal
manager: mchad
ms.author: arduppal
ms.reviewer: arduppal
origin.date: 06/19/2019
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: b9215f55796dc508d9d9c99aa03ecec6aba17337
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337312"
---
# <a name="store-data-at-the-edge-with-azure-blob-storage-on-iot-edge-preview"></a>通过 IoT Edge 上的 Azure Blob 存储（预览版）在边缘存储数据

IoT Edge 上的 Azure Blob 存储在边缘提供了[块 blob](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-block-blobs) 存储解决方案。 IoT Edge 设备上的 blob 存储模块的行为类似于 Azure 块 blob 服务，但块 blob 存储在本地 IoT Edge 设备上。 你可以使用相同的 Azure 存储 SDK 方法或已经习惯的块 blob API 调用来访问 blob。 

此模块附带 **deviceToCloudUpload** 和 **deviceAutoDelete** 功能。
> [!NOTE]
> IoT Edge 上的 Azure Blob 存储现为[公共预览版](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。


**deviceToCloudUpload** 是可配置的功能，用于将本地 Blob 存储中的数据自动上传到 Azure，并提供间歇性的 Internet 连接支持。 该功能允许：

- 启用/禁用 deviceToCloudUpload 功能。
- 选择将数据复制到 Azure 的顺序，例如，NewestFirst 或 OldestFirst。
- 指定要将数据上传到的 Azure 存储帐户。
- 指定要上传到 Azure 的容器。 此模块允许指定源和目标容器名称。
- 选择将内容上传到云存储后立即删除 Blob 的功能
- 执行完整 Blob 上传（使用 `Put Blob` 操作）和块级上传（使用 `Put Block` 和 `Put Block List` 操作）。

如果 Blob 由块构成，则此模块使用块级上传。 下面是一些常见场景：

- 应用程序需要更新以前上传的 Blob 的某些块；此模块只上传更新的块，而不上传整个 Blob。
- 当模块正在上传 Blob 时，Internet 连接断开；连接恢复后，该模块只上传剩余的块，而不上传整个 Blob。

如果在 Blob 上传期间发生意外的进程终止（例如电源故障），当模块重新联机时，将再次上传需要上传的所有块。

**deviceAutoDelete** 是可配置的功能，当本地存储中的 Blob 达到指定的持续时间（以分钟进行衡量）时，此模块会使用此功能自动删除这些 Blob。 该功能允许：

- 启用/禁用 deviceAutoDelete 功能。
- 指定以分钟为单位的时间 (deleteAfterMinutes)，该时间过后会自动删除这些 Blob。
- 选择在 deleteAfterMinutes 值到期后保留上传的 Blob 的功能。

对于需要在本地存储视频、图像、金融数据、医院数据或其他任何数据，以后在本地处理数据或将其传输到云的场合，都很适合使用此模块。

本文说明与 IoT Edge 容器中的 Azure Blob 存储相关的概念，该容器在 IoT Edge 设备上运行 Blob 服务。

## <a name="prerequisites"></a>先决条件

Azure IoT Edge 设备：

- 可以按照适用于 [Linux](quickstart-linux.md) 或 [Windows 设备](quickstart.md)的快速入门中的步骤，将开发计算机或虚拟机用作 IoT Edge 设备。

- IoT Edge 模块上的 Azure Blob 存储支持以下设备配置：

  | 操作系统 | AMD64 | ARM32v7 | ARM64 |
  | ---------------- | ----- | ----- | ---- |
  | Raspbian-stretch | 否 | 是 | 否 |  
  | Ubuntu Server 16.04 | 是 | 否 | 是（适用于使用 [Azure IoT Edge 1.0.8-rc1 及更高版本](https://github.com/Azure/azure-iotedge/releases)[安装](how-to-install-iot-edge-linux-arm.md#install-a-specific-version)） |
  | Ubuntu Server 18.04 | 是 | 否 | 是（适用于使用 [Azure IoT Edge 1.0.8-rc1 及更高版本](https://github.com/Azure/azure-iotedge/releases)[安装](how-to-install-iot-edge-linux-arm.md#install-a-specific-version)） |
  | Windows 10 IoT 企业版 17763 | 是 | 否 | 否 |
  | Windows Server 2019 内部版本 17763 | 是 | 否 | 否 |
  

云资源：

Azure 中的标准层 [IoT 中心](../iot-hub/iot-hub-create-through-portal.md)。

## <a name="devicetocloudupload-and-deviceautodelete-properties"></a>deviceToCloudUpload 和 deviceAutoDelete 属性

使用所需的属性设置 deviceToCloudUploadProperties 和 deviceAutoDeleteProperties。 可以在部署期间设置这些属性，或者，以后可以通过编辑模块孪生来更改这些属性，而无需重新部署。 我们建议检查“模块孪生”中的 `reported configuration` 和 `configurationValidation`，以确保正确传播值。

### <a name="devicetoclouduploadproperties"></a>deviceToCloudUploadProperties

此设置的名称为 `deviceToCloudUploadProperties`

| 字段 | 可能的值 | 说明 | 环境变量 |
| ----- | ----- | ---- | ---- |
| uploadOn | true、false | 默认设置为 `false`，若要启用它，可将它设置为 `true`| `deviceToCloudUploadProperties__uploadOn={false,true}` |
| uploadOrder | NewestFirst、OldestFirst | 用于选择将数据复制到 Azure 的顺序。 默认设置为 `OldestFirst`。 顺序由 Blob 的上次修改时间确定 | `deviceToCloudUploadProperties__uploadOrder={NewestFirst,OldestFirst}` |
| cloudStorageConnectionString |  | `"DefaultEndpointsProtocol=https;AccountName=<your Azure Storage Account Name>;AccountKey=<your Azure Storage Account Key>;EndpointSuffix=<your end point suffix>"` 是一个连接字符串，用于指定要将数据上传到的 Azure 存储帐户。 指定 `Azure Storage Account Name`、`Azure Storage Account Key` 或 `End point suffix`。 添加用于上传数据的适当 Azure EndpointSuffix，它在全局 Azure、政府 Azure 和 Microsoft Azure Stack 中是不同的。 | `deviceToCloudUploadProperties__cloudStorageConnectionString=<connection string>` |
| storageContainersForUpload | `"<source container name1>": {"target": "<target container name>"}`,<br><br> `"<source container name1>": {"target": "%h-%d-%m-%c"}`, <br><br> `"<source container name1>": {"target": "%d-%c"}` | 用于指定要上传到 Azure 的容器名称。 此模块允许指定源和目标容器名称。 如果未指定目标容器名称，系统会自动分配 `<IoTHubName>-<IotEdgeDeviceID>-<ModuleName>-<SourceContainerName>` 作为容器名称。 可以创建目标容器名称的模板字符串，具体请查看“可能的值”列。 <br>* %h -> IoT 中心名称（3 到 50 个字符）。 <br>* %d -> IoT Edge 设备 ID（1 到 129 个字符）。 <br>* %m -> 模块名称（1 到 64 个字符）。 <br>* %c -> 源容器名称（3 到 63 个字符）。 <br><br>容器名称的最大大小为 63 个字符。尽管系统会自动分配目标容器名称，但如果容器大小超过 63 个字符，系统会将每个部分（IoTHubName、IotEdgeDeviceID、ModuleName、SourceContainerName）修剪为 15 个字符。 | `deviceToCloudUploadProperties__storageContainersForUpload__<sourceName>__target: <targetName>` |
| deleteAfterUpload | true、false | 默认设置为 `false`。 将它设置为 `true` 时，它会在数据上传到云存储以后自动删除数据。 | `deviceToCloudUploadProperties__deleteAfterUpload={false,true}` |


### <a name="deviceautodeleteproperties"></a>deviceAutoDeleteProperties

此设置的名称为 `deviceAutoDeleteProperties`

| 字段 | 可能的值 | 说明 | 环境变量 |
| ----- | ----- | ---- | ---- |
| deleteOn | true、false | 默认设置为 `false`，若要启用它，可将它设置为 `true`| `deviceAutoDeleteProperties__deleteOn={false,true}` |
| deleteAfterMinutes | `<minutes>` | 以分钟为单位指定时间。 达到此值时，模块会自动删除本地存储中的 Blob | `deviceAutoDeleteProperties__ deleteAfterMinutes=<minutes>` |
| retainWhileUploading | true、false | 默认情况下，它设置为 `true`。在 deleteAfterMinutes 分钟过后，它会保留上传到云存储的 Blob。 可以将它设置为 `false`。在 deleteAfterMinutes 分钟过后，它会删除数据。 注意：要使此属性生效，应将 uploadOn 设置为 true| `deviceAutoDeleteProperties__retainWhileUploading={false,true}` |

## <a name="configure-log-files"></a>配置日志文件

若要了解如何为模块配置日志文件，请参阅这些[生产最佳做法](https://docs.microsoft.com/azure/iot-edge/production-checklist#set-up-logs-and-diagnostics)。

## <a name="connect-to-your-blob-storage-module"></a>连接到 blob 存储模块

可以使用为模块配置的帐户名和帐户密钥来访问 IoT Edge 设备上的 blob 存储。 

将你的 IoT Edge 设备指定为对其进行的任何存储请求的 blob 终结点。 你可以使用 IoT Edge 设备信息和配置的帐户名[为显式存储终结点创建连接字符串](../storage/common/storage-configure-connection-string.md#create-a-connection-string-for-an-explicit-storage-endpoint)。 

- 对于部署在设备上的模块，如果该设备上运行 IoT Edge 上的 Azure Blob 存储模块，则 Blob 终结点为 `http://<module name>:11002/<account name>`。
- 在设备上运行外部模块或应用程序时，如果该设备不运行 IoT Edge 上的 Azure Blob 存储模块，则可根据网络设置将 Blob 终结点设置为以下终结点之一，以便来自外部模块或应用程序的数据流可以到达运行 IoT Edge 上的 Azure Blob 存储模块的设备：
  - `http://<device IP >:11002/<account name>`
  - `http://<IoT Edge device hostname>:11002/<account name>`
  - `http://<fully qualified domain name>:11002/<account name>`

## <a name="azure-blob-storage-quickstart-samples"></a>Azure Blob 存储快速入门示例

Azure Blob 存储文档包括多种语言的快速入门示例代码。 可以通过将 Blob 终结点更改为连接到本地 Blob 存储模块来运行这些示例，以测试 IoT Edge 上的 Azure Blob 存储。

以下快速入门示例使用 IoT Edge 也同样支持的语言，因此，你可以将它们作为 IoT Edge 模块与 Blob 存储模块一起部署：

- [.NET](../storage/blobs/storage-quickstart-blobs-dotnet.md)
- [Java](../storage/blobs/storage-quickstart-blobs-java.md)
- [Python](../storage/blobs/storage-quickstart-blobs-python.md)
- [Node.js](../storage/blobs/storage-quickstart-blobs-nodejs.md)

## <a name="connect-to-your-local-storage-with-azure-storage-explorer"></a>通过 Azure 存储资源管理器连接到本地存储

可以使用 [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)连接到本地存储帐户。

1. 下载并安装 Azure 存储资源管理器

1. 使用连接字符串连接到 Azure 存储

1. 提供连接字符串：`DefaultEndpointsProtocol=http;BlobEndpoint=http://<host device name>:11002/<your local account name>;AccountName=<your local account name>;AccountKey=<your local account key>;`

1. 完成用于连接的步骤。

1. 在本地存储帐户中创建容器

1. 开始将文件作为块 Blob 上传。
   > [!NOTE]
   > 此模块不支持页 Blob。

1. 可以选择在上传数据的位置连接 Azure 存储帐户。 它提供适用于本地存储帐户和 Azure 存储帐户的单一视图。

## <a name="supported-storage-operations"></a>受支持的存储操作

IoT Edge 上的 Blob 存储模块使用相同的 Azure 存储 SDK，并与适用于块 Blob 终结点的 2017-04-17 版 Azure 存储 API 保持一致。 后期版本取决于客户需求。

由于并非所有 Azure Blob 存储操作都受 IoT Edge 上的 Azure Blob 存储支持，因此此部分列出了每项操作的状态。

### <a name="account"></a>帐户

受支持： 
* 列出容器

不受支持： 
* 获取和设置 blob 服务属性
* 预检 blob 请求
* 获取 blob 服务统计信息
* 获取帐户信息

### <a name="containers"></a>容器

受支持： 
* 创建和删除容器
* 获取容器属性和元数据
* 列出 Blob
* 获取和设置容器 ACL
* 设置容器元数据

不受支持：
* 租用容器

### <a name="blobs"></a>Blob

受支持： 
* 放置、获取和删除 blob
* 获取和设置 blob 属性
* 获取和设置 blob 元数据

不受支持： 
* 租用 blob
* 获取 blob 快照
* 复制和中止复制 blob
* 撤销删除 blob
* 设置 blob 层

### <a name="block-blobs"></a>块 Blob

受支持： 
* 放置块
* 放置和获取块列表

不受支持：
* 放置来自 URL 的块

## <a name="release-notes"></a>发行说明

这是此模块[在 Docker 中心的发行说明](https://hub.docker.com/_/microsoft-azure-blob-storage)

## <a name="feedback"></a>反馈

你的反馈对我们很重要，我们的目标是使此模块及其功能有用且易用。 请分享你的反馈并告诉我们该如何改进。

你可以通过 absiotfeedback@microsoft.com 联系我们

## <a name="next-steps"></a>后续步骤

详细了解如何[在 IoT Edge 上部署 Azure Blob 存储](how-to-deploy-blob.md)
