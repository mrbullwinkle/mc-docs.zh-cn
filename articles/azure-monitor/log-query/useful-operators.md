---
title: Azure Monitor 日志查询中的有用运算符 | Microsoft Docs
description: Azure Monitor 日志查询中用于各种方案的常见函数。
services: log-analytics
documentationcenter: ''
author: lingliw
manager: digimobile
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 08/21/2018
ms.author: v-lingwu
ms.openlocfilehash: be2d7064e2c663db275acc782da949a4c41a8b52
ms.sourcegitcommit: fd927ef42e8e7c5829d7c73dc9864e26f2a11aaa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67562906"
---
# <a name="useful-operators-in-azure-monitor-log-queries"></a>Azure Monitor 日志查询中的有用运算符

下表提供了一些常用函数，可用于 Azure Monitor 日志查询中的不同方案。

## <a name="useful-operators"></a>有用的运算符

Category                                |相关分析函数
----------------------------------------|----------------------------------------
所选内容和列别名            |`project`、`project-away`、`extend`
临时表和常量          |`let scalar_alias_name = …;` <br> `let table_alias_name =  …  …  … ;`| 
比较运算符和字符串运算符         |`startswith`、`!startswith`、`has`、`!has` <br> `contains`、`!contains`、`containscs` <br> `hasprefix`、`!hasprefix`、`hassuffix`、`!hassuffix`、`in`、`!in` <br> `matches regex` <br> `==`、`=~`、`!=`、`!~`
常用字符串函数                 |`strcat()`、`replace()`、`tolower()`、`toupper()`、`substring()`、`strlen()`
常用数学函数                   |`sqrt()`, `abs()` <br> `exp()`、`exp2()`、`exp10()`、`log()`、`log2()`、`log10()`、`pow()` <br> `gamma()`, `gammaln()`
分析文本                            |`extract()`、`extractjson()`、`parse`、`split()`
限制输出                         |`take`、`limit`、`top`、`sample`
日期函数                          |`now()`, `ago()` <br> `datetime()`、`datepart()`、`timespan` <br> `startofday()`、`startofweek()`、`startofmonth()`、`startofyear()` <br> `endofday()`、`endofweek()`、`endofmonth()`、`endofyear()` <br> `dayofweek()`、`dayofmonth()`、`dayofyear()` <br> `getmonth()`、`getyear()`、`weekofyear()`、`monthofyear()`
分组和聚合                |`summarize by` <br> `max()`、`min()`、`count()`、`dcount()`、`avg()`、`sum()` <br> `stddev()`、`countif()`、`dcountif()`、`argmax()`、`argmin()` <br> `percentiles()`, `percentile_array()`
联接和联合                        |`join kind=leftouter`、`inner`、`rightouter`、`fullouter`、`leftanti` <br> `union`
排序、顺序                             |`sort`, `order` 
动态对象（JSON 和数组）         |`parsejson()` <br> `makeset()`, `makelist()` <br> `split()`, `arraylength()` <br> `zip()`, `pack()`
逻辑运算符                       |`and`、`or`、`iff(condition, value_t, value_f)` <br> `binary_and()`、`binary_or()`、`binary_not()`、`binary_xor()`
机器学习                        |`evaluate autocluster`、`basket`、`diffpatterns`、`extractcolumns`


## <a name="next-steps"></a>后续步骤

- 完成关于[在 Azure Monitor 中编写日志查询](get-started-queries.md)的一课。
