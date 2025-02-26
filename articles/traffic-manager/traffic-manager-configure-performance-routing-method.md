---
title: 使用 Azure 流量管理器配置性能流量路由方法 | Azure
description: 本文介绍了如何配置流量管理器以将流量路由到终结点并保证最低延迟
services: traffic-manager
manager: digimobile
documentationcenter: ''
author: rockboyfor
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 03/20/2017
ms.date: 07/22/2019
ms.author: v-yeche
ms.openlocfilehash: b2dcd6dff154b2b02fc6c1b95498200ce06fdb46
ms.sourcegitcommit: 021dbf0003a25310a4c8582a998c17729f78ce42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68514187"
---
# <a name="configure-the-performance-traffic-routing-method"></a>配置性能流量路由方法

使用性能流量路由方法可将流量定向到客户端网络中延迟最低的终结点。 通常，延迟最低的数据中心就是地理距离最短的终结点。 此流量路由方法无法考虑网络配置或负载的实时变化。

## <a name="to-configure-performance-routing-method"></a>配置性能路由方法

1. 在浏览器中，登录 [Azure 门户](https://portal.azure.cn)。 如果还没有帐户，可注册 [1 个月期限的试用版](https://www.azure.cn/pricing/1rmb-trial/)。 
2. 在门户的搜索栏中，搜索“流量管理器配置文件”  ，并单击要为其配置路由方法的配置文件名称。
3. 在“流量管理器配置文件”  边栏选项卡中，检查要包含在配置中的云服务和网站是否都存在。
4. 在“设置”  部分，单击“配置”  ，并在“配置”  边栏选项卡中完成如下操作：
    1. 对于“流量路由方法设置”  和“路由方法”  ，请选择“性能”  。
    2. 为此配置文件中的所有终结点设置相同的“终结点监视器设置”  ，如下所示：
        1. 选择相应的“协议”  ，并指定“端口”  号。 
        2. 对于“路径”  ，请键入正斜杠 */* 。 若要监视终结点，必须指定路径和文件名。 正斜杠“/”是有效的相对路径条目，表示文件位于根目录（默认位置）中。
        3. 在页面顶部，单击“保存”  。
5. 按如下方式测试配置更改：
    1. 在门户的搜索栏中，搜索流量管理器配置文件名称，并在显示的结果中单击该流量管理器配置文件。
    2. 在“流量管理器配置文件”  边栏选项卡中，单击“概述”  。
    3. “流量管理器配置文件”  边栏选项卡会显示新建的流量管理器配置文件的 DNS 名称。 任意客户端（例如通过 Web 浏览器导航到此名称）均可使用此名称路由到根据路由类型确定的相应终结点。 在此情况下，所有请求均通过客户端网络路由到终结点并保证最低延迟。
6. 流量管理器配置文件正常工作后，请在权威 DNS 服务器上编辑 DNS 记录，将公司域名指向流量管理器域名。

![使用流量管理器配置性能流量路由方法][1]

## <a name="next-steps"></a>后续步骤

- 了解[加权流量路由方法](traffic-manager-configure-weighted-routing-method.md)。
- 了解[优先级路由方法](traffic-manager-configure-priority-routing-method.md)。
- 了解[地理路由方法](traffic-manager-configure-geographic-routing-method.md)。
- 了解如何[测试流量管理器设置](traffic-manager-testing-settings.md)。

<!--Image references-->
[1]: ./media/traffic-manager-performance-routing-method/traffic-manager-performance-routing-method.png

<!-- Update_Description: wording update, update link -->