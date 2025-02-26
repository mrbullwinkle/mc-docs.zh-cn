---
title: 创建支持外部流量重定向的应用程序网关 - Azure CLI | Microsoft Docs
description: 了解如何创建应用程序网关，将内部 web 流量重定向到相应的池中使用 Azure CLI。
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 01/24/2018
ms.date: 02/11/2019
ms.author: v-junlch
ms.openlocfilehash: 426a2e1fcac5b0bc1d7eddc18f34e75708fc33f6
ms.sourcegitcommit: b8fb6890caed87831b28c82738d6cecfe50674fd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/29/2019
ms.locfileid: "58626773"
---
# <a name="create-an-application-gateway-with-external-redirection-using-the-azure-cli"></a>使用 Azure CLI 创建支持外部重定向的应用程序网关

你可以使用 Azure CLI 配置 [web 流量重定向](multiple-site-overview.md)创建时[应用程序网关](overview.md)。 在本教程中，可以配置侦听器和重定向到外部站点的应用程序网关在到达的 web 流量的规则。

在本文中，学习如何：

> [!div class="checklist"]
> * 设置网络
> * 创建侦听器和重定向规则
> * 创建应用程序网关

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

如果选择在本地安装并使用 CLI，此快速入门教程要求运行 Azure CLI 2.0.4 版或更高版本。 若要查找版本，请运行 `az --version`。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

## <a name="create-a-resource-group"></a>创建资源组

资源组是在其中部署和管理 Azure 资源的逻辑容器。 使用 [az group create](/cli/group) 创建资源组。

以下示例在“chinanorth”  位置创建名为“myResourceGroupAG”  的资源组。

```azurecli 
az group create --name myResourceGroupAG --location chinanorth
```

## <a name="create-network-resources"></a>创建网络资源 

使用 [az network vnet create](/cli/network/vnet) 创建名为 *myVNet* 的虚拟网络和名为 *myAGSubnet* 的子网。 使用 [az network public-ip create](https://docs.azure.cn/zh-cn/cli/network/public-ip?view=azure-cli-latest#az-network-public-ip-create) 创建名为 *myAGPublicIPAddress* 的公共 IP 地址。 这些资源用于提供与应用程序网关及其关联资源的网络连接。

```azurecli
az network vnet create `
  --name myVNet `
  --resource-group myResourceGroupAG `
  --location chinanorth `
  --address-prefix 10.0.0.0/16 `
  --subnet-name myAGSubnet `
  --subnet-prefix 10.0.1.0/24
az network public-ip create `
  --resource-group myResourceGroupAG `
  --name myAGPublicIPAddress
```

## <a name="create-an-application-gateway"></a>创建应用程序网关

可以使用 [az network application-gateway create](/cli/network/application-gateway) 创建名为 *myAppGateway* 的应用程序网关。 使用 Azure CLI 创建应用程序网关时，请指定配置信息，例如容量、sku 和 HTTP 设置。 将应用程序网关分配给之前创建的 *myAGSubnet* 和 *myPublicIPAddress*。 

```azurecli
az network application-gateway create `
  --name myAppGateway `
  --location chinanorth `
  --resource-group myResourceGroupAG `
  --vnet-name myVNet `
  --subnet myAGsubnet `
  --capacity 2 `
  --sku Standard_Medium `
  --http-settings-cookie-based-affinity Disabled `
  --frontend-port 8080 `
  --http-settings-port 80 `
  --http-settings-protocol Http `
  --public-ip-address myAGPublicIPAddress
```

创建应用程序网关可能需要几分钟时间。 创建应用程序网关后，可以看到它的这些新功能：

- *appGatewayBackendPool* - 应用程序网关必须至少具有一个后端地址池。
- *appGatewayBackendHttpSettings* - 指定将端口 80 和 HTTP 协议用于通信。
- *appGatewayHttpListener* - 与 *appGatewayBackendPool* 关联的默认侦听器。
- *appGatewayFrontendIP* - 将 *myAGPublicIPAddress* 分配给 *appGatewayHttpListener*。
- *rule1* - 与 *appGatewayHttpListener* 关联的默认路由规则。

### <a name="add-the-redirection-configuration"></a>添加重定向配置

使用 [az network application-gateway redirect-config create](/cli/network/application-gateway/redirect-config) 将流量从 <em>www.consoto.org</em> 发送到 <em>www.contoso.com</em> 的侦听器的重定向配置添加到应用程序网关。

```azurecli
az network application-gateway redirect-config create `
  --name myredirect `
  --gateway-name myAppGateway `
  --resource-group myResourceGroupAG `
  --type Temporary `
  --target-url "https://bing.com"
```

### <a name="add-a-listener-and-routing-rule"></a>添加侦听器和路由规则

应用程序网关需要侦听器才能适当地将流量路由到后端池。 使用创建侦听器[az 网络应用程序网关 http 侦听器创建](/cli/network/application-gateway)使用创建的前端端口与[az 网络应用程序网关前端端口创建](/cli/network/application-gateway)。 侦听器需要使用规则来了解哪个后端池使用传入流量。 使用 [az network application-gateway rule create](/cli/network/application-gateway) 创建名为 *redirectRule* 的基本规则。

```azurecli
az network application-gateway frontend-port create `
  --port 80 `
  --gateway-name myAppGateway `
  --resource-group myResourceGroupAG `
  --name redirectPort
az network application-gateway http-listener create `
  --name redirectListener `
  --frontend-ip appGatewayFrontendIP `
  --frontend-port redirectPort `
  --resource-group myResourceGroupAG `
  --gateway-name myAppGateway
az network application-gateway rule create `
  --gateway-name myAppGateway `
  --name redirectRule `
  --resource-group myResourceGroupAG `
  --http-listener redirectListener `
  --rule-type Basic `
  --redirect-config myredirect
```

## <a name="test-the-application-gateway"></a>测试应用程序网关

若要获取应用程序网关的公共 IP 地址，可以使用 [az network public-ip show](https://docs.azure.cn/zh-cn/cli/network/public-ip?view=azure-cli-latest#az-network-public-ip-show)。 复制该公共 IP 地址，并将其粘贴到浏览器的地址栏。

应该会看到 *bing.com* 出现在浏览器中。

## <a name="next-steps"></a>后续步骤

在本教程中，你已学习了如何执行以下操作：

> * 设置网络
> * 创建侦听器和重定向规则
> * 创建应用程序网关

<!-- Update_Description: link update -->