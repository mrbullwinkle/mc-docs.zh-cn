---
title: Docker 容器
titleSuffix: Language Understanding - Azure Cognitive Services
description: LUIS 容器可将已训练或已发布的应用加载到 docker 容器中并提供对容器的 API 终结点中的查询预测的访问权限。
services: cognitive-services
author: lingliw
manager: digimobile
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 07/02/2019
ms.author: v-lingwu
ms.openlocfilehash: f543f5dcd567d5a128c582c3668f1bd9dff61b3e
ms.sourcegitcommit: 68f7c41974143a8f7bd9b7a54acf41c09893e587
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68332108"
---
# <a name="install-and-run-luis-docker-containers"></a>安装并运行 LUIS docker 容器
 
语言理解 (LUIS) 容器可将已训练或已发布的语言理解模型（也称为 [LUIS 应用](https://luis.azure.cn)）加载到 docker 容器中并提供对容器的 API 终结点中的查询预测的访问权限。 可以从容器中收集查询日志并将这些日志上传回语言理解应用以提高应用的预测准确性。

以下视频演示如何使用此容器。

[![认知服务的容器演示](./media/luis-container-how-to/luis-containers-demo-video-still.png)](https://aka.ms/luis-container-demo)

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/zh-cn/pricing/1rmb-trial-full/?form-type=identityauth)。

## <a name="prerequisites"></a>先决条件

若要运行 LUIS 容器，必须具有以下各项： 

|必须|目的|
|--|--|
|Docker 引擎| 需要在[主计算机](#the-host-computer)上安装 Docker 引擎。 Docker 提供用于在 [macOS](https://docs.docker.com/docker-for-mac/)、[Windows](https://docs.docker.com/docker-for-windows/) 和 [Linux](https://docs.docker.com/engine/installation/#supported-platforms) 上配置 Docker 环境的包。 有关 Docker 和容器的基础知识，请参阅 [Docker 概述](https://docs.docker.com/engine/docker-overview/)。<br><br> 必须将 Docker 配置为允许容器连接 Azure 并向其发送账单数据。 <br><br>  在 Windows 上，还必须将 Docker 配置为支持 Linux 容器。<br><br>|
|熟悉 Docker | 应对 Docker 概念有基本的了解，例如注册表、存储库、容器和容器映像，以及基本的 `docker` 命令的知识。| 
|Azure `Cognitive Services` 资源和 LUIS [打包应用](luis-how-to-start-new-app.md#export-app-for-containers)文件 |若要使用容器，必须具有：<br><br>* 一项_认知服务_ Azure 资源，以及关联的计费密钥和计费终结点 URI。 这两个值都可以在资源的“概述”和“密钥”页上找到，并且是启动容器所必需的。 需将 `luis/v2.0` 路由添加到终结点 URI，如以下 BILLING_ENDPOINT_URI 示例所示。 <br>* 已训练或已发布的应用，作为已安装的输入打包到具有其关联的应用 ID 的容器。 可以通过 LUIS 门户或创作 API 获取打包文件。 若要通过[创作 API](#authoring-apis-for-package-file) 获得 LUIS 打包应用，还将需要_创作密钥_。<br><br>这些要求用于将命令行参数传递到以下变量：<br><br>**{AUTHORING_KEY}** ：此密钥用于从云中的 LUIS 服务获取打包的应用并将查询日志上传回云。 格式为 `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`。<br><br>**{APPLICATION_ID}** ：此 ID 用于选择应用。 格式为 `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`。<br><br>**{ENDPOINT_KEY}** ：此密钥用于启动容器。 可以在两个位置找到终结点密钥。 第一个是 Azure 门户的“认知服务”  资源的密钥列表。 也可以在 LUIS 门户的“密钥和终结点”设置页上找到终结点密钥。 请勿使用初学者密钥。<br><br>**{BILLING_ENDPOINT}** ：例如：`https://westus.api.cognitive.microsoft.com/luis/v2.0`。<br><br>[创作密钥和终结点密钥](luis-boundaries.md#key-limits)具有不同的用途。 请勿互换使用。 |

### <a name="authoring-apis-for-package-file"></a>用于包文件的创作 API

用于打包应用的创作 API：

* [已发布的包 API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagepublishedapplicationasgzip)
* [未发布、仅经过训练的包 API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagetrainedapplicationasgzip)

### <a name="the-host-computer"></a>主计算机

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>容器要求和建议

此容器支持设置的最小值和建议值：

|容器| 最小值 | 建议 | TPS<br>(最小值, 最大值)|
|-----------|---------|-------------|--|
|LUIS|单核，2-GB 内存|单核，4-GB 内存|20,40|

* 每个核心必须至少为 2.6 千兆赫 (GHz) 或更快。
* TPS - 每秒事务数

核心和内存对应于 `--cpus` 和 `--memory` 设置，用作 `docker run` 命令的一部分。



## <a name="how-to-use-the-container"></a>如何使用容器

一旦容器位于[主计算机](#the-host-computer)上，请通过以下过程使用容器。

![使用语言理解 (LUIS) 容器的过程](./media/luis-container-how-to/luis-flow-with-containers-diagram.jpg)

1. 通过 LUIS 门户或 LUIS API [导出容器的包](#export-packaged-app-from-luis)。
1. 将包文件移动到[主计算机](#the-host-computer)上的所需  输入目录中。 请勿重命名、更改、覆盖或解压缩 LUIS 包文件。
1. 使用所需的输入装入点  和计费设置[运行容器](##run-the-container-with-docker-run)。 提供 `docker run` 命令的多个[示例](luis-container-configuration.md#example-docker-run-commands)。 
1. [查询容器的预测终结点](#query-the-containers-prediction-endpoint)。 
1. 使用完此容器后，从 LUIS 门户的输出装入点[导入终结点日志](#import-the-endpoint-logs-for-active-learning)并[停止](#stop-the-container)容器。
1. 在“查看终结点话语”  页上使用 LUIS 门户的[主动学习](luis-how-to-review-endpoint-utterances.md)改进应用。

无法更改正在容器中运行的应用。 若要更改容器中的应用，必须使用 [LUIS](https://luis.azure.cn) 门户或使用 LUIS [创作 API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f) 更改 LUIS 服务中的应用。 然后进行训练和/或发布，下载新包并再次运行该容器。

容器内的 LUIS 应用无法导出回 LUIS 服务。 只能上传查询日志。 

## <a name="export-packaged-app-from-luis"></a>从 LUIS 导出打包的应用

LUIS 容器需要已训练或已发布的 LUIS 应用才能回复用户话语的预测查询。 若要获取 LUIS 应用，请使用已训练或已发布的包 API。 

默认位置是运行 `docker run` 命令的 `input` 子目录。  

将包文件置于某个目录中，然后在运行 docker 容器时引用此目录作为输入装入点。 

### <a name="package-types"></a>包类型

输入装入点目录可以同时包含应用的  生产、过渡  和已训练  版本。 所有包均已装载。 

|包类型|查询终结点 API|查询可用性|包文件名格式|
|--|--|--|--|
|训练结束|获取、发布|仅容器|`{APPLICATION_ID}_v{APPLICATION_VERSION}.gz`|
|过渡|获取、发布|Azure 和容器|`{APPLICATION_ID}_STAGING.gz`|
|生产|获取、发布|Azure 和容器|`{APPLICATION_ID}_PRODUCTION.gz`|

> [!IMPORTANT]
> 请勿重命名、更改、覆盖或解压缩 LUIS 包文件。

### <a name="packaging-prerequisites"></a>打包先决条件

在打包 LUIS 应用程序之前，必须满足以下条件：

|打包要求|详细信息|
|--|--|
|Azure  认知服务资源实例|支持的区域包括<br><br>中国东部 (```chinaeast```)|
|已训练或已发布的 LUIS 应用|没有[不受支持的依赖项](#unsupported-dependencies)。 |
|访问[主计算机](#the-host-computer)的文件系统 |主计算机必须允许[输入装入点](luis-container-configuration.md#mount-settings)。|
  
### <a name="export-app-package-from-luis-portal"></a>通过 LUIS 门户导出应用包

LUIS [门户](https://luis.azure.cn)提供导出已训练或已发布的应用包的功能。 

### <a name="export-published-apps-package-from-luis-portal"></a>通过 LUIS 门户导出已发布的应用包

可从  “我的应用”列表页中获取已发布的应用包。 

1. 登录到 LUIS [门户](https://luis.azure.cn)。
1. 选中列表中应用名称左侧的复选框。 
1. 从列表上方的上下文工具栏中选择  “导出”项。
1. 选择“导出容器 (GZIP)”  。
1. 选择“生产槽”或“过渡槽”的环境   。
1. 将从浏览器下载包。

![从“应用”页面的“导出”菜单导出容器的已发布包](./media/luis-container-how-to/export-published-package-for-container.png)

### <a name="export-trained-apps-package-from-luis-portal"></a>通过 LUIS 门户导出已训练的应用包

可从  “版本”列表页中获取已训练的应用包。 

1. 登录到 LUIS [门户](https://luis.azure.cn)。
1. 在列表中选择应用。 
1. 在应用的导航栏中选择“管理”  。
1. 在左侧导航栏中选择“版本”  。
1. 选中列表中版本名称左侧的复选框。
1. 从列表上方的上下文工具栏中选择  “导出”项。
1. 选择“导出容器 (GZIP)”  。
1. 将从浏览器下载包。

![从“版本”页面的“导出”菜单导出容器的已训练包](./media/luis-container-how-to/export-trained-package-for-container.png)


### <a name="export-published-apps-package-from-api"></a>通过 API 导出已发布的应用包

使用以下 REST API 方法打包[已发布](luis-how-to-publish-app.md)的 LUIS 应用。 使用 HTTP 规范下方的表将 API 调用中的占位符替换为自己相应的值。

```http
GET /luis/api/v2.0/package/{APPLICATION_ID}/slot/{APPLICATION_ENVIRONMENT}/gzip HTTP/1.1
Host: {AZURE_REGION}.api.cognitive.microsoft.com
Ocp-Apim-Subscription-Key: {AUTHORING_KEY}
```

| 占位符 | Value |
|-------------|-------|
|{APPLICATION_ID} | 已发布 LUIS 应用的应用程序 ID。 |
|{APPLICATION_ENVIRONMENT} | 已发布 LUIS 应用的环境。 使用以下值之一：<br/>```PRODUCTION```<br/>```STAGING``` |
|{AUTHORING_KEY} | 已发布 LUIS 应用的 LUIS 帐户的创作密钥。<br/>可以从 LUIS 门户的“用户设置”  页面中获取创作密钥。 |
|{AZURE_REGION} | 相应的 Azure 区域：<br/><br/>```westus``` - 美国西部<br/>```westeurope``` - 西欧<br/>```australiaeast``` - 澳大利亚东部 |

若要下载已发布的包，请参考[此处的 API 文档][download-published-package]。 如果下载成功，响应是一个 LUIS 包文件。 将文件保存在为容器的输入装入点指定的存储位置中。 

### <a name="export-trained-apps-package-from-api"></a>通过 API 导出已训练的应用包

使用以下 REST API 方法打包[已训练](luis-how-to-train.md)的 LUIS 应用程序。 使用 HTTP 规范下方的表将 API 调用中的占位符替换为自己相应的值。

```http
GET /luis/api/v2.0/package/{APPLICATION_ID}/versions/{APPLICATION_VERSION}/gzip HTTP/1.1
Host: {AZURE_REGION}.api.cognitive.microsoft.com
Ocp-Apim-Subscription-Key: {AUTHORING_KEY}
```

| 占位符 | Value |
|-------------|-------|
|{APPLICATION_ID} | 已训练 LUIS 应用程序的应用程序 ID。 |
|{APPLICATION_VERSION} | 已训练 LUIS 应用程序的应用程序版本。 |
|{AUTHORING_KEY} | 已发布 LUIS 应用的 LUIS 帐户的创作密钥。<br/>可以从 LUIS 门户的“用户设置”  页面中获取创作密钥。  |
|{AZURE_REGION} | 相应的 Azure 区域：<br/><br/>```westus``` - 美国西部<br/>```westeurope``` - 西欧<br/>```australiaeast``` - 澳大利亚东部 |

若要下载经过训练的包，请参考[此处的 API 文档][download-trained-package]。 如果下载成功，响应是一个 LUIS 包文件。 将文件保存在为容器的输入装入点指定的存储位置中。 

## <a name="run-the-container-with-docker-run"></a>通过 `docker run` 运行容器

使用 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 命令运行容器。 该命令使用以下参数：

| 占位符 | Value |
|-------------|-------|
|{ENDPOINT_KEY} | 此密钥用于启动容器。 请勿使用初学者密钥。 |
|{BILLING_ENDPOINT} | Azure 门户的“`Cognitive Services` 概述”页上提供了计费终结点值。 需将 `luis/v2.0` 路由添加到终结点 URI，如以下示例所示：`https://westus.api.cognitive.microsoft.com/luis/v2.0`。|

在以下示例 `docker run` 命令中，请将这些参数替换为自己的值。 在 Windows 控制台中运行该命令。

```console
docker run --rm -it -p 5000:5000 ^
--memory 4g ^
--cpus 2 ^
--mount type=bind,src=c:\input,target=/input ^
--mount type=bind,src=c:\output\,target=/output ^
mcr.microsoft.com/azure-cognitive-services/luis ^
Eula=accept ^
Billing={BILLING_ENDPOINT} ^
ApiKey={ENDPOINT_KEY}
```

* 此示例使用 `C:` 驱动器外的目录来避免 Windows 上的任何权限冲突。 如果需要使用特定目录作为输入目录，则需要授予 docker 服务权限。 
* 除非非常熟悉 docker 容器，否则不要更改参数顺序。
* 如果使用的是不同的操作系统，请使用正确的控制台/终端、用于装载的文件夹语法和系统的行继续符。 这些示例假定 Windows 控制台使用行继续符 `^`。 由于容器是 Linux 操作系统，因此目标装载使用 Linux 样式的文件夹语法。

此命令：

* 从 LUIS 容器映像运行容器
* 从位于容器主机上的 c:\input 中的输入装入点加载 LUIS 应用
* 分配两个 CPU 内核和 4 千兆字节 (GB) 的内存
* 公开 TCP 端口 5000，并为容器分配伪 TTY
* 将容器和 LUIS 日志保存到位于容器主机上的 c:\output 中的输出装入点
* 退出后自动删除容器。 容器映像在主计算机上仍然可用。 

提供 `docker run` 命令的多个[示例](luis-container-configuration.md#example-docker-run-commands)。 

> [!IMPORTANT]
> 必须指定 `Eula`、`Billing` 和 `ApiKey` 选项运行容器；否则，该容器不会启动。  有关详细信息，请参阅[计费](#billing)。
> ApiKey 值是 LUIS 门户中“密钥和终结点”页面中的“密钥”  ，也可以在 Azure `Cognitive Services`资源密钥页上找到。  

[!INCLUDE [Running multiple containers on the same host](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

## <a name="endpoint-apis-supported-by-the-container"></a>容器支持的终结点 API

容器同时提供了 V2 和 [V3（预览）](luis-migration-api-v3.md)版 API。 

## <a name="query-the-containers-prediction-endpoint"></a>查询容器的预测终结点

容器提供了基于 REST 的查询预测终结点 API。 已发布（过渡或生产）应用的终结点包含的路由与已训练应用的终结点不同  。 

使用主机 `https://localhost:5000`，以获得容器 API。 

|包类型|方法|路由|查询参数|
|--|--|--|--|
|已发布|[获取](https://westus.dev.cognitive.microsoft.com/docs/services/5819c76f40a6350ce09de1ac/operations/5819c77140a63516d81aee78)、[发布](https://westus.dev.cognitive.microsoft.com/docs/services/5819c76f40a6350ce09de1ac/operations/5819c77140a63516d81aee79)|/luis/v2.0/apps/{appId}?|q={q}<br>&staging<br>[&timezoneOffset]<br>[&verbose]<br>[&log]<br>|
|训练结束|获取、发布|/luis/v2.0/apps/{appId}/versions/{versionId}?|q={q}<br>[&timezoneOffset]<br>[&verbose]<br>[&log]|

查询参数配置查询响应的返回方式以及返回内容：

|查询参数|类型|目的|
|--|--|--|
|`q`|string|用户的话语。|
|`timezoneOffset`|number|通过 timezoneOffset 可以[更改时区](luis-concept-data-alteration.md#change-time-zone-of-prebuilt-datetimev2-entity)，该时区由预生成实体 datetimeV2 使用。|
|`verbose`|布尔值|设置为 true 时，返回所有意向及其分数。 默认值为 false 时，仅返回评分最高的意向。|
|`staging`|布尔值|设置为 true 时，返回过渡环境结果中的查询。 |
|`log`|布尔值|记录查询，以便以后用于[主动学习](luis-how-to-review-endpoint-utterances.md)。 默认值为 true。|

### <a name="query-published-app"></a>查询已发布的应用

用于查询已发布应用的容器的示例 CURL 命令是：

```bash
curl -X GET \
"http://localhost:5000/luis/v2.0/apps/{APPLICATION_ID}?q=turn%20on%20the%20lights&staging=false&timezoneOffset=0&verbose=false&log=true" \
-H "accept: application/json"
```
若要对过渡  环境执行查询，请将过渡  查询字符串参数值更改为 true： 

`staging=true`

### <a name="query-trained-app"></a>查询已训练的应用

用于查询已训练应用的容器的示例 CURL 命令是： 

```bash
curl -X GET \
"http://localhost:5000/luis/v2.0/apps/{APPLICATION_ID}/versions/{APPLICATION_VERSION}?q=turn%20on%20the%20lights&timezoneOffset=0&verbose=false&log=true" \
-H "accept: application/json"
```
版本名称最多包含 10 个字符，并且仅包含 URL 中允许的字符。 

## <a name="import-the-endpoint-logs-for-active-learning"></a>导入终结点日志以供主动学习

如果为 LUIS 容器指定了输出装入点，则应用查询日志文件将保存在输出目录中，其中 {INSTANCE_ID} 是容器 ID。 应用查询日志包含提交到 LUIS 容器的每个预测查询的查询、响应和时间戳。 

以下位置显示了容器的日志文件的嵌套目录结构。
```
/output/luis/{INSTANCE_ID}/
```
 
从 LUIS 门户中选择应用，然后选择“导入终结点日志”  以上传这些日志。 

![导入容器的日志文件以供主动学习](./media/luis-container-how-to/upload-endpoint-log-files.png)

上传日志后，在 LUIS 门户中[查看终结点](https://docs.azure.cn/cognitive-services/LUIS/luis-concept-review-endpoint-utterances)话语。


<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>停止容器

若要关闭容器，请在运行容器的命令行环境中按 Ctrl+C  。

## <a name="troubleshooting"></a>故障排除

如果运行启用了输出[装入点](luis-container-configuration.md#mount-settings)和日志记录的容器，该容器会生成有助于排查启动或运行容器时发生的问题的日志文件。 

## <a name="billing"></a>计费

LUIS 容器使用 Azure 帐户中的认知服务  资源向 Azure 发送账单信息。 

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

有关这些选项的详细信息，请参阅[配置容器](luis-container-configuration.md)。

## <a name="supported-dependencies-for-latest-container"></a>支持的 `latest` 容器依赖项

在 2019 //Build 上发布的最新容器将支持：

* 必应拼写检查：使用 `&spellCheck=true&bing-spell-check-subscription-key={bingKey}` 查询字符串参数向查询预测终结点发出的请求。 若要了解详细信息，请参阅[必应拼写检查 v7 教程](luis-tutorial-bing-spellcheck.md)。 如果使用此功能，则容器会将话语发送到必应拼写检查 V7 资源。
* [新的预生成的域](luis-reference-prebuilt-domains.md)：这些聚焦于企业的域包括实体、示例话语以及模式。 扩展这些供自己使用的域。 

<a name="unsupported-dependencies"></a>

## <a name="unsupported-dependencies-for-latest-container"></a>不支持的 `latest` 容器依赖项

如果 LUIS 应用有不受支持的依赖项，则在删除这些不受支持的功能之前，无法[针对容器进行导出](#export-packaged-app-from-luis)。 尝试针对容器进行导出时，LUIS 门户会报告需删除的不受支持的功能。

如果 LUIS 应用程序不包括  以下任何依赖项，则可以使用该应用程序：

不受支持的应用配置|详细信息|
|--|--|
|不受支持的容器区域性| 荷兰语 (nl-NL)<br>日语 (ja-JP)<br>仅 [1.0.2 tokenizer](luis-language-support.md) 支持德语。|
|所有区域性不支持的实体|所有区域性的 [KeyPhrase](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-keyphrase) 预生成实体|
|英语 (EN-US) 区域性不支持的实体|[GeographyV2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-geographyv2) 预生成实体|
|语音启动|容器中不支持外部依赖项。|
|情绪分析|容器中不支持外部依赖项。|

<!--blogs/samples/video courses -->
[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>摘要

在本文中，我们已学习相关的概念，以及语言理解 (LUIS) 容器的下载、安装和运行工作流。 综上所述：

* 语言理解 (LUIS) 为 Docker 提供一个 Linux 容器，用于提供话语的终结点查询预测。
* 从 Azure 容器注册表 (MCR) 下载容器映像。
* 容器映像在 Docker 中运行。
* 可以使用 REST API 来通过指定容器的主机 URI 查询容器终结点。
* 必须在实例化容器时指定账单信息。

> [!IMPORTANT]
> 如果未连接到 Azure 进行计量，则无法授权并运行认知服务容器。 客户需要始终让容器向计量服务传送账单信息。 认知服务容器不会将客户数据（例如，正在分析的图像或文本）发送给 Microsoft。

## <a name="next-steps"></a>后续步骤

* 查看[配置容器](luis-container-configuration.md)了解配置设置
* 若要解决与 LUIS 功能相关的问题，请参阅[故障排除](troubleshooting.md)。
* 使用更多[认知服务容器](../cognitive-services-container-support.md)




