---
title: 适用于服务提供商的 Azure Monitor | Azure Docs
description: Azure Monitor 可以帮助托管服务提供商 (MSP)、大型企业、独立软件供应商 (ISV) 和主机托管服务提供商管理和监视客户本地或云基础结构中的服务器。
services: log-analytics
documentationcenter: ''
author: lingliw
manager: digimobile
editor: ''
ms.assetid: c07f0b9f-ec37-480d-91ec-d9bcf6786464
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 04/12/2019
ms.author: v-lingwu
ms.openlocfilehash: c9523a41db7100ab5012d44d2df9a1032da8bbb1
ms.sourcegitcommit: e78670855b207c6084997f747ad8e8c3afa3518b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68513895"
---
# <a name="azure-monitor-for-service-providers"></a>适用于服务提供商的 Azure Monitor
Azure Monitor 中的 Log Analytics 工作区可以帮助托管服务提供商 (MSP)、大型企业、独立软件供应商 (ISV) 和主机托管服务提供商管理和监视客户本地或云基础结构中的服务器。 

大型企业与服务提供商有许多相似之处，特别是当有一个集中式 IT 团队负责管理许多不同业务部门的 IT 时。 为了简单起见，本文档使用术语*服务提供商*，但同样的功能也可用于企业和其他客户。


## <a name="architectures-for-service-providers"></a>适用于服务提供商的体系结构

借助 Log Analytics 工作区，管理员可控制日志流动和隔离，并创建满足其特定业务需求的日志体系结构。 [本文](/log-analytics/log-analytics-manage-access)介绍有关工作区管理的一般注意事项。 服务提供商有其他注意事项。

对于 Log Analytics 工作区，有三种可能的体系结构适用于服务提供商：

### <a name="1-distributed---logs-are-stored-in-workspaces-located-in-the-customers-tenant"></a>1.分布式：日志存储在位于客户租户中的工作区内 

在此体系结构中，工作区部署在客户的租户中，此租户用于该客户的所有日志。 使用 [Azure Active Directory 来宾用户 (B2B)](/active-directory/b2b/what-is-b2b) 授予服务提供商管理员访问此工作区的权限。 服务提供商管理员必须在 Azure 门户中切换到其客户的目录才能访问这些工作区。

此体系结构的优点在于：
* 客户可以使用自己的[基于角色的访问权限](/role-based-access-control/overview)来管理对日志的访问。
* 每个客户都可以为其工作区设置不同的设置，例如保留期和数据上限。
* 在客户之间进行隔离以遵守监管和合规要求。
* 每个工作区的费用将计入客户的订阅中。
* 可从所有类型的资源中收集日志，而不仅仅是基于代理的资源。 例如，Azure 审核日志。

此体系结构的缺点在于：
* 对服务提供商而言，一次性管理大量客户租户难度更大。
* 必须在客户目录中预配服务提供商管理员。
* 服务提供商无法跨不同客户分析数据。

### <a name="2-central---logs-are-stored-in-a-workspace-located-in-the-service-provider-tenant"></a>2.集中式 - 日志存储在位于服务提供商租户中的工作区内

在此体系结构中，日志不会存储在客户的租户中，而是仅存储在服务提供商的某个订阅内的中心位置。 客户 VM 上安装的代理配置为使用工作区 ID 和密钥将其日志发送到此工作区。

此体系结构的优点在于：
* 可轻松管理大量客户并将其集成到各种后端系统。
* 服务提供商对日志和各种项目（例如函数和保存的查询）拥有完全所有权。
* 服务提供商可以对其所有客户执行分析。

此体系结构的缺点在于：
* 此体系结构仅适用于基于代理的 VM 数据，不涵盖 PaaS、SaaS 和 Azure 结构数据源。
* 将客户合并到单个工作区时，可能很难分隔客户之间的数据。 唯一效果不错的方法是使用计算机的完全限定域名 (FQDN) 或通过 Azure 订阅 ID 进行分隔。 
* 来自所有客户的全部数据都将存储在具有单独帐单和相同保留期及配置设置的同一区域中。
* Azure 结构及 Azure 诊断和 Azure 审核日志等 PaaS 服务要求工作区与资源位于同一租户中，因此它们无法将日志发送到中心工作区。
* 来自所有客户的全部 VM 代理都将使用相同的工作区 ID 和密钥对中心工作区进行身份验证。 无法在不干扰其他客户的情况下阻止来自特定客户的日志。


### <a name="3-hybrid---logs-are-stored-in-workspace-located-in-the-customers-tenant-and-some-of-them-are-pulled-to-a-central-location"></a>3.混合式：日志存储在位于客户租户中的工作区内，且其中一部分日志会被提取到中心位置。

第三种体系结构混合使用两个选项。 它以第一种分布式体系结构为基础，其中日志对于每个客户而言位于本地，但通过某种机制创建了日志的中心存储库。 一部分日志被提取到中心位置用于报告和分析。 此部分可以是少量数据类型，也可以是每日统计数据等活动摘要。

在中心位置实现日志有两种选择：

1. 中心工作区：服务提供商可以在其租户中创建工作区，并使用利用[查询 API](https://dev.loganalytics.io/) 和[数据收集 API](../../azure-monitor/platform/data-collector-api.md) 的脚本从各个工作区将数据提取到此中心位置。 脚本之外的另一种选择是使用 [Azure 逻辑应用](/logic-apps/logic-apps-overview)。



## <a name="next-steps"></a>后续步骤
* 使用[Resource Manager 模板](template-workspace-configuration.md)自动执行创建和配置工作区
* 使用 [PowerShell](../../azure-monitor/platform/powershell-workspace-configuration.md) 自动创建工作区 
* 使用[警报](../../azure-monitor/platform/alerts-overview.md)以便与现有系统集成




