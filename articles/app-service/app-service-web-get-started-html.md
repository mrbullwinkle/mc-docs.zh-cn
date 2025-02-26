---
title: 创建静态 HTML Web 应用 - Azure 应用服务 | Azure Docs
description: 了解如何通过部署静态 HTML 示例应用，在 Azure 应用服务中运行 Web 应用。
services: app-service\web
documentationcenter: ''
author: msangapu
manager: jeconnoc
editor: ''
ms.assetid: 60495cc5-6963-4bf0-8174-52786d226c26
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
origin.date: 08/24/2018
ms.date: 02/25/2019
ms.author: v-biyu
ms.custom: seodec18
ms.openlocfilehash: f9bd5ac9597dd12d9eca07dfd0d0ea5c383e8993
ms.sourcegitcommit: d5e91077ff761220be2db327ceed115e958871c8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/13/2019
ms.locfileid: "56222586"
---
# <a name="create-a-static-html-web-app-in-azure"></a>在 Azure 中创建静态 HTML Web 应用

[Azure 应用服务](overview.md)提供高度可缩放、自修补的 Web 托管服务。  本快速入门演示如何将基本 HTML+CSS 站点部署到 Azure 应用服务。 可以使用 [Azure CLI](/cli/install-azure-cli) 在本地运行这些命令

![示例应用主页](media/app-service-web-get-started-html/hello-world-in-browser-az.png)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]


## <a name="download-the-sample"></a>下载示例

创建一个 quickstart 目录，然后切换到该目录。

```bash
mkdir quickstart

cd $HOME/quickstart
```

接下来请运行以下命令，将示例应用存储库克隆到快速入门目录。

```bash
git clone https://github.com/Azure-Samples/html-docs-hello-world.git
```

## <a name="create-a-web-app"></a>创建 Web 应用

切换到包含示例代码的目录并运行 `az webapp up` 命令。

在以下示例中，请将 <app_name> 替换为一个唯一的应用名称。

```bash
cd html-docs-hello-world

az webapp up --location chinaeast --name <app_name>
```

`az webapp up` 命令执行以下操作：

- 创建一个默认的资源组。

- 创建一个默认的应用服务计划。

- 创建一个采用指定名称的应用。

- [使用 Zip](/app-service/deploy-zip) 将文件从当前工作目录部署到 Web 应用。

此命令可能需要花费几分钟时间运行。 运行时，该命令会显示类似于以下示例的信息：

```json
{
  "app_url": "https://<app_name>.chinacloudsites.cn",
  "location": "China East",
  "name": "<app_name>",
  "os": "Windows",
  "resourcegroup": "appsvc_rg_Windows_chinaeast",
  "serverfarm": "appsvc_asp_Windows_chinaeast",
  "sku": "FREE",
  "src_path": "/home/<username>/quickstart/html-docs-hello-world ",
  < JSON data removed for brevity. >
}
```

记下 `resourceGroup` 值。 需要在[清理资源](#clean-up-resources)部分使用它。

## <a name="browse-to-the-app"></a>浏览到应用

在浏览器中转到应用 URL：`http://<app_name>.chinacloudsites.cn`。

该页作为 Azure 应用服务 Web 应用运行。

![示例应用主页](media/app-service-web-get-started-html/hello-world-in-browser-az.png)

**祝贺你！** 现已将第一个 HTML 应用部署到应用服务。

## <a name="update-and-redeploy-the-app"></a>更新并重新部署应用

在终端窗口中，键入 `nano index.html` 以打开 nano 文本编辑器。 在 `<h1>` 标题标记中，将“Azure 应用服务 - 示例静态 HTML 站点”更改为“Azure 应用服务”，如下所示。

![Nano index.html](media/app-service-web-get-started-html/nano-index-html.png)

保存更改并退出 nano。 使用命令 `^O` 来保存，使用 `^X` 来退出。

现在，需使用同一 `az webapp up` 命令重新部署应用。

```bash
az webapp up --location chinaeast --name <app_name>
```

完成部署后，切换回“**浏览到应用**”步骤中打开的浏览器窗口，然后刷新页面。

![已更新的示例应用主页](media/app-service-web-get-started-html/hello-azure-in-browser-az.png)

## <a name="manage-your-new-azure-app"></a>管理新的 Azure 应用

转到 <a href="https://portal.azure.cn" target="_blank">Azure 门户</a>管理创建的 Web 应用。

在左侧菜单中单击“应用程序服务”  ，然后单击 Azure 应用的名称。

![在门户中导航到 Azure 应用](./media/app-service-web-get-started-html/portal1.png)

随后会显示 Web 应用的概述页。 在此处可以执行基本的管理任务，例如浏览、停止、启动、重启和删除。

![Azure 门户中的“应用服务”边栏选项卡](./media/app-service-web-get-started-html/portal2.png)

左侧菜单提供用于配置应用的不同页面。

## <a name="clean-up-resources"></a>清理资源

在前面的步骤中，在资源组中创建了 Azure 资源。 如果认为将来不需要这些资源，请在 CLI 中运行以下命令以删除资源组。 请记住，资源组名称已在[创建 Web 应用](#create-a-web-app)步骤中自动为你生成。

```bash
az group delete --name appsvc_rg_Windows_chinaeast
```

此命令可能需要花费一分钟时间运行。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [映射自定义域](app-service-web-tutorial-custom-domain.md)
