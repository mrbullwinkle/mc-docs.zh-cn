---
title: 使用 Azure Monitor 工作簿创建交互式报告 | Microsoft Docs
description: 使用用于 VM 的 Azure Monitor 的预定义和自定义参数化工作簿简化复杂的报告。
services: azure-monitor
documentationcenter: ''
author: lingliw
manager: digimobile
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/02/2019
ms.author: v-lingwu
ms.openlocfilehash: 57c3d476d2ef410b18fa44e11e70447ff3bb589e
ms.sourcegitcommit: e78670855b207c6084997f747ad8e8c3afa3518b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68513849"
---
# <a name="create-interactive-reports-with-azure-monitor-workbooks"></a>使用 Azure Monitor 工作簿创建交互式报表

工作簿可将文本、 [日志查询](../log-query/query-language.md)、指标和参数合并到丰富的交互式报告中。 有权访问相同 Azure 资源的其他团队成员都可编辑工作簿。

在如下所述的场景中，工作簿非常有用：

* 当你事先不知道所需的指标时，浏览虚拟机的使用情况：CPU 利用率、磁盘空间、内存、网络依赖关系，等等。与其他使用情况分析工具不同，工作簿可以结合多个类型的可视化效果和分析，非常适合这种自由探索。
* 通过显示关键计数器的指标和其他日志事件，向团队解释最近预配的 VM 的性能如何。
* 与团队的其他成员分享调整 VM 试验规模的结果。 可以用文本解释试验的目标，然后展示用于评估试验的每个使用情况指标和 Analytics 查询，以及说明每个指标是否高于或低于目标的标注。
* 结合数据、文本说明和后续步骤讨论，报告故障对 VM 使用的影响，从而防止未来发生故障。

用于 VM 的 Azure Monitor 包含多个工作簿以帮助你入门，下表做了汇总。

| 工作簿 | 说明 | 作用域 |
|----------|-------------|-------|
| 性能 | 在单个工作簿中提供“前 N 项列表”和“图表”视图的可自定义版本，利用所有已启用的 Log Analytics 性能计数器。| 大规模 |
| 性能计数器 | 基于众多性能计数器的“前 N 项”图表视图。 | 大规模 |
| 连接 | “连接”提供受监视 VM 建立的入站和出站连接的深入视图。 | 大规模 |
| 活动端口 | 提供已绑定到受监视 VM 上的端口的进程列表，及其在所选时间范围内的活动。 | 大规模 |
| 打开端口 | 提供受监视 VM 上已打开的端口数，以及有关这些已打开端口的详细信息。 | 大规模 |
| 失败的连接数 | 显示受监视 VM 上的失败连接计数、失败趋势，以及失败百分比是否不断增大。 | 大规模 |
| 安全和审核 | TCP/IP 流量的分析，其中报告了连接总数、恶意连接数，以及 IP 终结点的全球位置。  若要启用所有功能，需要启用安全检测。 | 大规模 |
| TCP 流量 | 受监视 VM 的排名报告，在网格中以趋势线的形式显示它们已发送、接收的网络流量和总量。 | 大规模 |
| 流量比较 | 在此工作簿中可以比较一台计算机或一组计算机的网络流量趋势。 | 大规模 |
| 性能 | 提供性能视图的可自定义版本，利用所有已启用的 Log Analytics 性能计数器。 | 单个 VM | 
| 连接 | “连接”提供 VM 建立的入站和出站连接的深入视图。 | 单个 VM |
 
## <a name="starting-with-a-template-or-saved-workbook"></a>从模板或已保存的工作簿开始

工作簿由可单独编辑的图表、表格、文本和输入控件构成的各部分组成。 为了更好地理解工作簿，首先让我们打开一个模板，并逐步创建一个自定义工作簿。 

