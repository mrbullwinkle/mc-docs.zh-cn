---
title: include 文件
description: include 文件
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
origin.date: 06/05/2018
ms.date: 07/29/2019
ms.author: v-biyu
ms.custom: include file
ms.openlocfilehash: 13e6cc5920f74ad053c34f9dc590d4f2d76c3140
ms.sourcegitcommit: 5f260ee1d8ac487702b554a94cb971a3ee62a40b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2019
ms.locfileid: "68232269"
---
## <a name="prepare-your-repository"></a>准备存储库

若要从 Azure 应用服务 Kudu 生成服务器获取自动生成，请确保项目中存储库根路径具有正确的文件。

| 运行时 | 根目录文件 |
|-|-|
| ASP.NET（仅限 Windows） | _\*.sln_、 _\*.csproj_ 或 _default.aspx_ |
| ASP.NET Core | _\*.sln_ 或 _\*.csproj_ |
| PHP | index.php  |
| Ruby（仅限 Linux） | Gemfile  |
| Node.js | server.js、app.js 或具有启动脚本的 package.json    |
| Python | _\*.py_ 、 _requirements.txt_ 或 _runtime.txt_ |
| HTML | default.htm、default.html、default.asp、index.htm、index.html 或 iisstart.htm       |
| Web 作业 | App\_Data/jobs/continuous（适用于连续的 WebJobs）或 App\_Data/jobs/triggered（适用于触发的 WebJobs）下的 \<job_name>/run.\<extension>    。 有关详细信息，请参阅 [Kudu WebJobs 文档](https://github.com/projectkudu/kudu/wiki/WebJobs)。 |
| 函数 | 请参阅 [Azure Functions 的连续部署](https://docs.azure.cn/zh-cn/azure-functions/functions-continuous-deployment#continuous-deployment-requirements)。 |

要自定义部署，可以在存储库根路径中添加 .deployment 文件  。 有关详细信息，请参阅[自定义部署](https://github.com/projectkudu/kudu/wiki/Customizing-deployments)和[自定义部署脚本](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script)。

> [!NOTE]
> 如果在 Visual Studio 中进行开发，让 [Visual Studio 创建存储库](/azure/devops/repos/git/creatingrepo?view=vsts&tabs=visual-studio)。 该项目可立即通过 Git 进行部署。
>

