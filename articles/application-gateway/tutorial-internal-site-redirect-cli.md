---
title: 创建支持内部重定向的应用程序网关 - Azure CLI | Microsoft Docs
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
origin.date: 07/14/2018
ms.date: 04/17/2019
ms.author: v-junlch
ms.openlocfilehash: 946520c30cbe45c53c80f016c9154178aa5fb7ee
ms.sourcegitcommit: bf3df5d77e5fa66825fe22ca8937930bf45fd201
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2019
ms.locfileid: "59686388"
---
# <a name="create-an-application-gateway-with-internal-redirection-using-the-azure-cli"></a>使用 Azure CLI 创建支持内部重定向的应用程序网关

你可以使用 Azure CLI 配置 [web 流量重定向](application-gateway-multi-site-overview.md)创建时[应用程序网关](application-gateway-introduction.md)。 在本教程中，将使用虚拟机规模集创建后端池。 然后，基于所拥有的域配置侦听器和规则，以确保 Web 流量可到达相应池。 本教程假定你拥有多个域，并使用示例 *www\.contoso.com* 和 *www\.contoso.org*。

在本文中，学习如何：

> [!div class="checklist"]
> * 设置网络
> * 创建应用程序网关
> * 添加侦听器和重定向规则
> * 使用后端池创建虚拟机规模集
> * 在域中创建 CNAME 记录

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

如果选择在本地安装并使用 CLI，此快速入门教程要求运行 Azure CLI 2.0.4 版或更高版本。 若要查找版本，请运行 `az --version`。 如果需要进行安装或升级，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

## <a name="create-a-resource-group"></a>创建资源组

资源组是在其中部署和管理 Azure 资源的逻辑容器。 使用 [az group create](/cli/group) 创建资源组。

以下示例在“chinanorth”  位置创建名为“myResourceGroupAG”  的资源组。

```azurecli 
az group create --name myResourceGroupAG --location chinanorth
```

## <a name="create-network-resources"></a>创建网络资源 

