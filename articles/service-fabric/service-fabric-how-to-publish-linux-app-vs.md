---
title: 了解如何创建 .NET Core 应用程序并将其发布到远程 Azure Service Fabric Linux 群集 | Azure
description: 在 Visual Studio 中创建 .NET Core 应用并将其发布到远程 Linux 群集
services: service-fabric
documentationcenter: .net
author: rockboyfor
manager: digimobile
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: troubleshooting
ms.tgt_pltfrm: NA
ms.workload: NA
origin.date: 05/20/2019
ms.date: 07/08/2019
ms.author: v-yeche
ms.openlocfilehash: fc9b040296e5b947397a4cbab8fee7e582d5e802
ms.sourcegitcommit: 8f49da0084910bc97e4590fc1a8fe48dd4028e34
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67844984"
---
<!--Verify successfully-->
<!--Need Reopen the Visual Studio project and refresh certification after about half an hour-->
# <a name="use-visual-studio-to-create-and-and-publish-net-core-applications-targeting-a-remote-linux-service-fabric-cluster"></a>使用 Visual Studio 创建 .NET Core 应用程序并将其发布到远程 Linux Service Fabric 群集
可以使用 Visual Studio 工具开发 .NET Core 应用程序并将其发布到 Linux Service Fabric 群集。 SDK 版本必须为 3.4 或更高才能在 Visual Studio 中将 .NET Core 应用程序部署到 Linux Service Fabric 群集。

> [!Note]
> Visual Studio 不支持调试面向 Linux 的 Service Fabric 应用程序。
>

## <a name="create-a-service-fabric-application-targeting-net-core"></a>创建面向 .NET Core 的 Service Fabric 应用程序
1. 以**管理员**身份启动 Visual Studio。
2. 通过单击“文件”->“新建”->“项目”来创建项目。 
3. 在“新建项目”对话框中，选择“云”->“Service Fabric 应用程序”。  
    ![create-application]
4. 命名应用程序，并单击“确定”  。
5. 在“新建 Service Fabric 服务”页中，选择要在“.NET Core 部分”创建的服务类型。  
    ![create-service]

## <a name="deploy-to-a-remote-linux-cluster"></a>部署到远程 Linux 群集
1. 在“解决方案资源管理器”中，右键单击该应用程序并选择“生成”。 

    ![build-application]
2. 应用程序的生成过程完成后，右键单击服务，然后选择编辑 **csproj 文件**。

    ![edit-csproj]
3. 将 UpdateServiceFabricManifestEnabled 属性从 True 编辑为 **False**，前提是服务为**执行组件项目类型**。 如果应用程序没有执行组件服务，请跳到步骤 4。
    
    ```xml
    <UpdateServiceFabricManifestEnabled>False</UpdateServiceFabricManifestEnabled>
    ```
    > [!Note]
    > 将 UpdateServiceFabricManifestEnabled 设置为 false 会禁止在生成过程中更新 ServiceManifest.xml。 任何更改（例如添加、删除或重命名此服务）都不会反映在 ServiceManifest.xml 中。 如果进行了任何更改，则必须手动更新 ServiceManifest manually，或者将 UpdateServiceFabricManifestEnabled 临时设置为 true 并生成一个服务来更新 ServiceManifest.xml，然后将其恢复为 false。
    >

4. 在服务项目中将 RuntimeIndetifier 从 win7-x64 更新为目标平台。
    
    ```xml
    <RuntimeIdentifier>ubuntu.16.04-x64</RuntimeIdentifier>
    ```
5. 在 ServiceManifest 中更新入口点程序，删除 .exe。 
    
    ```xml
    <EntryPoint> 
    <ExeHost> 
        <Program>Actor1</Program> 
    </ExeHost> 
    </EntryPoint>
    ```
6. 在“解决方案资源管理器”中，右键单击该应用程序并选择“发布”。  此时会显示“发布”对话框。 
7. 在“连接终结点”中，选择要将其作为目标的远程 Service Fabric Linux 群集的终结点  。
    
    ![publish-application]

<!--Image references-->
[create-application]:./media/service-fabric-how-to-vs-remote-linux-cluster/create-application-remote-linux.png
[create-service]:./media/service-fabric-how-to-vs-remote-linux-cluster/create-service-remote-linux.png
[build-application]:./media/service-fabric-how-to-vs-remote-linux-cluster/build-application-remote-linux.png
[edit-csproj]:./media/service-fabric-how-to-vs-remote-linux-cluster/edit-csproj-remote-linux.png
[publish-application]:./media/service-fabric-how-to-vs-remote-linux-cluster/publish-remote-linux.png

## <a name="next-steps"></a>后续步骤
* 了解 [Service Fabric 入门（使用 .NET Core）](https://azure.microsoft.com/resources/samples/service-fabric-dotnet-core-getting-started/)

<!-- Update_Description: new article about service fabric public linux app vs -->
<!--ms.date: 07/08/2018-->