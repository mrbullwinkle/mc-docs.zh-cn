---
title: 快速入门：创建公共标准负载均衡器 - Azure CLI
titlesuffix: Azure Load Balancer
description: 本快速入门介绍如何使用 Azure CLI 创建公共负载均衡器
services: load-balancer
documentationcenter: na
author: WenJason
manager: digimobile
tags: azure-resource-manager
Customer intent: I want to create a Standard Load balancer so that I can load balance internet traffic to VMs.
ms.assetid: a8bcdd88-f94c-4537-8143-c710eaa86818
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 01/25/2019
ms.date: 03/04/2019
ms.author: v-jay
ms.custom: mvc
ms.openlocfilehash: d043809ec8f3fc2628c178e4242d50bbba9248c9
ms.sourcegitcommit: e9f088bee395a86c285993a3c6915749357c2548
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/26/2019
ms.locfileid: "56836880"
---
# <a name="quickstart-create-a-standard-load-balancer-to-load-balance-vms-using-azure-cli"></a>快速入门：使用 Azure CLI 创建标准负载均衡器以对 VM 进行负载均衡

本快速入门演示如何创建标准负载均衡器。 为了测试负载均衡器，需要部署两个运行 Ubuntu 服务器的虚拟机 (VM)，并在两个 VM 之间对一个 Web 应用进行负载均衡。

本教程要求运行 Azure CLI 2.0.28 或更高版本。 若要查找版本，请运行 `az --version`。 如需进行安装或升级，请参阅[安装 Azure CLI]( /cli/install-azure-cli)。

## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](/cli/group) 创建资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

以下示例在“chinanorth”  位置创建名为“myResourceGroupSLB”  的资源组：

```cli
  az group create \
    --name myResourceGroupSLB \
    --location chinanorth
```

## <a name="create-a-public-standard-ip-address"></a>创建公共标准 IP 地址

若要通过 Internet 访问 Web 应用，需要负载均衡器有一个公共 IP 地址。 标准负载均衡器仅支持标准公共 IP 地址。 使用 [az network public-ip create](/cli/network/public-ip) 在 *myResourceGroupSLB* 中创建名为 *myPublicIP* 的标准公共 IP 地址。

```cli
  az network public-ip create --resource-group myResourceGroupSLB --name myPublicIP --sku standard
```

## <a name="create-azure-load-balancer"></a>创建 Azure 负载均衡器

本部分详细介绍如何创建和配置负载均衡器的以下组件：
  - 前端 IP 池，用于在负载均衡器上接收传入网络流量。
  - 后端 IP 池，前端池将负载均衡的网络流量发送到此处。
  - 运行状况探测，用于确定后端 VM 实例的运行状况。
  - 负载均衡器规则，用于定义如何将流量分配给 VM。

### <a name="create-the-load-balancer"></a>创建负载均衡器

使用 [az network lb create](/cli/network/lb?view=azure-cli-latest) 创建名为 **myLoadBalancer** 的公共 Azure 负载均衡器，该负载均衡器包括名为 **myFrontEnd** 的前端池、名为 **myBackEndPool** 的后端池（与在前一步中创建的公共 IP 地址 **myPublicIP** 相关联）。

```cli
  az network lb create \
    --resource-group myResourceGroupSLB \
    --name myLoadBalancer \
    --sku standard
    --public-ip-address myPublicIP \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool       
  ```

### <a name="create-the-health-probe"></a>创建运行状况探测

运行状况探测器将检查所有虚拟机实例，以确保它们可以发送网络流量。 探测器检查失败的虚拟机实例将从负载均衡器中删除，直到它恢复联机状态并且探测器检查确定它运行正常。 使用 [az network lb probe create](/cli/network/lb/probe?view=azure-cli-latest) 创建运行状况探测，以监视虚拟机的运行状况。 

```cli
  az network lb probe create \
    --resource-group myResourceGroupSLB \
    --lb-name myLoadBalancer \
    --name myHealthProbe \
    --protocol tcp \
    --port 80   
```

### <a name="create-the-load-balancer-rule"></a>创建负载均衡器规则

