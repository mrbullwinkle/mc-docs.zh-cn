---
title: Azure Database for MariaDB 服务器防火墙规则
description: 介绍 Azure Database for MariaDB 服务器的防火墙规则。
author: WenJason
ms.author: v-jay
ms.service: mariadb
ms.topic: conceptual
origin.date: 09/24/2018
ms.date: 07/22/2019
ms.openlocfilehash: d2511908d97141f2580e6592f323642a8dea733b
ms.sourcegitcommit: 1dac7ad3194357472b9c0d554bf1362c391d1544
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2019
ms.locfileid: "68308926"
---
# <a name="azure-database-for-mariadb-server-firewall-rules"></a>Azure Database for MariaDB 服务器防火墙规则
在指定哪些计算机具有访问权限之前，防火墙将禁止所有对数据库服务器的访问。 防火墙基于每个请求的起始 IP 地址授予对服务器的访问权限。

要配置防火墙，请创建防火墙规则，指定可接受的 IP 地址的范围。 可以在服务器级别创建防火墙规则。

**防火墙规则：** 这些规则允许客户端访问整个 Azure Database for MariaDB 服务器，即同一逻辑服务器内的所有数据库。 可使用 Azure 门户或 Azure CLI 命令配置服务器级的防火墙规则。 若要创建服务器级防火墙规则，用户必须是订阅所有者或订阅参与者。

## <a name="firewall-overview"></a>防火墙概述
防火墙将默认阻止对 Azure Database for MariaDB 服务器的所有数据库访问。 若要从另一台计算机开始使用服务器，需要指定一个或多个服务器级防火墙规则以允许访问服务器。 使用防火墙规则指定要允许的来自 Internet 的 IP 地址范围。 对 Azure 门户网站本身的访问不受防火墙规则影响。

来自 Internet 和 Azure 的连接尝试必须首先通过防火墙，然后才能访问 Azure Database for MariaDB 数据库，如下图中所示：

![防火墙工作流示例](./media/concepts-firewall-rules/1-firewall-concept.png)

## <a name="connecting-from-the-internet"></a>从 Internet 连接
服务器级防火墙规则适用于 Azure Database for MariaDB 服务器上的所有数据库。

如果该请求的 IP 地址位于服务器级防火墙规则中指定的某个范围内，则授予连接权限。

如果该请求的 IP 地址位于任何数据库级或服务器级防火墙规则中指定的范围外，则连接请求失败。

## <a name="connecting-from-azure"></a>从 Azure 连接
若要允许来自 Azure 的应用程序连接到 Azure Database for MariaDB 服务器，必须启用 Azure 连接。 例如，为了托管“Azure Web 应用”应用程序或 Azure VM 中运行的应用程序，或者为了从 Azure 数据工厂数据管理网关进行连接。 资源无需在同一虚拟网络 (VNet) 或资源组中，即可使用防火墙规则启用这些连接。 在应用程序尝试从 Azure 连接到你的数据库服务器时，防火墙将验证是否允许 Azure 连接。 有几种方法可启用这些类型的连接。 如果防火墙设置的开始地址和结束地址都等于 0.0.0.0，则表示允许这些连接。 或者，可以在门户中从“连接安全性”  窗格将“允许访问 Azure 服务”  选项设为“启用”  并点击“保存”  。 如果不允许连接尝试，则请求将不会到达 Azure Database for MariaDB 服务器。

> [!IMPORTANT]
> 该选项将防火墙配置为允许来自 Azure 的所有连接，包括来自其他客户的订阅的连接。 选择该选项时，请确保登录名和用户权限将访问权限限制为仅已授权用户使用。
> 

![在门户中配置“允许访问 Azure 服务”](./media/concepts-firewall-rules/allow-azure-services.png)

## <a name="programmatically-managing-firewall-rules"></a>以编程方式管理防火墙规则
除了 Azure 门户外，还可使用 Azure CLI 通过编程方式管理防火墙规则。 

<!--See also [Create and manage Azure Database for MariaDB firewall rules using Azure CLI](./howto-manage-firewall-using-cli.md)-->

## <a name="troubleshooting-the-database-firewall"></a>数据库防火墙故障排除
对 Azure Database for MariaDB 服务器服务的访问未按预期工作时，请考虑以下几点：

* **对允许列表的更改尚未生效：** 对 Azure Database for MariaDB 防火墙配置所做的更改可能最多需要 5 分钟的延迟才可生效。

* **登录名未授权或使用了错误的密码：** 如果某个登录名不具备对 Azure Database for MariaDB 服务器的权限或者使用的密码不正确，则与 Azure Database for MariaDB 服务器的连接会被拒绝。 创建防火墙设置仅向客户端提供尝试连接到服务器的机会；每个客户端必须提供必需的安全凭据。

* **动态 IP 地址：** 如果 Internet 连接使用动态 IP 寻址，并且在通过防火墙时遇到问题，则可以尝试以下解决方法之一：

* 向 Internet 服务提供商 (ISP) 询问分配给客户端计算机、用于访问 Azure Database for MariaDB 服务器的 IP 地址范围，然后将该 IP 地址范围作为防火墙规则添加。

* 改为获取用户的客户端计算机的静态 IP 地址，并将该 IP 地址作为防火墙规则添加。

## <a name="next-steps"></a>后续步骤
- [使用 Azure 门户创建和管理 Azure Database for MariaDB 防火墙规则](./howto-manage-firewall-portal.md)

<!--
- [Create and manage Azure Database for MariaDB firewall rules using Azure CLI](./howto-manage-firewall-using-cli.md) -->
