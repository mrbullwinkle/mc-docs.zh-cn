---
title: 使用 SSL 终端创建应用程序网关 - Azure CLI | Microsoft Docs
description: 了解如何使用 Azure CLI 创建应用程序网关并为 SSL 终端添加证书。
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
ms.service: application-gateway
ms.topic: article
ms.workload: infrastructure-services
origin.date: 07/14/2018
ms.date: 02/11/2019
ms.author: v-junlch
ms.openlocfilehash: 4f491a0dc519ea0e8ba2adf7cbc00c0d8a5da715
ms.sourcegitcommit: 713cf33290efd4ccc7a3eab2668e3ceb0b51686f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/12/2019
ms.locfileid: "56079646"
---
# <a name="create-an-application-gateway-with-ssl-termination-using-the-azure-cli"></a>通过 Azure CLI 使用 SSL 终端创建应用程序网关

可通过 Azure CLI 使用 [SSL 终端](application-gateway-backend-ssl.md)的证书创建使用[虚拟机规模集](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)作为后端服务器的[应用程序网关](application-gateway-introduction.md)。 在此示例中，规模集包含两个添加到应用程序网关的默认后端池的虚拟机实例。

在本文中，学习如何：

> [!div class="checklist"]
> * 创建自签名证书
> * 设置网络
> * 使用证书创建应用程序网关
> * 使用默认后端池创建虚拟机规模集

如果没有 Azure 订阅，请在开始之前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

如果选择在本地安装并使用 CLI，此快速入门教程要求运行 Azure CLI 2.0.4 版或更高版本。 若要查找版本，请运行 `az --version`。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

## <a name="create-a-self-signed-certificate"></a>创建自签名证书

为供生产使用，应导入由受信任的提供程序签名的有效证书。 对于本教程，请使用 openssl 命令创建自签名证书和 pfx 文件。

```azurecli
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out appgwcert.crt
```

输入对证书有意义的值。 可接受默认值。

```azurecli
openssl pkcs12 -export -out appgwcert.pfx -inkey privateKey.key -in appgwcert.crt
```

输入证书的密码。 在此示例中，使用 *Azure123456!* 。

## <a name="create-a-resource-group"></a>创建资源组

资源组是在其中部署和管理 Azure 资源的逻辑容器。 使用 [az group create](/cli/group) 创建资源组。

以下示例在“chinanorth”  位置创建名为“myResourceGroupAG”  的资源组。

```azurecli 
az group create --name myResourceGroupAG --location chinanorth
```

## <a name="create-network-resources"></a>创建网络资源

使用 [az network vnet create](/cli/network/vnet) 创建名为 *myVNet* 的虚拟网络和名为 *myAGSubnet* 的子网。 然后，可以使用 [az network vnet subnet create](/cli/network/vnet/subnet) 添加后端服务器所需的名为 *myBackendSubnet* 的子网。 使用 [az network public-ip create](/cli/network/public-ip) 创建名为 *myAGPublicIPAddress* 的公共 IP 地址。

```azurecli
az network vnet create `
  --name myVNet `
  --resource-group myResourceGroupAG `
  --location chinanorth `
  --address-prefix 10.0.0.0/16 `
  --subnet-name myAGSubnet `
  --subnet-prefix 10.0.1.0/24
az network vnet subnet create `
  --name myBackendSubnet `
  --resource-group myResourceGroupAG `
  --vnet-name myVNet `
  --address-prefix 10.0.2.0/24
az network public-ip create `
  --resource-group myResourceGroupAG `
  --name myAGPublicIPAddress
```

## <a name="create-the-application-gateway"></a>创建应用程序网关

可以使用 [az network application-gateway create](/cli/network/application-gateway) 创建应用程序网关。 使用 Azure CLI 创建应用程序网关时，请指定配置信息，例如容量、sku 和 HTTP 设置。 

将应用程序网关分配给之前创建的 *myAGSubnet* 和 *myAGPublicIPAddress*。 在此示例中，在创建应用程序网关时将关联所创建的证书及其密码。 

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
  --frontend-port 443 `
  --http-settings-port 80 `
  --http-settings-protocol Http `
  --public-ip-address myAGPublicIPAddress `
  --cert-file appgwcert.pfx `
  --cert-password "Azure123456!"

```

 创建应用程序网关可能需要几分钟时间。 创建应用程序网关后，可以看到它的这些新功能：

- *appGatewayBackendPool* - 应用程序网关必须至少具有一个后端地址池。
- *appGatewayBackendHttpSettings* - 指定将端口 80 和 HTTP 协议用于通信。
- *appGatewayHttpListener* - 与 *appGatewayBackendPool* 关联的默认侦听器。
- *appGatewayFrontendIP* - 将 *myAGPublicIPAddress* 分配给 *appGatewayHttpListener*。
- *rule1* - 与 *appGatewayHttpListener* 关联的默认路由规则。

## <a name="create-a-virtual-machine-scale-set"></a>创建虚拟机规模集

在此示例中，将创建虚拟机规模集，以便为应用程序网关的默认后端池提供服务器。 规模集中的虚拟机与 *myBackendSubnet* 和 *appGatewayBackendPool* 相关联。 若要创建规模集，可以使用 [az vmss create](/cli/vmss#az-vmss-create)。

```azurecli
az vmss create `
  --name myvmss `
  --resource-group myResourceGroupAG `
  --image UbuntuLTS `
  --admin-username azureuser `
  --admin-password Azure123456! `
  --instance-count 2 `
  --vnet-name myVNet `
  --subnet myBackendSubnet `
  --vm-sku Standard_DS2 `
  --upgrade-policy-mode Automatic `
  --app-gateway myAppGateway `
  --backend-pool-name appGatewayBackendPool
```

### <a name="install-nginx"></a>安装 NGINX

```azurecli
az vmss extension set `
  --publisher Microsoft.Azure.Extensions `
  --version 2.0 `
  --name CustomScript `
  --resource-group myResourceGroupAG `
  --vmss-name myvmss `
  --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'
```

## <a name="test-the-application-gateway"></a>测试应用程序网关

若要获取应用程序网关的公共 IP 地址，可以使用 [az network public-ip show](https://docs.microsoft.com/en-us/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-show)。 复制该公共 IP 地址，并将其粘贴到浏览器的地址栏。

```azurepowershell
az network public-ip show `
  --resource-group myResourceGroupAG `
  --name myAGPublicIPAddress `
  --query [ipAddress] `
  --output tsv
```

![安全警告](./media/application-gateway-ssl-cli/application-gateway-secure.png)

若要接受有关使用自签名证书的安全警告，请依次选择“详细信息”和“继续转到网页”。   随即显示受保护的 NGINX 站点，如下例所示：

![在应用程序网关中测试基 URL](./media/application-gateway-ssl-cli/application-gateway-nginx.png)

## <a name="next-steps"></a>后续步骤

在本教程中，你已学习了如何执行以下操作：

> [!div class="checklist"]
> * 创建自签名证书
> * 设置网络
> * 使用证书创建应用程序网关
> * 使用默认后端池创建虚拟机规模集

若要详细了解应用程序网关及其关联的资源，请继续阅读操作指南文章。

<!-- Update_Description: link update -->