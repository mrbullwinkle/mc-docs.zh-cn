---
title: Azure Functions 中的部署技术 | Microsoft Docs
description: 了解将代码部署到 Azure Functions 的不同方式。
services: functions
documentationcenter: .net
author: ColbyTresness
manager: dariac
ms.service: azure-functions
ms.custom: vs-azure
ms.topic: conceptual
origin.date: 04/25/2019
ms.date: 07/17/2019
ms.author: v-junlch
ms.openlocfilehash: 5811dac2c9e16c610b2573581bb46ccbf2bb0e04
ms.sourcegitcommit: c61b10764d533c32d56bcfcb4286ed0fb2bdbfea
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68331957"
---
# <a name="deployment-technologies-in-azure-functions"></a>Azure Functions 中的部署技术

可以使用多种不同的技术将 Azure Functions 项目代码部署到 Azure。 本文提供这些技术的详尽列表，介绍哪种技术适用于哪种 Functions 风格，解释使用每种方法时会发生什么情况，并提供有关在各种场合下使用的最佳方法的建议。 为部署到 Azure Functions 提供支持的各种工具已根据其环境和适当的技术进行优化。

## <a name="deployment-technology-availability"></a>部署技术的可用性

> [!IMPORTANT]
> Azure Functions 支持跨平台的本地开发以及使用 Windows 和 Linux 作为托管位置。 目前，可以使用三种托管计划：[消耗](functions-scale.md#consumption-plan)、高级和[专用（Azure 应用服务）](functions-scale.md#app-service-plan)。 每种计划有不同的行为。 并非所有部署技术都适用于每种风格的 Azure Functions。

| 部署技术 | Windows 消耗计划 | Windows 高级计划（预览版） | Windows 专用计划  | Linux 消耗计划（预览版） | Linux 专用计划 |
|-----------------------|:-------------------:|:-------------------------:|:-----------------:|:---------------------------:|:---------------:|
| 外部包 URL<sup>1</sup> |✔|✔|✔|✔|✔|
| 压缩部署 |✔|✔|✔| |✔|
| Docker 容器 | | | | |✔|
| Web 部署 |✔|✔|✔| | |
| 源代码管理 |✔|✔|✔| |✔|
| 本地 Git<sup>1</sup> |✔|✔|✔| |✔|
| 云同步<sup>1</sup> |✔|✔|✔| |✔|
| FTP<sup>1</sup> |✔|✔|✔| |✔|
| 门户编辑 |✔|✔|✔| |✔<sup>2</sup>|

<sup>1</sup> 需要[手动触发器同步](#trigger-syncing)的部署技术。  
<sup>2</sup> 仅对 Linux 上使用专用计划的 Functions 的 HTTP 和计时器触发器启用门户编辑。

## <a name="key-concepts"></a>关键概念

若要了解 Azure Functions 中的部署工作原理，必须先掌握一些关键概念。

### <a name="trigger-syncing"></a>触发器同步

更改任何触发器时，Functions 基础结构必须意识到这些更改。 对于许多部署技术而言，同步会自动进行。 但在某些情况下，必须手动同步触发器。 通过引用外部包 URL、本地 Git、云同步或 FTP 部署更新时，必须手动同步触发器。 可通过以下三种方式之一来同步触发器：

* 在 Azure 门户中重启函数应用
* 使用[主密钥](functions-bindings-http-webhook.md#authorization-keys)将 HTTP POST 请求发送到 `https://{functionappname}.chinacloudsites.cn/admin/host/synctriggers?code=<API_KEY>`。
* 将 HTTP POST 请求发送到 `https://management.chinacloudapi.cn/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP_NAME>/providers/Microsoft.Web/sites/<FUNCTION_APP_NAME>/syncfunctiontriggers?api-version=2016-08-01`。 请将占位符替换为你的订阅 ID、资源组名称和函数应用名称。

## <a name="deployment-technology-details"></a>部署技术详细信息 

Azure Functions 中提供了以下部署方法。

### <a name="external-package-url"></a>外部包 URL

可以使用外部包 URL 来引用包含函数应用的远程包 (.zip) 文件。 可从提供的 URL 下载该文件，应用将在[“从包运行”](run-functions-from-deployment-package.md)模式下运行。

>__如何使用：__ 将 `WEBSITE_RUN_FROM_PACKAGE` 添加到应用程序设置。 此设置的值应是一个 URL（要运行的特定包文件的位置）。 可以[在门户中](functions-how-to-use-azure-function-app-settings.md#settings)或[使用 Azure CLI](/cli/functionapp/config/appsettings#az-functionapp-config-appsettings-set) 来添加设置。 
>
>如果使用 Azure Blob 存储，请结合[共享访问签名 (SAS)](../vs-azure-tools-storage-manage-with-storage-explorer.md#generate-a-sas-in-storage-explorer) 使用专用容器，使 Functions 能够访问该包。 每当应用程序重启时，都会提取内容的副本。 引用必须在应用程序的整个生存期内有效。

>__何时使用：__ 对于消耗计划（预览版）中的、在 Linux 上运行的 Azure Functions，外部包 URL 是唯一受支持的部署方法。 更新函数应用引用的包文件时，必须[手动同步触发器](#trigger-syncing)，以告知 Azure 你的应用程序已更改。

### <a name="zip-deploy"></a>压缩部署

使用压缩部署可将包含函数应用的 .zip 文件推送到 Azure。 （可选）可将应用设置为在[“从包运行”](run-functions-from-deployment-package.md)模式下启动。

>__如何使用：__ 使用偏爱的客户端工具进行部署：[VS Code](functions-create-first-function-vs-code.md#publish-the-project-to-azure)、[Visual Studio](functions-develop-vs.md#publish-to-azure)或 [Azure CLI](functions-create-first-azure-function-azure-cli.md#deploy-the-function-app-project-to-azure)。 若要手动将 .zip 文件部署到函数应用，请遵照[从 .zip 文件或 URL 进行部署](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url)中的说明操作。
>
>使用压缩部署方法时，可将应用设置为在[“从包运行”](run-functions-from-deployment-package.md)模式下运行。 若要设置“从包运行”模式，请将 `WEBSITE_RUN_FROM_PACKAGE` 应用程序设置值设置为 `1`。 我们建议使用压缩部署。 此方法可以缩短应用程序加载时间，并且是 VS Code、Visual Studio 和 Azure CLI 的默认部署方法。

>__何时使用：__ 对于 Windows 上运行的 Azure Functions，以及专门计划中的、在 Linux 上运行的 Azure Functions，我们建议使用压缩部署。

### <a name="docker-container"></a>Docker 容器

可以部署包含函数应用的 Linux 容器映像。

>__如何使用：__ 在专用计划中创建一个 Linux 函数应用，并指定要从哪个容器映像运行。 可通过以下方式实现此目的：
>
>* 在 Azure 门户上的 Azure 应用服务计划中创建一个 Linux 函数应用。 对于“发布”，请选择“Docker 映像”，然后配置容器。   输入映像所在的位置。
>
>
>若要使用自定义容器部署到现有应用，请在 [Azure Functions Core Tools](functions-run-local.md) 中使用 [`func deploy`](functions-run-local.md#publish) 命令。

>__何时使用：__ 需要更好地控制运行函数应用的 Linux 环境时，请使用 Docker 容器选项。 此部署机制仅适用于应用服务计划中的、在 Linux 上运行的 Functions。

### <a name="web-deploy-msdeploy"></a>Web 部署 (MSDeploy)

Web 部署可打包 Windows 应用程序（包括 Azure 中的 Windows 上运行的函数应用）并将其部署到任何 IIS 服务器。

>__如何使用：__ 使用 [Visual Studio Tools for Azure Functions](functions-create-your-first-function-visual-studio.md)。 清除“从包文件运行(建议)”复选框。 
>
>还可以下载 [Web 部署 3.6](https://www.iis.net/downloads/microsoft/web-deploy) 并直接调用 `MSDeploy.exe`。

>__何时使用：__ 可以顺利地支持 Web 部署，但是，首选机制是[在启用“从包运行”的情况下使用压缩部署](#zip-deploy)。 有关详细信息，请参阅 [Visual Studio开发指南](functions-develop-vs.md#publish-to-azure)。

### <a name="source-control"></a>源代码管理

使用源代码管理将函数应用连接到 Git 存储库。 更新该存储库中的代码会触发部署。 有关详细信息，请参阅 [Kudu Wiki](https://github.com/projectkudu/kudu/wiki/VSTS-vs-Kudu-deployments)。

>__如何使用：__ 使用 Azure Functions 门户中的部署中心设置从源代码管理进行发布。 

>__何时使用：__ 对于协作开发函数应用的团队而言，最佳做法是使用源代码管理。 源代码管理是一种很好的部署选项，可实现更复杂的部署管道。

### <a name="local-git"></a>本地 Git

可以使用本地 Git 将代码从本地计算机推送到 Azure Functions。

>__如何使用：__ 请遵照[使用本地 Git 部署到 Azure 应用服务](../app-service/deploy-local-git.md)中的说明。

>__何时使用：__ 一般而言，我们建议使用其他部署方法。 从本地 Git 发布时，必须[手动同步触发器](#trigger-syncing)。

### <a name="cloud-sync"></a>云同步

使用云同步可将 Dropbox 和 OneDrive 中的内容同步到 Azure Functions。

>__何时使用：__ 一般而言，我们建议使用其他部署方法。 使用云同步发布时，必须[手动同步触发器](#trigger-syncing)。

### <a name="ftp"></a>FTP

可以使用 FTP 将文件直接传输到 Azure Functions。

>__如何使用：__ 请遵照[使用 FTP/s 部署内容](../app-service/deploy-ftp.md)中的说明。

>__何时使用：__ 一般而言，我们建议使用其他部署方法。 使用 FTP 发布时，必须[手动同步触发器](#trigger-syncing)。

### <a name="portal-editing"></a>门户编辑

在基于门户的编辑器中，可以直接编辑函数应用中的文件（基本上每次保存更改都要进行部署）。

>__如何使用：__ 若要在 Azure 门户中编辑函数，必须事先[在门户中创建函数](functions-create-first-azure-function.md)。 若要保留单一事实源，使用任何其他部署方法会使函数变为只读，并会阻止在门户中继续编辑。 若要恢复到可在 Azure 门户中编辑文件的状态，可以手动将编辑模式改回 `Read/Write`，并删除与部署相关的任何应用程序设置（例如 `WEBSITE_RUN_FROM_PACKAGE`）。 

>__何时使用：__ 在门户中可以十分方便地开始使用 Azure Functions。 对于更密集的开发工作，我们建议使用客户端工具：
>
>* [开始使用 VS Code](functions-create-first-function-vs-code.md)
>* [开始使用 Azure Functions Core Tools](functions-run-local.md)
>* [开始使用 Visual Studio](functions-create-your-first-function-visual-studio.md)

下表显示了支持门户编辑的操作系统和语言：

| | Windows 消耗计划 | Windows 高级计划（预览版） | Windows 专用计划 | Linux 消耗计划（预览版） | Linux 专用计划 |
|-|:-----------------: |:-------------------------:|:-----------------:|:---------------------------:|:---------------:|
| C# | | | | | |
| C# 脚本 |✔|✔|✔| |✔<sup>*</sup>|
| F# | | | | | |
| Java | | | | | |
| JavaScript (Node.js) |✔|✔|✔| |✔<sup>*</sup>|
| Python（预览版） | | | | | |
| PowerShell（预览版） |✔|✔|✔| | |
| TypeScript (Node.js) | | | | | |

<sup>*</sup> 仅对 Linux 上使用专用计划的 Functions 的 HTTP 和计时器触发器启用门户编辑。

## <a name="deployment-slots"></a>部署槽

将函数应用部署到 Azure 时，可以部署到单独的部署槽，而不是直接部署到生产槽。 有关部署槽的详细信息，请参阅 [Azure 应用服务槽](../app-service/deploy-staging-slots.md)。

### <a name="deployment-slots-levels-of-support"></a>部署槽的支持级别

部署槽有两个支持级别：

* **正式版 (GA)** ：完全受支持，并已获批在生产环境中使用。
* **预览**：尚不支持，但将来有望达到正式版的支持状态。

| OS/托管计划 | 支持级别 |
| --------------- | ------ |
| Windows 消耗计划 | 预览 |
| Windows 专用计划 | 正式版 |
| Linux 消耗计划 | 不支持 |
| Linux 专用计划 | 正式版 |

## <a name="next-steps"></a>后续步骤

请阅读以下文章详细了解如何部署函数应用： 

+ [Azure Functions 的压缩部署](deployment-zip-push.md)
+ [从包文件运行 Azure Functions](run-functions-from-deployment-package.md)
+ [为 Azure Functions 中的函数应用自动执行资源部署](functions-infrastructure-as-code.md)

