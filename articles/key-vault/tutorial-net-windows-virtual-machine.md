---
title: 教程 - 将 Azure Key Vault 与通过 .NET 编写的 Windows 虚拟机配合使用 | Azure Docs
description: 本教程介绍如何将 ASP.NET Core 应用程序配置为从 Key Vault 读取机密。
services: key-vault
author: msmbaldwin
manager: rajvijan
ms.service: key-vault
ms.topic: tutorial
origin.date: 09/05/2018
ms.date: 07/01/2019
ms.author: v-biyu
ms.custom: mvc
ms.openlocfilehash: cd27e4b41e1c31c0eac0d9ee5efab2b45e052f75
ms.sourcegitcommit: 153236e4ad63e57ab2ae6ff1d4ca8b83221e3a1c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/18/2019
ms.locfileid: "67171355"
---
# <a name="tutorial-use-azure-key-vault-with-a-windows-virtual-machine-in-net"></a>教程：将 Azure Key Vault 与通过 .NET 编写的 Windows 虚拟机配合使用

Azure Key Vault 用于保护机密，例如访问应用程序、服务和 IT 资源所需的 API 密钥、数据库连接字符串。

本教程介绍如何获取控制台应用程序，以便从 Azure Key Vault 读取信息。 为此，请将托管标识用于 Azure 资源。 

本教程介绍如何：

> [!div class="checklist"]
> * 创建资源组。
> * 创建密钥保管库。
> * 将机密添加到 Key Vault。
> * 从密钥保管库检索机密。
> * 创建一个 Azure 虚拟机。
> * 为虚拟机启用托管标识。
> * 为 VM 标识分配权限。

