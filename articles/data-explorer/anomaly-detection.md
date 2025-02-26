---
title: 在 Azure 数据资源管理器中进行时序异常情况检测和预测
description: 了解如何使用 Azure 数据资源管理器分析时序数据以检测和预测异常情况。
author: orspod
ms.author: v-biyu
ms.reviewer: jasonh
ms.service: data-explorer
ms.topic: conceptual
origin.date: 04/24/2019
ms.date: 07/22/2019
ms.openlocfilehash: 74e4916b0cafbbcc3447deb04b4193541cd0443c
ms.sourcegitcommit: ea5dc30371bc63836b3cfa665cc64206884d2b4b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/10/2019
ms.locfileid: "67717400"
---
# <a name="anomaly-detection-and-forecasting-in-azure-data-explorer"></a>在 Azure 数据资源管理器中进行异常情况检测和预测

Azure 数据资源管理器持续从云服务或 IoT 设备收集遥测数据。 分析这些数据可获得各种见解，例如，监视服务的运行状况、物理生产流程、使用趋势和负载预测。 分析是针对所选指标的时序执行的，以找出指标模式与其典型正常基线模式之间的偏差。 Azure 数据资源管理器原生支持创建、操作和分析多个时序。 它可以在几秒钟内创建和分析数千个时序，实现近实时的监视解决方案和工作流。

本文将详细介绍 Azure 数据资源管理器时序异常情况检测和预测功能。 适用的时序函数基于一个可靠的已知分解模型，其中的每个原始时序将分解成季节性组件、趋势组件和残余组件。 异常情况是根据残余组件上的离群值检测的，而预测则是通过推算季节性组件和趋势组件执行的。 Azure 数据资源管理器实现显著增强了基本分解模型，它可以自动检测季节性、可靠分析离群值，并使用矢量化实现在几秒钟内处理数千个时序。

## <a name="prerequisites"></a>先决条件

