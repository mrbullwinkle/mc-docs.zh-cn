---
title: 管理 Azure 流量管理器配置文件 | Azure
description: 本文有助于创建、禁用、启用和删除 Azure 流量管理器配置文件。
services: traffic-manager
documentationcenter: ''
author: rockboyfor
ms.service: traffic-manager
manager: digimobile
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 05/10/2017
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: 2384c9387ac1c0b8e1955a9bedc71cff6272f9cc
ms.sourcegitcommit: 021dbf0003a25310a4c8582a998c17729f78ce42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68514291"
---
# <a name="manage-an-azure-traffic-manager-profile"></a>管理 Azure 流量管理器配置文件

流量管理器配置文件使用流量路由方法控制云服务或网站终结点的流量分布。 本文介绍如何创建和管理这些配置文件。

## <a name="create-a-traffic-manager-profile"></a>创建流量管理器配置文件

可以使用 Azure 门户创建流量管理器配置文件。 在创建配置文件后，可以在 Azure 门户中配置终结点、监视以及其他设置。 流量管理器支持每个配置文件最多 200 个终结点。 但是，大多数使用方案只需要少量的终结点。

### <a name="to-create-a-traffic-manager-profile"></a>创建流量管理器配置文件

1. 在浏览器中，登录 [Azure 门户](https://portal.azure.cn)。 如果还没有帐户，可注册 [1 个月期限的试用版](https://www.azure.cn/pricing/1rmb-trial/)。 
2. 单击“创建资源” > “网络” > “流量管理器配置文件” > “创建”     。
4. 在“创建流量管理器配置文件”  中，按如下所示完成操作：
    1. 在**名称**中，提供配置文件的名称。 此名称必须在 trafficmanager.cn 区域中唯一，并会生成 DNS 名称 (`<name>`,trafficmanager.cn)，该名称用于访问流量管理器配置文件。
    2. 在**路由方法**中，选择“优先级”  路由方法。
    3. 在**订阅**中，选择要创建此配置文件的订阅
    4. 在**资源组**中，创建新的资源组，以在其下放置此配置文件。
    5. 在**资源组位置**中，选择资源组的位置。 此设置指的是资源组的位置，对将全局部署的流量管理器配置文件没有影响。
    6. 单击**创建**。
    7. 流量管理器配置文件的全局部署完成后，它会在相应的资源组中作为资源之一列出。

## <a name="disable-enable-or-delete-a-profile"></a>禁用、启用或删除配置文件

可以禁用现有的配置文件，使流量管理器不会将用户请求路由到配置的终结点。 禁用流量管理器配置文件后，配置文件及其中包含的信息将保持不变，并且可以在流量管理器界面中进行编辑。  重新启用配置文件时，路由会恢复。 在 Azure 门户中创建流量管理器配置文件时，会自动启用该配置文件。 如果确定不再需要某个配置文件，可以将其删除。

### <a name="to-disable-a-profile"></a>禁用配置文件

1. 如果使用自定义域名，请更改 Internet DNS 服务器上的 CNAME 记录，使它不再指向流量管理器配置文件。
2. 流量不再通过流量管理器配置文件设置定向到终结点。
3. 在浏览器中，登录 [Azure 门户](https://portal.azure.cn)。
2. 在门户的搜索栏中，搜索要修改的流量管理器配置文件  名称，并在显示的结果中单击该流量管理器配置文件。
3. 单击“概览”   >   “禁用”。
4. 确认禁用流量管理器配置文件。

### <a name="to-enable-a-profile"></a>启用配置文件

1. 在浏览器中，登录 [Azure 门户](https://portal.azure.cn)。
2. 在门户的搜索栏中，搜索要修改的流量管理器配置文件  名称，并在显示的结果中单击该流量管理器配置文件。
3. 单击“概览”   >   “启用”。
1. 如果使用自定义域名，请在 Internet DNS 服务器上创建一条指向流量管理器配置文件域名的 CNAME 资源记录。
2. 然后，流量将再次定向到终结点。

### <a name="to-delete-a-profile"></a>删除配置文件

1. 确保 Internet DNS 服务器上的 DNS 资源记录不再使用指向流量管理器配置文件域名的 CNAME 资源记录。
2. 在门户的搜索栏中，搜索要修改的流量管理器配置文件  名称，并在显示的结果中单击该流量管理器配置文件。
3. 单击“概览”   >   “删除”。
4. 确认删除流量管理器配置文件。

## <a name="next-steps"></a>后续步骤

* [添加终结点](traffic-manager-endpoints.md)
* [配置优先级路由方法](traffic-manager-configure-priority-routing-method.md)
* [配置地理路由方法](traffic-manager-configure-geographic-routing-method.md) 
* [配置加权路由方法](traffic-manager-configure-weighted-routing-method.md)
* [配置性能路由方法](traffic-manager-configure-performance-routing-method.md)

<!-- Update_Description: wording meta properties -->