---
title: 使用 .NET 向 Azure Key Vault 进行服务到服务身份验证
description: 使用 Microsoft.Azure.Services.AppAuthentication 库通过 .NET 向 Azure Key Vault 进行身份验证。
keywords: azure key-vault 身份验证本地凭据
author: msmbaldwin
manager: barbkess
services: key-vault
ms.author: v-biyu
origin.date: 11/15/2017
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: key-vault
ms.openlocfilehash: 50d91b2d5ee6ff6d507d65f33f5e2f068f1c4fcf
ms.sourcegitcommit: 5f260ee1d8ac487702b554a94cb971a3ee62a40b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2019
ms.locfileid: "68232310"
---
# <a name="service-to-service-authentication-to-azure-key-vault-using-net"></a>使用 .NET 向 Azure Key Vault 进行服务到服务身份验证

若要对 Azure Key Vault 进行身份验证，需要提供 Azure Active Directory (AD) 凭据（共享机密或证书）。 

管理此类凭据可能很困难，因此可以考虑将凭据绑定到应用中，方法是将其包括到源或配置文件中。  用于 .NET 库的 `Microsoft.Azure.Services.AppAuthentication` 简化了此问题。 它使用开发人员的凭据在本地开发期间进行身份验证。 随后将解决方案部署到 Azure 时，该库会自动切换到应用程序凭据。    在本地开发期间使用开发人员凭据更安全，因为不需创建 Azure AD 凭据，或者不需在开发人员之间共享凭据。

`Microsoft.Azure.Services.AppAuthentication` 库自动管理身份验证，这样你就可以专注于自己的解决方案而非凭据。  该库支持使用 Microsoft Visual Studio、Azure CLI 或 Azure AD 集成身份验证进行本地开发。 部署到支持托管标识的 Azure 资源时，该库会自动使用 Azure 资源的托管标识。 不需代码或配置更改。 当托管标识不可用时，或者当开发人员的安全上下文不能在本地开发期间确定时，该库还支持直接使用 Azure AD [客户端凭据](../azure-resource-manager/resource-group-authenticate-service-principal.md)。

## <a name="using-the-library"></a>使用库

对于 .NET 应用程序，若要使用托管标识，最简单的方式是通过 `Microsoft.Azure.Services.AppAuthentication` 包。 下面介绍如何入门：

1. 向应用程序添加对 [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) 和 [Microsoft.Azure.KeyVault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault) NuGet 包的引用。 

2. 添加以下代码：

    ``` csharp
    using Microsoft.Azure.Services.AppAuthentication;
    using Microsoft.Azure.KeyVault;

    // Instantiate a new KeyVaultClient object, with an access token to Key Vault
    var azureServiceTokenProvider1 = new AzureServiceTokenProvider();
    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider1.KeyVaultTokenCallback));

    // Optional: Request an access token to other Azure services
    var azureServiceTokenProvider2 = new AzureServiceTokenProvider();
    string accessToken = await azureServiceTokenProvider2.GetAccessTokenAsync("https://management.azure.com/").ConfigureAwait(false);
    ```

`AzureServiceTokenProvider` 类将令牌缓存在内存中，在过期前才将其从 Azure AD 检索出来。 结果就是，不再需要在调用 `GetAccessTokenAsync` 方法之前检查是否过期。 在需要使用令牌时直接调用该方法即可。 

`GetAccessTokenAsync` 方法需要资源标识符。

## <a name="local-development-authentication"></a>本地开发身份验证