1. 登录到 [Azure 门户](https://portal.azure.cn)。

2. 选择“虚拟机”。 

3. 从列表中选择一个虚拟机。

4. 在“VM”页上的“监视”部分，选择“见解(预览版)”。  

5. 在 VM 见解页面上选择“性能”或“映射”选项卡，然后通过页面上的链接选择“查看工作簿”。    

    ![导航到工作簿的屏幕截图](media/vminsights-workbooks/workbook-option-01.png)

6. 从底部的下拉列表中选择“转到库”。 

    ![工作簿下拉列表屏幕截图](media/vminsights-workbooks/workbook-dropdown-gallery-01.png)

    这会启动包含大量预生成工作簿的工作簿库，以帮助你入门。

7. 我们将从“默认模板”开始，位于“快速入门”标题下方   。

    ![工作簿库的屏幕截图](media/vminsights-workbooks/workbook-gallery-01.png)

## <a name="editing-workbook-sections"></a>编辑工作簿部分

工作簿具有两种模式：编辑模式和阅读模式   。 默认工作簿模板首次启动时，将以**编辑模式**打开。 它会显示工作簿的所有内容，其中包括以任何方式隐藏的任何步骤和参数。 **阅读模式**提供了简化的报表样式视图。 在阅读模式下，可以免去创建报告时所遇到的复杂性，同时仍保留相应的基础机制，即只需几次单击即可进行修改。

![用于 VM 的 Azure Monitor 工作簿部分编辑控件](media/vminsights-workbooks/workbook-new-workbook-editor-01.png)

1. 完成编辑某个部分后，单击该部分左下角的“完成编辑”  。

2. 创建某个部分的副本，请单击“克隆此部分”  图标。 创建副本是迭代查询的绝佳方式，不会丢失以前的迭代。

3. 要上移工作簿中的某个部分，请单击“上移”  或“下移”  图标。

4. 若要永久删除某个部分，请单击“删除”  图标。

## <a name="adding-text-and-markdown-sections"></a>添加文本和 Markdown 部分

为工作簿添加标题、说明和注释有助于叙述一组表和图表。 工作簿中的文本部分支持用于设置文本格式的 [Markdown 语法](https://daringfireball.net/projects/markdown/)，如标题、粗体、斜体和点符列表。

若要将文本部分添加到工作簿，使用工作簿底部或任何部分底部的“添加文本”  按钮。

## <a name="adding-query-sections"></a>添加查询部分

![工作簿中的查询部分](media/vminsights-workbooks/005-workbook-query-section.png)

若要将查询部分添加到工作簿，使用工作簿底部或任何部分底部的“添加查询”按钮  。

查询部分非常灵活，可用于回答以下问题：

* 在网络流量增大的同一时间段内，CPU 利用率如何？
* 过去一个月的可用磁盘空间趋势如何？
* 我的 VM 在过去两周发生了多少次网络连接失败？ 

此外，不仅限于在通过工作簿启动的虚拟机的上下文中进行查询。 可以跨多个虚拟机以及 Log Analytics 工作区查询，前提是你对这些资源拥有访问权限。

使用**工作区**标识符包含其他 Log Analytics 工作区或特定 Application Insights 应用中的数据。 若要了解有关跨资源查询的详细信息，请参阅[官方指南](../log-query/cross-workspace-query.md)。

### <a name="advanced-analytic-query-settings"></a>高级分析查询设置

每个部分都有自己的高级设置，可通过位于“添加参数”按钮右侧的![工作簿部分编辑控件](media/vminsights-workbooks/006-settings.png)图标设置访问  。

![用于 VM 的 Azure Monitor 工作簿部分编辑控件](media/vminsights-workbooks/007-settings-expanded.png)

|         |          |
| ---------------- |:-----|
| **自定义宽度**    | 将某项设置为任意大小，以便可将许多项容纳在单行中，使你更好地将图表和表格整理到丰富的交互式报告中。  |
| **有条件可见** | 指定在阅读模式下根据参数隐藏步骤。 |
| **导出参数**| 允许网格或图表中的所选行能够导致后续步骤更改值或变得可见。  |
| **未进行编辑时显示查询** | 在图表或表格上方显示查询，即便处于阅读模式也是如此。
| **未进行编辑时显示“在分析中打开”按钮** | 向图表的右侧添加蓝色的“分析”图标，以允许单击访问。|

大多数这些设置都相当直观，但若要了解导出参数，最好查看使用此功能的工作簿  。

预生成的工作簿之一：“TCP 流量”，提供有关 VM 发出的连接指标的信息  。

工作簿的第一部分基于日志查询数据。 第个部分也基于日志查询数据，但在第一个表中选择某一行将交互更新图表的内容：

![用于 VM 的 Azure Monitor 工作簿部分编辑控件](media/vminsights-workbooks/008-workbook-tcp-traffic.png)

可以通过使用在表格的日志查询中启用的高级设置“选择项目后，导出参数”实现该行为  。

![用于 VM 的 Azure Monitor 工作簿部分编辑控件](media/vminsights-workbooks/009-settings-export.png)

然后，在选择某个行创建部分标题和图表使用的一组值时，第二个日志查询将利用导出的值。 如果未选择任何行，则会隐藏部分标题和图表。 

例如，第二个部分中已隐藏的参数将从网格中选定行使用以下引用：

```
VMConnection
| where TimeGenerated {TimeRange}
| where Computer in ("{ComputerName}") or '*' in ("{ComputerName}") 
| summarize Sent = sum(BytesSent), Received = sum(BytesReceived) by bin(TimeGenerated, {TimeRange:grain})
```

## <a name="adding-metrics-sections"></a>添加指标部分

指标部分提供完全访问权限，以将 Azure Monitor 指标数据纳入交互式报表。 在用于 VM 的 Azure Monitor 中，预生成的工作簿通常包含分析查询数据而不是指标数据。  可以选择创建包含指标数据的工作簿，以便在一个位置充分利用这两项功能。 此外，还能够从任何有权访问的订阅中的资源中提取指标数据。

下例是关于被拉取到工作簿中以提供 CPU 性能的网格可视化效果的虚拟机数据：

![用于 VM 的 Azure Monitor 工作簿部分编辑控件](media/vminsights-workbooks/010-metrics-grid.png)

## <a name="adding-parameter-sections"></a>添加参数部分

工作簿参数使你能够更改工作簿中的值，而无需手动编辑查询或文本部分。 这免除了对了解基础分析查询语言的需求，并极大地扩大了基于工作簿的报表的潜在受众范围。

通过将参数名称放在大括号中（如 ``{parameterName}``），以查询、文本或其他参数部分替换参数值。 参数名称限于类似 JavaScript 标识符的规则，包含字母字符或下划线，后接字母数字字符或下划线。 例如，允许使用 a1，但不允许使用 1a   。

参数是线性的，从工作簿的顶部开始并向下转到后续步骤。  工作簿中稍后声明的参数可以替代前面声明的参数。 此外，这还允许使用查询的参数访问前面定义的参数值。 在参数步骤本身中，参数也是线性的，从左到右，右侧的参数可以依赖以前在该步骤中声明的参数。
 
当前支持四种不同类型的参数：

|                  |      |
| ---------------- |:-----|
| **文本**    | 允许用户编辑文本框，你可以选择提供一个查询用于填充默认值。 |
| **下拉列表** | 允许用户从一组值中进行选择。 |
| **时间范围选取器**| 允许用户从一组预定义的时间范围值中选择，或者从自定义时间范围内选择。|
| **资源选取器** | 允许用户从为工作簿所选资源中选择。|

### <a name="using-a-text-parameter"></a>使用文本参数

用户在文本框中输入的值将直接在查询中替换，不带任何转义或引号。 如果所需值为字符串，则查询应该在参数周围采用引号（如 '{parameter}'）  。

文本参数允许在任意位置使用文本框中的值。 它可以是表名、列名、函数名、运算符等等。文本参数类型包含“从分析查询获取默认值”设置，使工作簿作者能够使用查询填充该文本框的默认值  。

使用日志查询中的默认值时，仅第一行的第一个值（行 0，列 0）用作默认值。 因此建议限制查询，以仅返回一行和一列。 忽略由查询返回的其他任何数据。 

查询返回的任何值都将被直接替换，不带任何转义或引号。 如果查询不返回行，该参数的结果为空字符串（如果参数不是必需的）或未定义（如果参数是必需的）。

### <a name="using-a-drop-down"></a>使用下拉列表

下拉列表参数类型可让你创建一个下拉列表控件，支持选择一个或多个值。

下拉列表由日志查询或 JSON 填充。 如果查询返回一列，则该列中的值同时为下拉列表控件中的值和标签。 如果查询返回两列，第一列为值，第二列为下拉列表中显示的标签。 如果查询返回三列，则第三列用于指示该下拉列表中的默认选择。 此列可以是任何类型，但最简单的是使用布尔值或数值类型，其中 0 表示 false，1 表示 true。

如果列为字符串类型，null/空字符串被视为 false，而任何其他值被视为 true。 对于单项选择下拉列表，值为 true 的第一个值用作默认选择。  对于多项选择下拉列表，值为 true 的所有值用作默认选择集。 下拉列表中的项以查询返回行的任何顺序显示。 

让我们看看“连接概述”报告中提供的参数。 单击“方向”旁边的编辑符号  。

![用于 VM 的 Azure Monitor 工作簿部分编辑控件](media/vminsights-workbooks/011-workbook-using-dropdown.png)

此时会启动“编辑参数”菜单项  。

![用于 VM 的 Azure Monitor 工作簿部分编辑控件](media/vminsights-workbooks/012-workbook-edit-parameter.png)

使用 JSON 可以生成填充了内容的任意表。 例如，以下 JSON 在下拉列表中生成两个值：

```
[
    { "value": "inbound", "label": "Inbound"},
    { "value": "outbound", "label": "Outbound"}
]
```

更适用的示例使用下拉列表来按名称从一组性能计数器中选择：

```
Perf
| summarize by CounterName, ObjectName
| order by ObjectName asc, CounterName asc
| project Counter = pack('counter', CounterName, 'object', ObjectName), CounterName, group = ObjectName
```

该查询将显示如下所示的结果：

![性能计数器下拉列表](media/vminsights-workbooks/013-workbook-edit-parameter-perf-counters.png)

下拉列表是极其强大的工具，可用于自定义和创建交互式报告。

### <a name="time-range-parameters"></a>时间范围参数

尽管可以通过下拉列表参数类型制作自己的自定义时间范围参数，但如果不需要相同程度的灵活性，还可使用现成的时间范围参数类型。 

时间范围参数类型包含 15 个默认范围，从 5 分钟到 90 天不等。 此外，还可以通过一个选项，允许自定义时间范围选择，使报表操作员能够选择明确的开始和停止时间范围值。

### <a name="resource-picker"></a>资源选取器

资源选取器参数类型使你能够将报表类型范围限定到特定资源类型。 “性能”工作簿就是利用资源选取器类型的一个预生成的工作簿示例  。

![工作区下拉列表](media/vminsights-workbooks/014-workbook-edit-parameter-workspaces.png)

## <a name="saving-and-sharing-workbooks-with-your-team"></a>保存并与团队共享工作簿

工作簿保存在 Log Analytics 工作区或虚拟机资源中，具体取决于如何访问工作簿库。 可将工作簿保存到你专用的“我的报告”部分，或者可供有权访问该资源的任何用户可访问的“共享报告”部分   。 若要查看资源中的所有工作簿，请单击操作栏中的“打开”  按钮。

共享目前在“我的报表”  中的工作簿：

1. 在操作栏中，单击“打开” 
2. 单击要共享的工作簿旁的“...”按钮
3. 单击“移到共享报表”  。

若要通过链接或电子邮件共享工作簿，请单击操作栏中的“共享”  。 请记住，链接的收件人需要在 Azure 门户中访问此资源才能查看该工作簿。 若要进行编辑，收件人至少需要资源的参与者权限。

将工作簿的链接固定到 Azure 仪表板：

1. 在操作栏中，单击“打开” 
2. 单击需固定的工作簿旁的“...”按钮
3. 单击“固定到仪表板”  。

## <a name="next-steps"></a>后续步骤
若要了解如何使用“运行状况”功能，请参阅[查看 Azure VM 运行状况](vminsights-health.md)；若要查看已发现的应用程序依赖项，请参阅[查看用于 VM 的 Azure Monitor 映射](vminsights-maps.md)。 