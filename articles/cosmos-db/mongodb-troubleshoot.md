---
title: 排查 Azure Cosmos DB 的 API for Mongo DB 中的常见错误
description: 本文档讨论解决 Azure Cosmos DB 的 API for MongoDB 中遇到的常见问题的方法。
author: rockboyfor
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: conceptual
origin.date: 06/05/2019
ms.date: 06/17/2019
ms.author: v-yeche
ms.openlocfilehash: f63feacae46a4fb8955b887e678330b73f0f11e3
ms.sourcegitcommit: 153236e4ad63e57ab2ae6ff1d4ca8b83221e3a1c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/18/2019
ms.locfileid: "67171440"
---
# <a name="troubleshoot-common-issues-in-azure-cosmos-dbs-api-for-mongodb"></a>解决 Azure Cosmos DB 的 API for MongoDB 中的常见问题

Azure Cosmos DB 可实现常用 NoSQL 数据库（包括 MongoDB）的线路协议。 由于线路协议的实现，你可以通过使用与 NoSQL 数据库一起工作的现有客户端 SDK、驱动程序和工具来透明地与 Azure Cmoss DB 进行交互。 Azure Cosmos DB 不使用数据库的任何源代码为任何 NoSQL 数据库提供与线路兼容的 API。 任何理解线路协议版本的 MongoDB 客户端驱动程序都可以连接到 Azure Cosmos DB。

虽然 Azure Cosmos DB 的 API for MongoDB 与 3.2 版 MongoDB 线路协议兼容（版本 3.4 中添加的查询运算符和功能目前以预览版的形式提供），但有一些自定义错误代码与 Azure Cosmos DB 特定错误相对应。 本文介绍了不同的错误、错误代码以及解决这些错误的步骤。

## <a name="common-errors-and-solutions"></a>常见错误和解决方法

| 错误               | 代码  | 说明  | 解决方案  |
|---------------------|-------|--------------|-----------|
| TooManyRequests     | 16500 | 使用的请求单位总数超过了集合的预配请求单位率，已达到限制。 | 请考虑从 Azure 门户对分配给一个容器或一组容器的吞吐量进行缩放，也可以重试该操作。 |
| ExceededMemoryLimit | 16501 | 作为一种多租户服务，操作已超出客户端的内存配额。 | 通过限制性更强的查询条件缩小操作的作用域，或者通过 [Azure 门户](https://support.azure.cn/zh-cn/support/support-azure/)联系技术支持。 示例： `db.getCollection('users').aggregate([{$match: {name: "Andy"}}, {$sort: {age: -1}}]))` |
| MongoDB 线路版本问题 | - | 旧版本的 MongoDB 驱动程序无法在连接字符串中检测 Azure Cosmos 帐户的名称。 | 在 Cosmos DB 的 API for MongoDB 连接字符串末尾追加 appName=@**accountName**@  ，其中 ***accountName*** 是 Cosmos DB 帐户名。 |

## <a name="next-steps"></a>后续步骤

- 了解如何将 [Studio 3T](mongodb-mongochef.md) 与 Azure Cosmos DB 的用于 MongoDB 的 API 配合使用。
- 了解如何将 [Robo 3T](mongodb-robomongo.md) 与 Azure Cosmos DB 的用于 MongoDB 的 API 配合使用。
- 通过 Azure Cosmos DB 的用于 MongoDB 的 API 来浏览 MongoDB [示例](mongodb-samples.md)。

<!--Update_Description: new articles on mongodb troubleshoot -->
<!--ms.date: 06/24/2019-->