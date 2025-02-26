---
title: include 文件
description: include 文件
services: azure-resource-manager
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: include
origin.date: 07/11/2019
ms.date: 07/22/2019
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: b4f9997f0c49477a131c8daec0a90f9a9c2e9d1e
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337440"
---
可以将标记应用于 Azure资源，从而将元数据有条理地组织到分类中。 每个标记均由名称和值对组成。 例如，可以对生产中的所有资源应用名称“Environment”和值“Production”。

应用标记以后，即可使用该标记名称和值检索订阅中的所有资源。 使用标记可以从不同资源组中检索相关资源。 需要为计费或管理目的组织资源时，此方法可能很有用。

除了自动标记策略之外，分类还应考虑自助式元数据标记策略，以减轻用户负担并提高准确性。

以下限制适用于标记：

* 并非所有资源类型都支持标记。 若要确定是否可以将标记应用到资源类型，请参阅 [Azure 资源的标记支持](../articles/azure-resource-manager/tag-support.md)。
* 每个资源或资源组最多可以有 50 个标记名称/值对。 目前，存储帐户仅支持 15 个标记，但在未来的版本中，此限制将提高到 50 个。 如果需要应用的标记超过最大允许数量，请使用 JSON 字符串作为标记值。 JSON 字符串可以包含多个应用于单个标记名称的值。 一个资源组可以包含多个资源，这些资源每个都有 50 个标记名称/值对。
* 标记名称不能超过 512 个字符，标记值不能超过 256 个字符。 对于存储帐户，标记名称不能超过 128 个字符，标记值不能超过 256 个字符。
* 通用化 VM 不支持标记。
* 应用于资源组的标记不会被该资源组中的资源继承。
* 不能将标记应用到云服务等经典资源。
* 标记名称不能包含以下字符：`<`、`>`、`%`、`&`、`\`、`?`、`/`
