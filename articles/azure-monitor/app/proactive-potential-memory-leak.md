---
title: 智能检测 - Azure Application Insights 检测到的可能内存泄漏 | Azure Docs
description: 使用 Azure Application Insights 监视应用程序是否存在可能的内存泄漏。
services: application-insights
documentationcenter: ''
author: lingliw
manager: digimobile
ms.assetid: ea2a28ed-4cd9-4006-bd5a-d4c76f4ec20b
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 6/4/2019
ms.author: v-lingwu
ms.openlocfilehash: 223ca08c462dd09fa57c2452713129eb89a8745b
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562667"
---
# <a name="memory-leak-detection-preview"></a>内存泄漏检测（预览）

Application Insights 自动分析应用程序中每个进程的内存消耗量，并可以就可能的内存泄漏或内存消耗量增加向你发出警告。

此功能需要[配置性能计数器](/azure-monitor/app/performance-counters)，除此之外，不需要其他特殊设置。 当应用生成的内存性能计数器遥测数据（例如，专用字节数）足够多时，它处于活动状态。

## <a name="when-would-i-get-this-type-of-smart-detection-notification"></a>何时会收到此类型的智能检测通知？
在属于你应用程序的一个或多个进程和/或一个或多个计算机中，如果内存消耗量在较长的时间段内持续增加，将发出一个典型的通知。 机器学习算法用于检测与内存泄漏模式匹配的内存消耗增加。

## <a name="does-my-app-really-have-a-problem"></a>我的应用真的有问题吗？
不是，通知并不意味着应用肯定有问题。 虽然内存泄漏模式通常指示应用程序有问题，但这些模式可能对特定进程而言是非常典型的，或者可能有自然的业务理由，并且可以被忽略。

## <a name="how-do-i-fix-it"></a>如何解决问题？
通知包括诊断信息，以在诊断分析进程中提供支持：
1. **会审**。 通知显示增加的内存量（以 GB 为单位），以及内存增加的时间范围。 这可以帮助你对问题分配优先级。
2. **划分范围。** 多少台计算机表现出内存泄漏模式？ 可能内存泄漏期间触发了多少个异常？ 可以从通知中获取此信息。
3. **诊断。** 检测包含内存泄漏模式，该模式显示随时间推移进程的内存消耗量。 还可以使用链接到支持信息的相关项和报告，帮助进一步诊断问题。




