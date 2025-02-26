---
title: 将经典虚拟网络连接到 Azure 资源管理器 VNet：PowerShell | Microsoft Docs
description: 使用 VPN 网关和 PowerShell 在经典 VNet 与资源管理器 VNet 之间创建 VPN 连接。
services: vpn-gateway
author: WenJason
ms.service: vpn-gateway
ms.topic: conceptual
origin.date: 10/17/2018
ms.date: 03/04/2019
ms.author: v-jay
ms.openlocfilehash: 9437bc6bc8924f3560e1599ed82a9348b2bee034
ms.sourcegitcommit: 5fc46672ae90b6598130069f10efeeb634e9a5af
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2019
ms.locfileid: "67236488"
---
# <a name="connect-virtual-networks-from-different-deployment-models-using-powershell"></a>使用 PowerShell 从不同的部署模型连接虚拟网络

本文可帮助你将经典 VNet 连接到资源管理器 VNet，以使位于单独部署模型中的资源能够相互通信。 本文中的步骤使用 PowerShell 完成，但也可通过从此列表中选择文章使用 Azure 门户来创建此配置。

> [!div class="op_single_selector"]
> * [门户](vpn-gateway-connect-different-deployment-models-portal.md)
> * [PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
> 
> 

将经典 VNet 连接到资源管理器 VNet 类似于将 VNet 连接到本地站点位置。 这两种连接类型都使用 VPN 网关来提供使用 IPsec/IKE 的安全隧道。 可以在区域中的 VNet 之间创建连接。 还可以连接已连接到本地网络的 VNet，只要它们配置的网关是动态或基于路由的。 有关 VNet 到 VNet 连接的详细信息，请参阅本文末尾的 [VNet 到 VNet 常见问题解答](#faq) 。 

如果还没有虚拟网络网关并且不想创建一个，建议你改为考虑使用 VNet 对等互连连接 VNet。 VNet 对等互连不使用 VPN 网关。 有关详细信息，请参阅 [VNet 对等互连](../virtual-network/virtual-network-peering-overview.md)。

## <a name="before"></a>准备工作

以下步骤指导您完成为每个 VNet 配置动态或基于路由的网关以及在网关之间创建 VPN 连接所需的设置。 此配置不支持静态或基于策略的网关。

### <a name="pre"></a>先决条件

* 已创建了两个 VNet。 如果需要创建资源管理器虚拟网络，请参阅[创建资源组和虚拟网络](../virtual-network/quick-create-powershell.md#create-a-resource-group-and-a-virtual-network)。 若要创建经典虚拟网络，请参阅[创建经典 VNet](/virtual-network/create-virtual-network-classic)。
* 两个 VNet 的地址范围不相互重叠，也不与网关可能连接到的其他连接的任何范围重叠。
* 已安装最新的 PowerShell cmdlet。 有关详细信息，请参阅 [如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview) 。 请确保安装服务管理 (SM) 和 Resource Manager (RM) cmdlet。 

### <a name="exampleref"></a>示例设置

可使用这些值创建测试环境，或参考这些值以更好地理解本文中的示例。

**经典 VNet 设置**

VNet 名称 = ClassicVNet <br>
位置 = 中国北部 <br>
虚拟网络地址空间 = 10.0.0.0/24 <br>
子网 1 = 10.0.0.0/27 <br>
GatewaySubnet = 10.0.0.32/29 <br>
本地网络名称 = RMVNetLocal <br>
网关类型 = DynamicRouting

**Resource Manager VNet 设置**

VNet 名称 = RMVNet <br>
资源组 = RG1 <br>
虚拟网络 IP 地址空间 = 192.168.0.0/16 <br>
子网 1 = 192.168.1.0/24 <br>
GatewaySubnet = 192.168.0.0/26 <br>
位置 = 中国北部 <br>
网关公共 IP 名称 = gwpip <br>
本地网关 = ClassicVNetLocal <br>
虚拟网关名称 = RMGateway <br>
网关 IP 寻址配置 = gwipconfig

## <a name="createsmgw"></a>第 1 节 — 配置经典 VNet
### <a name="1-download-your-network-configuration-file"></a>1.下载网络配置文件
1. 在 PowerShell 控制台中，使用提升的权限登录到 Azure 帐户。 以下 cmdlet 会提示您提供 Azure 帐户的登录凭据。 登录后它会下载帐户设置，以便这些信息可供 Azure PowerShell 使用。 在本部分中使用经典服务管理 (SM) Azure PowerShell cmdlet。

   ```powershell
   Add-AzureAccount -Environment AzureChinaCloud
   ```

   获取 Azure 订阅。

   ```powershell
   Get-AzureSubscription
   ```

   如果有多个订阅，请选择要使用的订阅。

   ```powershell
   Select-AzureSubscription -SubscriptionName "Name of subscription"
   ```
2. 通过运行以下命令，导出 Azure 网络配置文件。 如有必要，可以将文件的导出位置更改为其他位置。

   ```powershell
   Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
   ```
3. 打开下载的 .xml 文件进行编辑。 有关网络配置文件的示例，请参阅 [Network Configuration Schema](https://msdn.microsoft.com/library/jj157100.aspx)（网络配置架构）。

### <a name="2-verify-the-gateway-subnet"></a>2.验证网关子网
在 **VirtualNetworkSites** 元素中，向 VNet 添加一个网关子网（如果尚未创建）。 在使用网络配置文件时，网关子网必须命名为“GatewaySubnet”，否则 Azure 无法识别并将其用作网关子网。

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

**示例：**

    <VirtualNetworkSites>
      <VirtualNetworkSite name="ClassicVNet" Location="China North">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/24</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="Subnet-1">
            <AddressPrefix>10.0.0.0/27</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.0.0.32/29</AddressPrefix>
          </Subnet>
        </Subnets>
      </VirtualNetworkSite>
    </VirtualNetworkSites>

### <a name="3-add-the-local-network-site"></a>3.添加本地网络站点
所添加的本地网络站点表示要连接到的 RM VNet。 如果文件中尚不存在 **LocalNetworkSites** 元素，请进行添加。 此时，在配置中，VPNGatewayAddress 可以是任何有效的公共 IP 地址，因为我们尚未针对 Resource Manager VNet 创建网关。 一旦创建网关，便会将此占位符 IP 地址替换为已分配给 RM 网关的正确公共 IP 地址。

    <LocalNetworkSites>
      <LocalNetworkSite name="RMVNetLocal">
        <AddressSpace>
          <AddressPrefix>192.168.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>

### <a name="4-associate-the-vnet-with-the-local-network-site"></a>4.将 VNet 与本地网络站点关联
在此部分中，我们将指定您要将 VNet 连接到的本地网络站点。 在本例中，该站点即前面提到的 Resource Manager VNet。 请确保名称相匹配。 此步骤不会创建网关。 它指定网关要连接到的本地网络。

        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="RMVNetLocal">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>

### <a name="5-save-the-file-and-upload"></a>5.保存文件并上传
保存文件，并运行以下命令以将其导入到 Azure。 确保根据环境需要更改文件路径。

```powershell
Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml
```

表明导入成功的类似结果随即就会显示。

        OperationDescription        OperationId                      OperationStatus                                                
        --------------------        -----------                      ---------------                                                
        Set-AzureVNetConfig        e0ee6e66-9167-cfa7-a746-7casb9    Succeeded 

### <a name="6-create-the-gateway"></a>6.创建网关

运行此示例之前，请参阅所下载的网络配置文件，了解 Azure 所需要的确切名称。 网络配置文件包含经典虚拟网络的值。 在 Azure 门户中创建经典 VNet 设置时，由于部署模型的不同，有时经典 VNet 的名称在网络配置文件中会发生变化。 例如，如果使用 Azure 门户创建一个名为“Classic VNet”的经典 VNet，并在名为“ClassicRG”的资源组中创建它，则网络配置文件中的名称将变为“Group ClassicRG Classic VNet”。 指定包含空格的 VNet 的名称时，请使用引号将值引起来。


使用以下示例创建动态路由网关：

```powershell
New-AzureVNetGateway -VNetName ClassicVNet -GatewayType DynamicRouting
```

可以使用 **Get-AzureVNetGateway** cmdlet 检查网关状态。

## <a name="creatermgw"></a>第 2 节 - 配置 RM VNet 网关

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

先决条件是假设你已创建了一个 RM VNet。 在此步骤中，你将为 RM VNet 创建一个 VPN 网关。 请务必在检索到经典 VNet 的网关的公共 IP 地址之后再开始执行以下步骤。 

1. 在 PowerShell 控制台中登录到 Azure 帐户。 以下 cmdlet 会提示您提供 Azure 帐户的登录凭据。 登录后将下载帐户设置，以便 Azure PowerShell 使用这些设置。

   ```powershell
   Connect-AzAccount -Environment AzureChinaCloud
   ``` 
   若要验证是否在使用正确的订阅，请运行以下 cmdlet：  

   ```powershell
   Get-AzSubscription
   ```
   
   如果有多个订阅，请指定要使用的订阅。

   ```powershell
   Select-AzSubscription -SubscriptionName "Name of subscription"
   ```
2. 创建本地网关。 在虚拟网络中，局域网网关通常指本地位置。 在本例中，本地网关是指经典 VNet。 指定该网关的名称以供 Azure 引用，同时指定地址空间前缀。 Azure 使用指定的 IP 地址前缀来识别要发送到本地位置的流量。 如果稍后需要在创建网关之前调整此处的信息，可以修改这些值并再次运行该示例。
   
   **-Name** 是要分配的名称，表示本地网关。<br>
   **-AddressPrefix** 是经典 VNet 的地址空间。<br>
   **-GatewayIpAddress** 是经典 VNet 网关的公共 IP 地址。 请务必更改下面的示例文本“n.n.n.n”以反映正确的 IP 地址。<br>

   ```powershell
   New-AzLocalNetworkGateway -Name ClassicVNetLocal `
   -Location "ChinaNorth" -AddressPrefix "10.0.0.0/24" `
   -GatewayIpAddress "n.n.n.n" -ResourceGroupName RG1
   ```
3. 请求一个公共 IP 地址并将其分配到 Resource Manager VNet 的虚拟网关。 无法指定要使用的 IP 地址。 IP 地址动态分配到虚拟网关。 但是，这并不意味着 IP 地址会更改。 虚拟网关 IP 地址只在删除和重新创建网关时更改。 该地址不会因为网关大小调整、重置或其他内部维护/升级而更改。

   本步骤还会设置一个要在后续步骤中使用的变量。

   ```powershell
   $ipaddress = New-AzPublicIpAddress -Name gwpip `
   -ResourceGroupName RG1 -Location 'ChinaNorth' `
   -AllocationMethod Dynamic
   ```

4. 验证虚拟网络是否包含网关子网。 如果不存在任何网关子网，则添加一个。 请确保网关子网命名为 *GatewaySubnet*。
5. 通过运行以下命令，检索用于网关的子网。 在此步骤中，我们还会设置一个要在下一步使用的变量。
   
   **-Name** 是 Resource Manager VNet 的名称。<br>
   **-ResourceGroupName** 是 VNet 所关联的资源组。 此 VNet 必须已经存在网关子网，并且该子网必须命名为 *GatewaySubnet* 才能正常工作。<br>

   ```powershell
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name GatewaySubnet `
   -VirtualNetwork (Get-AzVirtualNetwork -Name RMVNet -ResourceGroupName RG1)
   ``` 

6. 创建网关 IP 寻址配置。 网关配置定义要使用的子网和公共 IP 地址。 使用以下示例创建网关配置。

   在本步骤中， **-SubnetId** 和 **-PublicIpAddressId** 参数必须分别从子网和 IP 地址对象传递 ID 属性。 不能使用简单字符串。 会在请求公共 IP 的步骤和检索子网的步骤中设置这些变量。

   ```powershell
   $gwipconfig = New-AzVirtualNetworkGatewayIpConfig `
   -Name gwipconfig -SubnetId $subnet.id `
   -PublicIpAddressId $ipaddress.id
   ```
7. 通过运行以下命令，创建 Resource Manager 虚拟网关。 `-VpnType` 必须是 *RouteBased*。 创建网关可能需要 45 分钟或更长时间。

   ```powershell
   New-AzVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1 `
   -Location "ChinaNorth" -GatewaySKU Standard -GatewayType Vpn `
   -IpConfigurations $gwipconfig `
   -EnableBgp $false -VpnType RouteBased
   ```
8. VPN 网关创建好后，复制公共 IP 地址。 为经典 VNet 配置本地网络设置时要使用该地址。 可以使用以下 cmdlet 来检索公共 IP 地址。 公共 IP 地址在返回结果中作为 *IpAddress*列出。

   ```powershell
   Get-AzPublicIpAddress -Name gwpip -ResourceGroupName RG1
   ```

## <a name="localsite"></a>第 3 节 - 修改经典 VNet 本地站点设置

本节涉及经典 VNet。 需替换在指定本地站点设置时使用的占位符 IP 地址，这些设置用于连接到 Resource Manager VNet 网关。 

1. 导出网络配置文件。

   ```powershell
   Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
   ```
2. 使用文本编辑器，修改 VPNGatewayAddress 的值。 将占位符 IP 地址替换为 Resource Manager 网关的公共 IP 地址，并保存所做的更改。

   ```
   <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
   ```
3. 将修改后的网络配置文件导入到 Azure。

   ```powershell
   Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml
   ```

## <a name="connect"></a>第 4 节 - 在网关之间创建连接
在网关之间创建连接需要用到 PowerShell。 可能需要添加 Azure 帐户才能使用经典版 PowerShell cmdlet。 为此，请使用 **Add-AzureAccount -Environment AzureChinaCloud**。

1. 在 PowerShell 控制台中设置共享密钥。 运行 cmdlet 之前，请参阅已下载的网络配置文件，了解 Azure 所需要的确切名称。 指定包含空格的 VNet 的名称时，请使用单引号将值引起来。<br><br>在以下示例中， **-VNetName** 是经典 VNet 的名称， **-LocalNetworkSiteName** 是为本地网络站点指定的名称。 **-SharedKey** 是生成并指定的值。 在示例中使用的是“abc123”，但可以生成和使用更复杂的。 重要的是，此处指定的值必须与下一步中创建连接时指定的值相同。 返回结果应显示“状态:  成功”。

   ```powershell
   Set-AzureVNetGatewayKey -VNetName ClassicVNet `
   -LocalNetworkSiteName RMVNetLocal -SharedKey abc123
   ```
2. 运行以下命令创建 VPN 连接：
   
   设置变量。

   ```powershell
   $vnet01gateway = Get-AzLocalNetworkGateway -Name ClassicVNetLocal -ResourceGroupName RG1
   $vnet02gateway = Get-AzVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1
   ```
   
   创建连接。 请注意， **-ConnectionType** 是 IPsec，而不是 Vnet2Vnet。

   ```powershell
   New-AzVirtualNetworkGatewayConnection -Name RM-Classic -ResourceGroupName RG1 `
   -Location "ChinaNorth" -VirtualNetworkGateway1 `
   $vnet02gateway -LocalNetworkGateway2 `
   $vnet01gateway -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
   ```

## <a name="verify"></a>第 5 节 - 验证连接

### <a name="to-verify-the-connection-from-your-classic-vnet-to-your-resource-manager-vnet"></a>验证从经典 VNet 到 Resource Manager VNet 的连接

#### <a name="powershell"></a>PowerShell

[!INCLUDE [vpn-gateway-verify-connection-ps-classic](../../includes/vpn-gateway-verify-connection-ps-classic-include.md)]

#### <a name="azure-portal"></a>Azure 门户

[!INCLUDE [vpn-gateway-verify-connection-azureportal-classic](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]


### <a name="to-verify-the-connection-from-your-resource-manager-vnet-to-your-classic-vnet"></a>验证从 Resource Manager VNet 到经典 VNet 的连接

#### <a name="powershell"></a>PowerShell

[!INCLUDE [vpn-gateway-verify-ps-rm](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

#### <a name="azure-portal"></a>Azure 门户

[!INCLUDE [vpn-gateway-verify-connection-portal-rm](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="faq"></a>VNet 到 VNet 常见问题解答

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]
<!--Update_Description: code update -->