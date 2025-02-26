---
title: 本地数据网关
description: 如果 Azure 中的 Analysis Services 服务器要连接到本地数据源，则本地网关是必需的。
author: rockboyfor
manager: digimobile
ms.service: azure-analysis-services
ms.topic: conceptual
origin.date: 04/30/2019
ms.date: 07/22/2019
ms.author: v-yeche
ms.reviewer: minewiskan
ms.openlocfilehash: e1fe29e069edb94776b7461c3ca600b98fdd4733
ms.sourcegitcommit: e84b0fe3c1b2a6c9551084b6b27740c648b460ae
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2019
ms.locfileid: "68308846"
---
# <a name="connecting-to-on-premises-data-sources-with-on-premises-data-gateway"></a>使用本地数据网关连接到本地数据源
本地数据网关提供本地数据源与云中的 Azure Analysis Services 服务器之间的安全数据传输。 除了适用于同一区域中的多个 Azure Analysis Services 服务器以外，最新版本的网关还适用于 Azure 逻辑应用、Power BI、Power Apps 和 Azure Flow。 可将同一订阅和同一区域中的多个服务与单个网关相关联。 

首次安装网关的过程由四个部分组成：

- **下载并运行安装程序** - 此步骤在组织中的计算机上安装网关服务。 还需要使用你的租户的 Azure AD 中的一个帐户登录到 Azure。 不支持 Azure B2B（来宾）帐户。
    
    <!--MOONCAKE: Not Available on [tenant's](https://docs.microsoft.com/zh-cn/previous-versions/azure/azure-services/jj573650(v=azure.100)#BKMK_WhatIsAnAzureADTenant)-->

- **注册网关** - 在这一步中，指定网关的名称和恢复密钥，然后选择区域，在网关云服务中注册你的网关。 网关资源可以注册在任何区域中，但我们建议处于与 Analysis Services 服务器位于同一区域。 

- **在 Azure 中创建网关资源** - 此步骤在 Azure 订阅中创建网关资源。

- **将服务器连接到网关资源** - 在订阅中创建网关资源后，可以开始将服务器连接到该资源。 可以连接多个服务器和其他资源，前提是它们位于同一订阅和同一区域中。

若要立即开始，请参阅[安装并配置本地数据网关](analysis-services-gateway-install.md)。

## <a name="how-it-works"></a>工作原理
在你组织中的计算机上安装的网关作为 Windows 服务（本地数据网关）  运行。 此本地服务已通过 Azure 服务总线注册到网关云服务。 然后，可为 Azure 订阅创建网关资源网关云服务。 Azure Analysis Services 服务器随后会连接到网关资源。 如果服务器上的模型需要连接到本地数据源以执行查询或处理，查询和数据流会遍历网关资源、Azure 服务总线、本地数据网关服务和数据源。 

![工作原理](./media/analysis-services-gateway/aas-gateway-how-it-works.png)

查询和数据流：

1. 查询是云服务使用本地数据源的加密凭据创建的。 查询随后会发送到队列让网关处理。
2. 网关云服务分析该查询，并将请求推送到 [Azure 服务总线](/service-bus/)。
3. 本地数据网关会针对挂起的请求轮询 Azure 服务总线。
4. 网关获取查询，对凭据进行解密，并使用这些凭据连接到数据源。
5. 网关将查询发送到数据源以便执行。
6. 结果会从数据源返回到网关，并返回到云服务和服务器。

## <a name="windows-service-account"> </a>Windows 服务帐户
本地数据网关配置为将 *NT SERVICE\PBIEgwService* 用于 Windows 服务登录凭据。 默认情况下，它有权作为服务登录；这位于正在安装网关的计算机的上下文中。 此凭据与用于连接到本地数据源或 Azure 帐户的帐户不相同。  

如果由于身份验证，代理服务器遇到问题，建议将 Windows 服务帐户更改为域用户或托管服务帐户。

## <a name="ports"> </a>端口
网关会创建与 Azure 服务总线之间的出站连接。 它在以下出站端口上进行通信：TCP 443（默认值）、5671、5672、9350 到 9354。 网关不需要入站端口。

我们建议在防火墙中将数据区域的 IP 地址列入允许列表。 可以下载 [Azure 数据中心 IP 列表](https://www.microsoft.com/download/confirmation.aspx?id=57062)。 该列表每周都会进行更新。

> [!NOTE]
> Azure 数据中心 IP 列表中列出的 IP 地址使用的是 CIDR 表示法。 若要了解详细信息，请参阅 [Classless Inter-Domain Routing](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)（无类别域际路由）。
>
>

以下是该网关所用的完全限定域名。

| 域名 | 出站端口 | 说明 |
| --- | --- | --- |
| *.powerbi.cn |80 |用于下载安装程序的 HTTP。 |
| *.powerbi.cn |443 |HTTPS |
| *.analysis.chinacloudapi.cn |443 |HTTPS |
| *.login.chinacloudapi.cn |443 |HTTPS |
| *.servicebus.chinacloudapi.cn |5671-5672 |高级消息队列协议 (AMQP) |
| *.servicebus.chinacloudapi.cn |443, 9350-9354 |通过 TCP 的服务总线中继上的侦听器（需要 443 来获取访问控制令牌） |
| *.frontend.clouddatahub.net |443 |HTTPS |
| *.core.chinacloudapi.cn |443 |HTTPS |
| login.chinacloudapi.cn |443 |HTTPS |
| *.msftncsi.com |443 |在 Power BI 服务无法访问网关时用于测试 Internet 连接。 |
| *.microsoftonline-p.com |443 |用于根据配置进行身份验证。 |

<a name="force-https"></a>
### <a name="forcing-https-communication-with-azure-service-bus"></a>强制与 Azure 服务总线进行 HTTPS 通信
可以强制网关使用 HTTPS 而非直接 TCP 与 Azure 服务总线进行通信，但此操作可能会显著降低性能。 若要修改 *Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config* 文件，可将值从 `AutoDetect` 更改为 `Https`。 此文件通常位于 *C:\Program Files\On-premises data gateway*。

```
<setting name="ServiceBusSystemConnectivityModeString" serializeAs="String">
    <value>Https</value>
</setting>
```

## <a name="tenant-level-administration"></a>租户级管理 

目前没有单独的位置可让租户管理员管理其他用户安装和配置的所有网关。  如果你是租户管理员，建议你要求组织中的用户将你作为管理员添加到他们安装的每个网关。 这样，即可通过“网关设置”页面或通过 [PowerShell 命令](https://docs.microsoft.com/zh-cn/power-bi/service-gateway-high-availability-clusters#powershell-support-for-gateway-clusters)管理组织中的所有网关。 

<a name="faq"></a>
## <a name="frequently-asked-questions"></a>常见问题

### <a name="general"></a>常规

**问**：对于云中的数据源（如 Azure SQL 数据库），是否需要网关？ <br/>
**答**：否。 如果仅连接到本地数据源，则网关是必需的。

**问**：网关是否必须安装在与数据源相同的计算机上？ <br/>
**答**：否。 网关只需能够连接到服务器即可，通常在同一个网络上。

<a name="why-azure-work-school-account"></a>

**问**：为何需要使用工作或学校帐户登录？ <br/>
**答**：安装本地数据网关时只能使用组织的工作或学校帐户。 而且，该帐户必须与要在其中配置网关资源的订阅在同一租户中。 登录帐户存储在由 Azure Active Directory (Azure AD) 管理的租户中。 通常，Azure AD 帐户的用户主体名称 (UPN) 与电子邮件地址匹配。

**问**：凭据存储在何处？ <br/>
**答**：为数据源输入的凭据会加密，并存储在网关云服务中。 凭据在本地数据网关中解密。

**问**：对于网络带宽是否有要求？ <br/>
**答**：建议为网络连接配置较高的吞吐量。 每个环境是不同的，所发送的数据量会影响效果。 使用 ExpressRoute 可以帮助保证本地与 Azure 数据中心之间的吞吐量级别。

<!--MOONCAKE Not Available on You can use the third-party tool Azure Speed Test app to help gauge your throughput.-->

**问**：从网关运行对数据源的查询时的延迟是多少？ 最佳体系结构是什么？ <br/>
**答**：若要降低网络延迟，请将网关安装在尽可能靠近数据源的位置。 如果可以在实际数据源上安装网关，这种距离可最大程度降低造成的延迟。 还需考虑数据中心。 例如，如果服务使用中国北部的数据中心，而你在 Azure VM 中托管 SQL Server，则 Azure VM 也应该位于中国北部。 这种距离可最大程度降低延迟并避免 Azure VM 产生传出费用。

**问**：如何将结果发送回云？ <br/>
**答**：可通过 Azure 服务总线发送结果。

**问**：是否存在从云到网关的入站连接？ <br/>
**答**：否。 网关使用与 Azure 服务总线之间的出站连接。

**问**：如果阻止出站连接，会发生什么情况？ 我需要打开什么？ <br/>
**答**：查看网关使用的端口和主机。

**问**：实际 Windows 服务的名称是什么？<br/>
**答**：在“服务”中，网关名为“本地数据网关服务”。

**问**：是否可以使用 Azure Active Directory 帐户运行网关 Windows 服务？ <br/>
**答**：否。 该 Windows 服务必须具有有效的 Windows 帐户。 默认情况下，服务使用服务 SID“NT SERVICE\PBIEgwService”来运行。

**问**：如何接管网关？ <br/>
**答**：若要接管网关（通过在“控制面板”>“程序”中运行“安装程序”/“更改”），你需要成为 Azure 中网关资源的所有者并且需要有恢复密钥。 可在访问控制中配置网管资源所有者。

<a name="high-availability"></a>
### <a name="high-availability-and-disaster-recovery"></a>高可用性和灾难恢复

**问**：如何具有更高可用性？  
**答**：可在另一台计算机上安装网关以创建群集。 若要了解详细信息，请参阅 Power BI Gateway 文档中的[本地数据网关的高可用性群集](https://docs.microsoft.com/zh-cn/power-bi/service-gateway-high-availability-clusters)。

**问**：有哪些选项可用于灾难恢复？ <br/>
**答**：可以使用恢复密钥还原或移动网关。 安装网关时，指定恢复密钥。

**问**：恢复密钥的好处是什么？ <br/>
**答**：使用恢复密钥可在发生灾难后迁移或恢复网关设置。

## <a name="troubleshooting"></a>故障排除

**问**：尝试在 Azure 中创建网关资源时，为什么在网关实例列表中看不到我的网关？ <br/>
**答**：有两种可能的原因。 第一种可能性是已在当前或某个其他订阅中为该网关创建了资源。 若要消除该可能性，请从门户枚举“本地数据网关”  类型的资源。 枚举所有资源时，请确保选择所有订阅。 创建资源后，该网关将不会出现在“创建网关资源”门户体验的网关实例列表中。 第二种可能性是安装该网关的用户的 Azure AD 标识与登录到 Azure 门户的用户不同。 若要解决此问题，请使用安装该网关的用户帐户登录到门户。

**问**：如何查看正在发送到本地数据源的查询？ <br/>
**答**：可以启用查询跟踪（包括已发送的查询）。 请记得在完成故障排除后将查询跟踪改回原始值。 一直保持启用查询跟踪会创建大量的日志。

还可以查看数据源用于跟踪查询的工具。 例如，可以使用 SQL Server 的扩展事件或 SQL 事件探查器以及 Analysis Services。

**问**：网关日志在何处？ <br/>
**答**：请参阅本文中后面的“日志”。

<a name="update"></a>
### <a name="update-to-the-latest-version"></a>更新到最新版本

如果网关版本过时，可能会出现很多问题。 良好的常规做法是确保使用最新版本。 如果有一个月或更长时间未更新网关，可能要考虑安装最新版本的网关，并确定是否可以重现问题。

### <a name="error-failed-to-add-user-to-group--2147463168-pbiegwservice-performance-log-users"></a>错误：无法将用户添加到组。 (-2147463168 PBIEgwService Performance Log Users)

如果尝试在不受支持的域控制器上安装网关，可能会收到此错误。 请确保将网关部署在不是域控制器的计算机上。

<a name="logs"></a>
## <a name="logs"></a>日志

进行故障排除时，日志文件是一项重要资源。

#### <a name="enterprise-gateway-service-logs"></a>企业网关服务日志

`C:\Users\PBIEgwService\AppData\Local\Microsoft\On-premises data gateway\<yyyyymmdd>.<Number>.log`

#### <a name="configuration-logs"></a>配置日志

`C:\Users\<username>\AppData\Local\Microsoft\On-premises data gateway\GatewayConfigurator.log`

#### <a name="event-logs"></a>事件日志

可在“应用程序和服务日志”下找到数据管理网关和 PowerBIGateway 日志。 

## <a name="next-steps"></a>后续步骤
* [安装并配置本地数据网关](analysis-services-gateway-install.md)。   
* [管理 Analysis Services](analysis-services-manage.md)
* [从 Azure Analysis Services 获取数据](analysis-services-connect.md)

<!-- Update_Description: update meta properties, wording update -->