有关时序功能的概述，请参阅 [Azure 数据资源管理器中的时序分析](https://docs.microsoft.com/zh-cn/data-explorer/time-series-analysis)。

## <a name="time-series-decomposition-model"></a>时序分解模型

用于时序预测和异常情况检测的 Azure 数据资源管理器本机实现使用一个已知的分解模型。 此模型将应用到预期的指标时序，以揭示定期行为和趋势的行为（例如服务流量、组件检测信号和 IoT 定期度量值），从而预测将来的指标值和检测异常的指标值。 此回归过程的假设条件是，时序是随机分布的，而不是存在事先已知的季节性行为和趋势行为。 然后，你可以通过季节性组件和趋势组件（统称为基线）预测将来的指标值，并忽略残余部分。 也可以仅使用残余部分基于离群值分析检测异常值。
若要创建分解模型，请使用函数 [`series_decompose()`](https://docs.microsoft.com/zh-cn/azure/kusto/query/series-decomposefunction)。 `series_decompose()` 函数采用一系列时序，并自动将每个时序分解成其季节性、趋势、残余和基线组件。 

例如，可以使用以下查询分解内部 Web 服务的流量：

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t step dt by sid 
| where sid == 'TS1'   //  select a single time series for a cleaner visualization
| extend (baseline, seasonal, trend, residual) = series_decompose(num, -1, 'linefit')  //  decomposition of a set of time series to seasonal, trend, residual, and baseline (seasonal+trend)
| render timechart with(title='Web app. traffic of a month, decomposition', ysplit=panels)
```

![时序分解](media/anomaly-detection/series-decompose-timechart.png)

* 原始时序带有 **num**（如红色所示）标签。 
* 分解过程首先使用函数 [`series_periods_detect()`](https://docs.microsoft.com/zh-cn/azure/kusto/query/series-periods-detectfunction) 自动检测季节性，并提取**季节性**模式（如紫色所示）。
* 从原始时序中减去季节性模式，并使用函数 [`series_fit_line()`](https://docs.microsoft.com/zh-cn/azure/kusto/query/series-fit-linefunction) 运行线性回归，以找到**趋势**组件（如浅蓝色所示）。
* 该函数减去趋势，余下的部分是**残余**组件（如绿色所示）。
* 最后，该函数将季节性组件和趋势组件相加，以生成**基线**（如蓝色所示）。

## <a name="time-series-anomaly-detection"></a>时序异常情况检测

函数 [`series_decompose_anomalies()`](https://docs.microsoft.com/zh-cn/azure/kusto/query/series-decompose-anomaliesfunction) 查找一组时序中的异常点。 此函数调用 `series_decompose()` 来生成分解模型，然后对残余组件运行 [`series_outliers()`](https://docs.microsoft.com/zh-cn/azure/kusto/query/series-outliersfunction)。 `series_outliers()` 使用 Tukey 隔离测试计算残余组件的每个点的异常评分。 异常评分大于 1.5 或小于 -1.5 分别表示异常有轻微的上升或下降。 异常评分大于 3.0 或小于 -3.0 表示明显的异常。 

使用以下查询可以检测内部 Web 服务流量的异常：

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t step dt by sid 
| where sid == 'TS1'   //  select a single time series for a cleaner visualization
| extend (anomalies, score, baseline) = series_decompose_anomalies(num, 1.5, -1, 'linefit')
| render anomalychart with(anomalycolumns=anomalies, title='Web app. traffic of a month, anomalies') //use "| render anomalychart with anomalycolumns=anomalies" to render the anomalies as bold points on the series charts.
```

![时序异常情况检测](media/anomaly-detection/series-anomaly-detection.png)

* 原始时序（如红色所示）。 
* 基线（季节性 + 趋势）组件（如蓝色所示）。
* 原始时序顶层的异常点（如紫色所示）。 异常点明显偏离于预期的基线值。

## <a name="time-series-forecasting"></a>时序预测

函数 [`series_decompose_forecast()`](https://docs.microsoft.com/zh-cn/azure/kusto/query/series-decompose-forecastfunction) 预测一组时序的未来值。 此函数调用 `series_decompose()` 生成分解模型，然后针对每个时序，推断未来的基线组件。

使用以下查询可以预测下一周的 Web 服务流量：

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
let horizon=7d;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t+horizon step dt by sid 
| where sid == 'TS1'   //  select a single time series for a cleaner visualization
| extend forecast = series_decompose_forecast(num, toint(horizon/dt))
| render timechart with(title='Web app. traffic of a month, forecasting the next week by Time Series Decmposition')
```

![时序预测](media/anomaly-detection/series-forecasting.png)

* 原始指标（如红色所示）。 未来值缺失，已按默认设置为 0。
* 推断基线组件（如蓝色所示）以预测下一周的值。

## <a name="scalability"></a>可伸缩性

Azure 数据资源管理器查询语言语法允许通过单个调用来处理多个时序。 其独特的优化实现可以提高性能，在近实时方案中监视数千个计数器时，若要有效进行异常情况检测和预测，这种优势非常关键。

以下查询显示同时处理三个时序的结果：

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
let horizon=7d;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t+horizon step dt by sid
| extend offset=case(sid=='TS3', 4000000, sid=='TS2', 2000000, 0)   //  add artificial offset for easy visualization of multiple time series
| extend num=series_add(num, offset)
| extend forecast = series_decompose_forecast(num, toint(horizon/dt))
| render timechart with(title='Web app. traffic of a month, forecasting the next week for 3 time series')
```

![时序可伸缩性](media/anomaly-detection/series-scalability.png)

## <a name="summary"></a>摘要

本文档详细介绍了用于时序异常情况检测和预测的本机 Azure 数据资源管理器函数。 每个原始时序将分解成季节性、趋势和残余组件，以检测异常情况和/或进行预测。 这些功能可用于近实时监视方案，例如故障检测、预测性维护以及需求和负载预测。

## <a name="next-steps"></a>后续步骤

了解 Azure 数据资源管理器中的[机器学习功能](https://docs.microsoft.com/zh-cn/azure/data-explorer/machine-learning-clustering)。