使用 [az network vnet create](/cli/network/vnet) 创建名为 *myVNet* 的虚拟网络和名为 *myAGSubnet* 的子网。 然后，可以使用 [az network vnet subnet create](/cli/network/vnet/subnet) 添加后端服务器池所需的名为 *myBackendSubnet* 的子网。 使用 [az network public-ip create](https://docs.azure.cn/zh-cn/cli/network/public-ip?view=azure-cli-latest#az-network-public-ip-create) 创建名为 *myAGPublicIPAddress* 的公共 IP 地址。

```azurecli
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location chinanorth \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24
az network vnet subnet create \
  --name myBackendSubnet \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --address-prefix 10.0.2.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress
```

## <a name="create-the-application-gateway"></a>创建应用程序网关

可以使用 [az network application-gateway create](/cli/network/application-gateway) 创建名为 *myAppGateway* 的应用程序网关。 使用 Azure CLI 创建应用程序网关时，请指定配置信息，例如容量、sku 和 HTTP 设置。 将应用程序网关分配给之前创建的 *myAGSubnet* 和 *myAGPublicIPAddress*。 

```azurecli
az network application-gateway create \
  --name myAppGateway \
  --location chinanorth \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGsubnet \
  --capacity 2 \
  --sku Standard_Medium \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress
```

创建应用程序网关可能需要几分钟时间。 创建应用程序网关后，可以看到它的这些新功能：

- *appGatewayBackendPool* - 应用程序网关必须至少具有一个后端地址池。
- *appGatewayBackendHttpSettings* - 指定将端口 80 和 HTTP 协议用于通信。
- *appGatewayHttpListener* - 与 *appGatewayBackendPool* 关联的默认侦听器。
- *appGatewayFrontendIP* - 将 *myAGPublicIPAddress* 分配给 *appGatewayHttpListener*。
- *rule1* - 与 *appGatewayHttpListener* 关联的默认路由规则。


## <a name="add-listeners-and-rules"></a>添加侦听器和规则 

应用程序网关需要侦听器才能适当地将流量路由到后端池。 在本教程中，将为两个域创建两个侦听器。 在此示例中，将为域 *www\.contoso.com* 和 *www\.contoso.org* 创建侦听器。

使用 [az network application-gateway http-listener create](/cli/network/application-gateway) 添加路由流量所需的后端侦听器。

```azurecli
az network application-gateway http-listener create \
  --name contosoComListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.com
az network application-gateway http-listener create \
  --name contosoOrgListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port appGatewayFrontendPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway \
  --host-name www.contoso.org   
  ```

### <a name="add-the-redirection-configuration"></a>添加重定向配置

使用 [az network application-gateway redirect-config create](/cli/network/application-gateway/redirect-config) 在应用程序网关中添加从 *www\.consoto.org* 将流量发送到 *www\.contoso.com* 的侦听器的重定向配置。

```azurecli
az network application-gateway redirect-config create \
  --name orgToCom \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --type Permanent \
  --target-listener contosoComListener \
  --include-path true \
  --include-query-string true
```

### <a name="add-routing-rules"></a>添加路由规则

规则按照其创建顺序进行处理，并且使用与发送到应用程序网关的 URL 匹配的第一个规则定向流量。 本教程中不需要已创建的默认基本规则。 在此示例中，将创建名为 *contosoComRule* 和 *contosoOrgRule* 的两个新规则并删除已创建的默认规则。  可以使用 [az network application-gateway rule create](/cli/network/application-gateway) 添加规则。

```azurecli
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoComRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoComListener \
  --rule-type Basic \
  --address-pool appGatewayBackendPool
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name contosoOrgRule \
  --resource-group myResourceGroupAG \
  --http-listener contosoOrgListener \
  --rule-type Basic \
  --redirect-config orgToCom
az network application-gateway rule delete \
  --gateway-name myAppGateway \
  --name rule1 \
  --resource-group myResourceGroupAG
```

## <a name="create-virtual-machine-scale-sets"></a>创建虚拟机规模集

在此示例中，将创建一个虚拟机规模集以支持已创建的默认后端池。 创建的规模集名为 *myvmss*，并包含两个在其上安装了 NGINX 的虚拟机实例。

```azurecli
az vmss create \
  --name myvmss \
  --resource-group myResourceGroupAG \
  --image UbuntuLTS \
  --admin-username azureuser \
  --admin-password Azure123456! \
  --instance-count 2 \
  --vnet-name myVNet \
  --subnet myBackendSubnet \
  --vm-sku Standard_DS2 \
  --upgrade-policy-mode Automatic \
  --app-gateway myAppGateway \
  --backend-pool-name appGatewayBackendPool
```

### <a name="install-nginx"></a>安装 NGINX

```azurecli
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroupAG \
  --vmss-name myvmss \
  --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'
```

## <a name="create-cname-record-in-your-domain"></a>在域中创建 CNAME 记录

使用其公共 IP 地址创建应用程序网关后，可以获取 DNS 地址并使用它在域中创建 CNAME 记录。 可以使用 [az network public-ip show](https://docs.azure.cn/zh-cn/cli/network/public-ip?view=azure-cli-latest#az-network-public-ip-show) 获取应用程序网关的 DNS 地址。 复制 DNSSettings 的 *fqdn* 值并使用它作为所创建的 CNAME 记录的值。 不建议使用 A 记录，因为重新启动应用程序网关后 VIP 可能会变化。

```azurecli
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [dnsSettings.fqdn] \
  --output tsv
```

## <a name="test-the-application-gateway"></a>测试应用程序网关

在浏览器的地址栏中输入域名。 例如， http://www.contoso.com 。

![在应用程序网关中测试 contoso 站点](./media/tutorial-internal-site-redirect-cli/application-gateway-nginxtest.png)

将地址更改为其他域（例如 http://www.contoso.org ），应会看到流量已被重定向回 www\. contoso.com 的侦听器。

## <a name="next-steps"></a>后续步骤

在本教程中，你已学习了如何执行以下操作：

> [!div class="checklist"]
> * 设置网络
> * 创建应用程序网关
> * 添加侦听器和重定向规则
> * 使用后端池创建虚拟机规模集
> * 在域中创建 CNAME 记录

> [!div class="nextstepaction"]
> [详细了解应用程序网关的作用](./application-gateway-introduction.md)

<!-- Update_Description: wording update -->