在开始之前，请阅读 [Key Vault 的基本概念](key-vault-whatis.md#basic-concepts)。 

如果没有 Azure 订阅，请创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="prerequisites"></a>先决条件

对于 Windows、Mac 和 Linux：
  * [Git](https://git-scm.com/downloads)
  * 本教程要求在本地运行 Azure CLI。 必须安装 Azure CLI 2.0.4 或更高版本。 运行 `az --version` 即可查找版本。 如果需要安装或升级 CLI，请参阅[安装 Azure CLI 2.0](https://docs.azure.cn/cli/install-azure-cli?view=azure-cli-latest)。

## <a name="about-managed-service-identity"></a>关于托管服务标识

Azure Key Vault 可以安全地存储凭据，因此不需要在代码中显示凭据。 但是，需要对 Azure Key Vault 进行身份验证才能检索密钥。 若要对 Key Vault 进行身份验证，需要提供凭据。 因此，在启动过程中，这是一个难以兼顾的典型问题。 托管服务标识 (MSI) 提供简化该过程的启动标识，可以解决此问题。 

为 Azure 服务（例如 Azure 虚拟机、Azure 应用服务或 Azure Functions）启用 MSI 时，Azure 会创建一个[服务主体](key-vault-whatis.md#basic-concepts)。 MSI 针对 Azure Active Directory (Azure AD) 中的服务实例提供启动标识，并将服务主体凭据注入该实例。 

![MSI](media/MSI.png)

接下来，为了获取访问令牌，代码会调用 Azure 资源上提供的本地元数据服务。 代码使用从本地 MSI 终结点获取的访问令牌，以便向 Azure Key Vault 服务进行身份验证。 

## <a name="create-resources-and-assign-permissions"></a>创建资源并分配权限

在开始编码之前，需要创建一些资源，将机密放入密钥保管库中，并分配权限。

### <a name="sign-in-to-azure"></a>登录 Azure

若要使用 Azure CLI 登录到 Azure，请输入：

```azurecli
az cloud set –n  AzureChinaCloud 
az login
```

### <a name="create-a-resource-group"></a>创建资源组

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 使用 [az group create](/cli/group#az-group-create) 命令创建资源组。 

此示例在“美国西部”位置创建资源组：

```azurecli
# To list locations: az account list-locations --output table
az group create --name "<YourResourceGroupName>" --location "China North"
```

新建的资源组将在本教程中使用。

### <a name="create-a-key-vault-and-populate-it-with-a-secret"></a>创建密钥保管库并使用机密填充它

通过提供带有以下信息的 [az keyvault create](/cli/keyvault?view=azure-cli-latest#az-keyvault-create) 命令，在资源组中创建密钥保管库：

* Key Vault 名称：由 3 到 24 个字符构成的字符串，可以包含数字 (0-9)、字母 (a-z, A-Z) 和连字符 (-)
* 资源组名称
* 位置：**中国北部**。

```azurecli
az keyvault create --name "<YourKeyVaultName>" --resource-group "<YourResourceGroupName>" --location "China North"
```
目前，只有你的 Azure 帐户才有权对这个新的 Key Vault 执行操作。

现在使用 [az keyvault secret set](/cli/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-set) 命令向密钥保管库中添加机密


若要在名为 **AppSecret** 的 Key Vault 中创建机密，请输入以下命令：

```azurecli
az keyvault secret set --vault-name "<YourKeyVaultName>" --name "AppSecret" --value "MySecret"
```

此机密将存储值 **MySecret**。

### <a name="create-a-virtual-machine"></a>创建虚拟机
使用以下方法之一创建虚拟机：

[Azure CLI](https://docs.azure.cn/zh-cn/virtual-machines/windows/quick-create-cli) 
[Powershell](https://docs.azure.cn/zh-cn/virtual-machines/windows/quick-create-powershell)
[门户](https://docs.azure.cn/zh-cn/virtual-machines/windows/quick-create-portal)

### <a name="assign-an-identity-to-the-vm"></a>为 VM 分配标识
使用 [az vm identity assign](/cli/vm/identity?view=azure-cli-latest#az-vm-identity-assign) 命令为虚拟机创建系统分配的标识：

```azurecli
az vm identity assign --name <NameOfYourVirtualMachine> --resource-group <YourResourceGroupName>
```

记下以下代码中显示的系统分配的标识。 以上命令的输出为： 

```azurecli
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

### <a name="assign-permissions-to-the-vm-identity"></a>为 VM 标识分配权限
使用 [az keyvault set-policy](/cli/keyvault?view=azure-cli-latest#az-keyvault-set-policy) 命令将以前创建的标识权限分配给密钥保管库：

```azurecli
az keyvault set-policy --name '<YourKeyVaultName>' --object-id <VMSystemAssignedIdentity> --secret-permissions get list
```

### <a name="sign-in-to-the-virtual-machine"></a>登录到虚拟机

若要登录到虚拟机，请按[连接并登录到运行 Windows 的 Azure 虚拟机](https://docs.azure.cn/zh-cn/virtual-machines/windows/connect-logon)中的说明操作。

## <a name="set-up-the-console-app"></a>设置控制台应用

创建控制台应用，并使用 `dotnet` 命令安装所需的包。

### <a name="install-net-core"></a>安装 .NET Core

若要安装 .NET Core，请转到 [.NET 下载](https://www.microsoft.com/net/download)页。

### <a name="create-and-run-a-sample-net-app"></a>创建并运行示例 .NET 应用

打开命令提示符。

可以运行以下命令，将“Hello World”输出到控制台：

```console
dotnet new console -o helloworldapp
cd helloworldapp
dotnet run
```

### <a name="install-the-packages"></a>安装包

 在控制台窗口中，安装本快速入门所需的 .NET 包：

 ```console
dotnet add package System.IO;
dotnet add package System.Net;
dotnet add package System.Text;
dotnet add package Newtonsoft.Json;
dotnet add package Newtonsoft.Json.Linq;
```

## <a name="edit-the-console-app"></a>编辑控制台应用

打开 *Program.cs* 文件并添加以下包：

```csharp
using System;
using System.IO;
using System.Net;
using System.Text;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
```

编辑类文件，使之包含在下面的两步过程中使用的代码：

1. 从 VM 上的本地 MSI 终结点获取一个令牌。 这还会从 Azure AD 获取令牌。
1. 将令牌传递到 Key Vault，然后获取机密。 

```csharp
 class Program
    {
        static void Main(string[] args)
        {
            // Step 1: Get a token from the local (URI) Managed Service Identity endpoint, which in turn fetches it from Azure AD
            var token = GetToken();

            // Step 2: Fetch the secret value from your key vault
            System.Console.WriteLine(FetchSecretValueFromKeyVault(token));
        }

        static string GetToken()
        {
            WebRequest request = WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.cn");
            request.Headers.Add("Metadata", "true");
            WebResponse response = request.GetResponse();
            return ParseWebResponse(response, "access_token");
        }

        static string FetchSecretValueFromKeyVault(string token)
        {
            WebRequest kvRequest = WebRequest.Create("https://<YourVaultName>.vault.vault.azure.cn/secrets/<YourSecretName>?api-version=2016-10-01");
            kvRequest.Headers.Add("Authorization", "Bearer "+  token);
            WebResponse kvResponse = kvRequest.GetResponse();
            return ParseWebResponse(kvResponse, "value");
        }

        private static string ParseWebResponse(WebResponse response, string tokenName)
        {
            string token = String.Empty;
            using (Stream stream = response.GetResponseStream())
            {
                StreamReader reader = new StreamReader(stream, Encoding.UTF8);
                String responseString = reader.ReadToEnd();

                JObject joResponse = JObject.Parse(responseString);    
                JValue ojObject = (JValue)joResponse[tokenName];             
                token = ojObject.Value.ToString();
            }
            return token;
        }
    }
```

以上代码演示了如何在 Windows 虚拟机中通过 Azure Key Vault 执行操作。

## <a name="clean-up-resources"></a>清理资源

不再需要本教程中创建的虚拟机和 Key Vault 时，请将其删除。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Azure Key Vault REST API](https://docs.microsoft.com/rest/api/keyvault/)
