---
title: Azure Database for MySQL 的服务器日志
description: 介绍 Azure Database for MySQL 中提供的慢查询日志，以及用于启用不同日志记录级别的可用参数。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.topic: conceptual
origin.date: 05/29/2019
ms.date: 07/15/2019
ms.openlocfilehash: b5fdbf2cca0cc6c0c9f6226fc6e97c8dd319d1b9
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845111"
---
# <a name="slow-query-logs-in-azure-database-for-mysql"></a>Azure Database for MySQL 中的慢查询日志

> [!NOTE]
> 将要查看的是 Azure Database for MySQL 的新服务。 若要查看经典 MySQL Database for Azure 的文档，请访问[此页](https://docs.azure.cn/zh-cn/mysql-database-on-azure/)。

在 Azure Database for MySQL 中，慢查询日志可供用户使用。 不支持访问事务日志。 可以使用慢查询日志来查明性能瓶颈以进行故障排除。

有关 MySQL 慢查询日志的详细信息，请参阅 MySQL 参考手册中的[慢查询日志部分](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)。

## <a name="access-slow-query-logs"></a>访问慢查询日志
可以使用 Azure 门户和 Azure CLI 列出和下载 Azure Database for MySQL 慢查询日志。

在 Azure 门户中，选择 Azure Database for MySQL 服务器。 在“监视”标题下，选择“服务器日志”页面。  

有关 Azure CLI 的详细信息，请参阅[使用 Azure CLI 配置和访问服务器日志](howto-configure-server-logs-in-cli.md)。

## <a name="log-retention"></a>日志保留期
日志从其创建时开始算起，最多可保留七天。 如果可用日志的总大小超过了 7 GB，则会删除最旧的文件，直到有空间可用。 

日志每 24 小时或每 7 GB 轮换一次（以先达到的条件为准）。

## <a name="configure-slow-query-logging"></a>配置慢查询日志记录 
默认情况下，慢查询日志被禁用。 若要启用它，请将 slow_query_log 设置为 ON。

可以调整的其他参数包括：

- **long_query_time**：如果某个查询花费的时间超过了 long_query_time（以秒为单位），则会记录该查询。 默认为 10 秒。
- **log_slow_admin_statements**：如果为 ON，则会在写入到 slow_query_log 的语句中包括管理性语句，例如 ALTER_TABLE 和 ANALYZE_TABLE。
- **log_queries_not_using_indexes**：确定是否将未使用索引的查询记录到 slow_query_log 中
- **log_throttle_queries_not_using_indexes**：此参数限制可以写入到慢查询日志的非索引查询的数目。 当 log_queries_not_using_indexes 设置为 ON 时，此参数生效。

有关慢查询日志参数的完整说明，请参阅 MySQL [慢查询日志文档](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)。

## <a name="diagnostic-logs"></a>诊断日志
Azure Database for MySQL 集成了 Azure Monitor 诊断日志。 在 MySQL 服务器上启用慢查询日志后，可以选择将它们发送到 Azure Monitor 日志、事件中心或 Azure 存储。 若要详细了解如何启用诊断日志，请参阅[诊断日志文档](../azure-monitor/platform/diagnostic-logs-overview.md)中的操作说明部分。

> [!IMPORTANT]
> 服务器日志的此诊断功能仅适用于“常规用途”和“内存优化”的[定价层](concepts-pricing-tiers.md)。

下表介绍了每个日志中的内容。 根据输出方法，包含的字段以及这些字段出现的顺序可能会有所不同。

| **属性** | **说明** |
|---|---|
| `TenantId` | 租户 ID |
| `SourceSystem` | `Azure` |
| `TimeGenerated` [UTC] | 记录日志时的时间戳 (UTC) |
| `Type` | 日志的类型。 始终是 `AzureDiagnostics` |
| `SubscriptionId` | 服务器所属的订阅的 GUID |
| `ResourceGroup` | 服务器所属的资源组的名称 |
| `ResourceProvider` | 资源提供程序的名称。 始终是 `MICROSOFT.DBFORMYSQL` |
| `ResourceType` | `Servers` |
| `ResourceId` | 资源 URI |
| `Resource` | 服务器的名称 |
| `Category` | `MySqlSlowLogs` |
| `OperationName` | `LogEvent` |
| `Logical_server_name_s` | 服务器的名称 |
| `start_time_t` [UTC] | 查询开始时间 |
| `query_time_s` | 查询执行的总时间 |
| `lock_time_s` | 查询被锁定的总时间 |
| `user_host_s` | 用户名 |
| `rows_sent_s` | 发送的行数 |
| `rows_examined_s` | 检查的行数 |
| `last_insert_id_s` | [last_insert_id](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_last-insert-id) |
| `insert_id_s` | 插入 ID |
| `sql_text_s` | 完整查询 |
| `server_id_s` | 服务器 ID |
| `thread_id_s` | 线程 ID |
| `\_ResourceId` | 资源 URI |

## <a name="next-steps"></a>后续步骤
- [如何通过 Azure CLI 配置和访问服务器日志](howto-configure-server-logs-in-cli.md)。