对于本地开发，有两种主要的身份验证方案：[对 Azure 服务进行身份验证](#authenticating-to-azure-services)；[对自定义服务进行身份验证](#authenticating-to-custom-services)。

### <a name="authenticating-to-azure-services"></a>向 Azure 服务进行身份验证

本地计算机不支持 Azure 资源的托管标识。  因此，`Microsoft.Azure.Services.AppAuthentication` 库使用开发人员凭据在本地开发环境中运行。 当解决方案部署到 Azure 时，该库使用托管标识切换到 OAuth 2.0 客户端凭据授予流。  这意味着可以对同一代码进行本地和远程测试，无需担心。

对于本地开发，`AzureServiceTokenProvider` 使用 **Visual Studio**、**Azure 命令行界面** (CLI) 或 **Azure AD 集成身份验证**提取令牌。 将按顺序尝试每个选项，该库会使用获得成功的第一个选项。 如果没有选项成功，则会引发一个包含详细信息的 `AzureServiceTokenProviderException` 意外。

### <a name="authenticating-with-visual-studio"></a>使用 Visual Studio 进行身份验证

使用 Visual Studio 进行身份验证包含以下先决条件：

1. [Visual Studio 2017 v15.5](https://blogs.msdn.microsoft.com/visualstudio/2017/10/11/visual-studio-2017-version-15-5-preview/) 或更高版本。

2. [Visual Studio 的应用身份验证扩展](https://go.microsoft.com/fwlink/?linkid=862354)以 Visual Studio 2017 Update 5 的独立扩展形式发布，与 Update 6 及更高版本中的产品绑定在一起。 使用 Update 6 或更高版本时，可以验证是否安装了应用身份验证扩展，方法是在 Visual Studio 安装程序中选择 Azure 开发工具。
 
登录到 Visual Studio，通过“工具”  &nbsp;>&nbsp;  “选项”&nbsp;>&nbsp;  “Azure 服务身份验证”选择一个用于本地开发的帐户。 

如果在使用 Visual Studio 时遇到问题，例如有关令牌提供程序文件的错误，请仔细查看这些步骤。 

可能还需对开发人员令牌重新进行身份验证。 为此，请转到“工具”  &nbsp;>&nbsp;  “选项”> **&nbsp;&nbsp;** “Azure 服务身份验证”，查找所选帐户下的“重新进行身份验证”  链接。  选择该链接进行身份验证。 

### <a name="authenticating-with-azure-cli"></a>使用 Azure CLI 进行身份验证

若要使用 Azure CLI 进行本地开发，请执行以下操作：

1. 安装 [Azure CLI v2.0.12](/cli/install-azure-cli) 或更高版本。 升级早期版本。 

2. 使用 **az login** 登录到 Azure。

请使用 `az account get-access-token` 验证访问权限。  如果收到错误，请验证步骤 1 是否已成功完成。 

如果没有将 Azure CLI 安装到默认目录，则可能会收到错误，指出 `AzureServiceTokenProvider` 找不到 Azure CLI 的路径。  请使用 **AzureCLIPath** 环境变量来定义 Azure CLI 安装文件夹。 `AzureServiceTokenProvider` 在需要时将 **AzureCLIPath** 环境变量中指定的目录添加到 **Path** 环境变量。

如果使用多个帐户登录到 Azure CLI，或者帐户可以访问多个订阅，则需指定要使用的具体订阅。  为此，请使用：

```
az account set --subscription <subscription-id>
```

此命令仅在发生故障时生成输出。  若要验证当前帐户设置，请使用：

```
az account list
```

### <a name="authenticating-with-azure-ad-authentication"></a>使用 Azure AD 身份验证进行身份验证

若要使用 Azure AD 身份验证，请验证：

- 本地 Active Directory 是否已同步到 Azure AD

- 代码是否在已加入域的计算机上运行。


### <a name="authenticating-to-custom-services"></a>向自定义服务进行身份验证

当某个服务调用 Azure 服务时，上述步骤适用，因为 Azure 服务允许访问用户和应用程序。  

创建调用自定义服务的服务时，请使用 Azure AD 客户端凭据进行本地开发身份验证。  有两个选项： 

1.  使用服务主体登录到 Azure：

    1.  [创建服务主体](/cli/create-an-azure-service-principal-azure-cli)。

    2.  使用 Azure CLI 登录：

        ```
        az login --service-principal -u <principal-id> --password <password> --tenant <tenant-id> --allow-no-subscriptions
        ```

        由于服务主体可能没有订阅访问权限，因此请使用 `--allow-no-subscriptions` 参数。

2.  使用环境变量来指定服务主体详细信息。  有关详细信息，请参阅[使用服务主体运行应用程序](#running-the-application-using-a-service-principal)。

登录到 Azure 以后，`AzureServiceTokenProvider` 使用服务主体来检索本地开发的令牌。

这仅适用于本地开发。 当解决方案部署到 Azure 时，该库会切换到托管标识以进行身份验证。

## <a name="running-the-application-using-managed-identity-or-user-assigned-identity"></a>使用托管标识或用户分配标识运行应用程序 

在启用托管标识的 Azure 应用服务或 Azure VM 上运行代码时，库自动使用托管标识。 


## <a name="running-the-application-using-a-service-principal"></a>使用服务主体运行应用程序 

可能需要创建一个用于身份验证的 Azure AD 客户端凭据。 常见示例包括：

1. 代码运行在本地开发环境中，但没有使用开发人员的标识。  例如，Service Fabric 使用 [NetworkService 帐户](https://docs.azure.cn/zh-cn/service-fabric/service-fabric-application-secret-management)进行本地开发。
 
2. 代码在本地开发环境中运行，而身份验证则通过自定义服务进行，因此不能使用开发人员标识。 
 
3. 代码在尚不支持 Azure 资源的托管标识的 Azure 计算资源（例如 Azure Batch）上运行。

可通过三种主要方法使用服务主体来运行应用程序。 若要使用其中的任何方法，必须先[创建服务主体](/cli/create-an-azure-service-principal-azure-cli)。

### <a name="use-a-certificate-in-local-keystore-to-sign-into-azure-ad"></a>使用本地密钥存储中的证书登录到 Azure AD

1. 使用 Azure CLI [az ad sp create-for-rbac](/cli/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) 命令创建服务主体证书。 

    ```azurecli
    az ad sp create-for-rbac --create-cert
    ```

    这会创建一个存储在主目录中的 .pem 文件（私钥）。 将此证书部署到 *LocalMachine* 或 *CurrentUser* 存储。 


3. 将名为 **AzureServicesAuthConnectionString** 的环境变量设置为：

    ```
    RunAs=App;AppId={AppId};TenantId={TenantId};CertificateThumbprint={Thumbprint};
          CertificateStoreLocation={CertificateStore}
    ```
 
    将  {AppId}、  {TenantId} 和  {Thumbprint} 替换为步骤 1 中生成的值。 根据部署计划，将 *{CertificateStore}* 替换为 `LocalMachine` 或 `CurrentUser`。

4. 运行应用程序。 

### <a name="use-a-shared-secret-credential-to-sign-into-azure-ad"></a>使用共享机密凭据登录到 Azure AD

1. 使用 [az ad sp create-for-rbac --password](/cli/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) 创建包含密码的服务主体证书。 

2. 将名为 **AzureServicesAuthConnectionString** 的环境变量设置为：

    ```
    RunAs=App;AppId={AppId};TenantId={TenantId};AppKey={ClientSecret} 
    ```

    将 _{AppId}_ 、 _{TenantId}_ 和 _{ClientSecret}_ 替换为步骤 1 中生成的值。

3. 运行应用程序。 

一切正确设置以后，不需进一步更改代码。  `AzureServiceTokenProvider` 使用环境变量和证书向 Azure AD 进行身份验证。 

### <a name="use-a-certificate-in-key-vault-to-sign-into-azure-ad"></a>使用 Key Vault 中的证书登录到 Azure AD

使用此选项可将服务主体的客户端证书存储在 Key Vault 中，并将其用于服务主体身份验证。 在以下情况下，可以使用此选项：

* 本地身份验证：你想要使用显式服务主体进行身份验证，并希望将服务主体凭据安全保存在 Key Vault 中。 开发人员帐户必须有权访问 Key Vault。 
* 从 Azure 进行身份验证：你想要使用显式凭据（例如，对于跨租户方案），并希望将服务主体凭据安全保存在 Key Vault 中。 托管标识必须有权访问 Key Vault。 

托管标识或开发人员标识必须有权从 Key Vault 检索客户端证书。 AppAuthentication 库使用检索到的证书作为服务主体的客户端凭据。

使用客户端证书进行服务主体身份验证

1. 使用 Azure CLI [az ad sp create-for-rbac --keyvault <keyvaultname> --cert <certificatename> --create-cert --skip-assignment](/cli/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) 命令创建一个服务主体证书，并自动将其存储在 Key Vault 中：

    ```azurecli
    az ad sp create-for-rbac --keyvault <keyvaultname> --cert <certificatename> --create-cert --skip-assignment
    ```
    
    证书标识符是采用 `https://<keyvaultname>.vault.azure.cn/secrets/<certificatename>` 格式的 URL

1. 请将此连接字符串中的 `{KeyVaultCertificateSecretIdentifier}` 替换为证书标识符：

    ```
    RunAs=App;AppId={TestAppId};KeyVaultCertificateSecretIdentifier={KeyVaultCertificateSecretIdentifier}
    ```

    例如，如果 Key Vault 名为“myKeyVault”，而你创建了名为“myCert”的证书，则证书标识符为：

    ```
    RunAs=App;AppId={TestAppId};KeyVaultCertificateSecretIdentifier=https://myKeyVault.vault.azure.cn/secrets/myCert
    ```


## <a name="connection-string-support"></a>连接字符串支持

默认情况下，`AzureServiceTokenProvider` 使用多种方法来检索令牌。 

若要控制此过程，请使用传递到 `AzureServiceTokenProvider` 构造函数或在  AzureServicesAuthConnectionString 环境变量中指定的连接字符串。 

可以使用以下选项：

| 连接字符串选项 | 方案 | 注释|
|:--------------------------------|:------------------------|:----------------------------|
| `RunAs=Developer; DeveloperTool=AzureCli` | 本地开发 | AzureServiceTokenProvider 使用 AzureCli 来获取令牌。 |
| `RunAs=Developer; DeveloperTool=VisualStudio` | 本地开发 | AzureServiceTokenProvider 使用 Visual Studio 来获取令牌。 |
| `RunAs=CurrentUser` | 本地开发 | AzureServiceTokenProvider 使用 Azure AD 集成身份验证来获取令牌。 |
| `RunAs=App` | Azure 资源的托管标识 | AzureServiceTokenProvider 使用托管标识来获取令牌。 |
| `RunAs=App;AppId={ClientId of user-assigned identity}` | Azure 资源的用户分配标识 | AzureServiceTokenProvider 使用用户分配标识来获取令牌。 |
| `RunAs=App;AppId={TestAppId};KeyVaultCertificateSecretIdentifier={KeyVaultCertificateSecretIdentifier}` | 自定义服务身份验证 | KeyVaultCertificateSecretIdentifier = 证书的机密标识符。 |
| `RunAs=App;AppId={AppId};TenantId={TenantId};CertificateThumbprint={Thumbprint};CertificateStoreLocation={LocalMachine or CurrentUser}`| 服务主体 | `AzureServiceTokenProvider` 使用证书从 Azure AD 获取令牌。 |
| `RunAs=App;AppId={AppId};TenantId={TenantId};CertificateSubjectName={Subject};CertificateStoreLocation={LocalMachine or CurrentUser}` | 服务主体 | `AzureServiceTokenProvider` 使用证书从 Azure AD 获取令牌|
| `RunAs=App;AppId={AppId};TenantId={TenantId};AppKey={ClientSecret}` | 服务主体 |`AzureServiceTokenProvider` 使用机密从 Azure AD 获取令牌。 |

## <a name="samples"></a>示例

若要了解 `Microsoft.Azure.Services.AppAuthentication` 库的运作方式，请参阅以下代码示例。

1. [在运行时使用托管标识从 Azure Key Vault 检索机密](https://github.com/Azure-Samples/app-service-msi-keyvault-dotnet)

2. [Programmatically deploy an Azure Resource Manager template from an Azure VM with a managed identity](https://github.com/Azure-Samples/windowsvm-msi-arm-dotnet)（使用托管标识以编程方式从 Azure VM 部署 Azure 资源管理器模板）。

3. [Use .NET Core sample and a managed identity to call Azure services from an Azure Linux VM](https://github.com/Azure-Samples/linuxvm-msi-keyvault-arm-dotnet/)（使用 .NET Core 示例和托管标识从 Azure Linux VM 调用 Azure 服务）。

## <a name="next-steps"></a>后续步骤

- 了解[对应用进行身份验证和授权](https://docs.azure.cn/zh-cn/app-service/app-service-authentication-overview)的不同方式。
- 详细了解 Azure AD [身份验证方案](https://docs.azure.cn/zh-cn/active-directory/develop/authentication-scenarios#web-browser-to-web-application)。