负载均衡器规则定义传入流量的前端 IP 配置和后端 IP 池以接收流量，同时定义所需源和目标端口。 使用 [az network lb rule create](/cli/network/lb/rule?view=azure-cli-latest) 创建负载均衡器规则 *myLoadBalancerRuleWeb*，以便侦听前端池 *myFrontEnd* 中的端口 80，并且将经过负载均衡的网络流量发送到也使用端口 80 的后端地址池 *myBackEndPool*。 

```cli
  az network lb rule create \
    --resource-group myResourceGroupSLB \
    --lb-name myLoadBalancer \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe  
```

## <a name="configure-virtual-network"></a>配置虚拟网络

需要先创建支持的虚拟网络资源，然后才能部署一些 VM 并测试负载均衡器。

### <a name="create-a-virtual-network"></a>创建虚拟网络

使用 [az network vnet create](/cli/network/vnet) 在 *myResourceGroup* 中创建名为 *myVnet* 的虚拟网络，该虚拟网络包含名为 *mySubnet* 的子网。

```cli
  az network vnet create \
    --resource-group myResourceGroupSLB \
    --location chinanorth \
    --name myVnet \
    --subnet-name mySubnet
```
###  <a name="create-a-network-security-group"></a>创建网络安全组

对于标准负载均衡器，后端地址池中的 VM 需要具有属于网络安全组的 NIC。 创建网络安全组，以定义虚拟网络的入站连接。

```cli
  az network nsg create \
    --resource-group myResourceGroupSLB \
    --name myNetworkSecurityGroup
```

### <a name="create-a-network-security-group-rule"></a>创建网络安全组规则

创建网络安全组规则，以允许通过端口 80 的入站连接。

```cli
  az network nsg rule create \
    --resource-group myResourceGroupSLB \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRuleHTTP \
    --protocol tcp \
    --direction inbound \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 80 \
    --access allow \
    --priority 200
```
### <a name="create-nics"></a>创建 NIC

使用 [az network nic create](/cli/network/nic#az-network-nic-create) 创建三个网络接口，并将它们与公共 IP 地址和网络安全组关联。 

```cli
for i in `seq 1 2`; do
  az network nic create \
    --resource-group myResourceGroupSLB \
    --name myNic$i \
    --vnet-name myVnet \
    --subnet mySubnet \
    --network-security-group myNetworkSecurityGroup \
    --lb-name myLoadBalancer \
    --lb-address-pools myBackEndPool
done
```


## <a name="create-backend-servers"></a>创建后端服务器

本示例将创建三个要用作负载均衡器后端服务器的虚拟机。 若要验证负载均衡器是否已成功创建，还需要在虚拟机上安装 NGINX。

### <a name="create-an-availability-set"></a>创建可用性集

使用 [az vm availabilityset create](/cli/network/nic) 创建可用性集

 ```cli
  az vm availability-set create \
    --resource-group myResourceGroupSLB \
    --name myAvailabilitySet
```

### <a name="create-two-virtual-machines"></a>创建两个虚拟机

可使用 cloud-init 配置文件在 Linux 虚拟机上安装 NGINX 并运行“Hello World”Node.js 应用。 在当前 shell 中创建名为“cloud-init.txt”的文件，并将以下配置复制粘贴到 shell。 请确保正确复制整个 cloud-init 文件，尤其是第一行：

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
``` 
 
使用 [az vm create](/cli/vm#az-vm-create) 创建虚拟机。

 ```cli
for i in `seq 1 2`; do
  az vm create \
    --resource-group myResourceGroupSLB \
    --name myVM$i \
    --availability-set myAvailabilitySet \
    --nics myNic$i \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
    --no-wait
    done
```
VM 可能需要几分钟才能部署好。

## <a name="test-the-load-balancer"></a>测试负载均衡器

若要获取负载均衡器的公共 IP 地址，请使用 [az network public-ip show](/cli/network/public-ip#az-network-public-ip-show)。 复制该公共 IP 地址，并将其粘贴到浏览器的地址栏。

```cli
  az network public-ip show \
    --resource-group myResourceGroupSLB \
    --name myPublicIP \
    --query [ipAddress] \
    --output tsv
``` 
   ![测试负载均衡器](./media/load-balancer-standard-public-cli/running-nodejs-app.png)

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组、负载均衡器和所有相关的资源，可以使用 [az group delete](/cli/group#az-group-delete) 命令将其删除。

```cli 
  az group delete --name myResourceGroupSLB
```
