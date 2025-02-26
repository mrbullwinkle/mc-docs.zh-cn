---
title: Azure 自动化 Desired State Configuration (DSC) 问题疑难解答
description: 本文提供有关 Desired State Configuration (DSC) 疑难解答的信息
services: automation
ms.service: automation
ms.subservice: ''
author: WenJason
ms.author: v-jay
origin.date: 04/16/2019
ms.date: 07/22/2019
ms.topic: conceptual
manager: digimobile
ms.openlocfilehash: 1fee68d5276b3fe639625c7570215fb9dd821a24
ms.sourcegitcommit: 98cc8aa5b8d0e04cd4818b34f5350c72f617a225
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/17/2019
ms.locfileid: "68298099"
---
# <a name="troubleshoot-desired-state-configuration-dsc"></a>Desired State Configuration (DSC) 疑难解答

本文提供有关 Desired State Configuration (DSC) 问题疑难解答的信息。

## <a name="common-errors-when-working-with-desired-state-configuration-dsc"></a>使用所需状态配置 (DSC) 时的常见错误

### <a name="unsupported-characters"></a>场景：无法从门户删除带有特殊字符的配置

#### <a name="issue"></a>问题

尝试通过门户删除 DSC 配置时，将看到以下错误：

```error
An error occurred while deleting the DSC configuration '<name>'.  Error-details: The argument configurationName with the value <name> is not valid.  Valid configuration names can contain only letters,  numbers, and underscores.  The name must start with a letter.  The length of the name must be between 1 and 64 characters.
```

#### <a name="cause"></a>原因

此错误是计划要解决的临时问题。

#### <a name="resolution"></a>解决方法

* 使用 Az Cmdlet "Remove-AzAutomationDscConfiguration" 删除配置。
* 此 cmdlet 的文档尚未更新。  在其更新前，请参考 AzureRM 模块的文档。
  * [Remove-AzureRmAutomationDSCConfiguration](https://docs.microsoft.com/powershell/module/azurerm.automation/Remove-AzureRmAutomationDscConfiguration)

### <a name="failed-not-found"></a>场景：节点处于失败状态，出现“未找到”错误

#### <a name="issue"></a>问题

该节点有一个状态为“失败”的报表，其中包含错误  ：

```error
The attempt to get the action from server https://<url>//accounts/<account-id>/Nodes(AgentId=<agent-id>)/GetDscAction failed because a valid configuration <guid> cannot be found.
```

#### <a name="cause"></a>原因

将节点分配到配置名称（例如 ABC）而不是节点配置名称（例如 ABC.WebServer）时，通常会发生此错误。

#### <a name="resolution"></a>解决方法

* 确保要为节点分配“节点配置名称”，而不是“配置名称”。
* 可以使用 Azure 门户或 PowerShell cmdlet 将节点配置分配给节点。

  * 若要使用 Azure 门户将节点配置分配给节点，请打开“DSC 节点”页，然后选择一个节点，并单击“分配节点配置”按钮   。  
  * 若要使用 PowerShell cmdlet 将节点配置分配给节点，请使用 **Set-AzureRmAutomationDscNode** cmdlet

### <a name="no-mof-files"></a>场景：编译配置时未生成节点配置（MOF 文件）

#### <a name="issue"></a>问题

DSC 编译作业暂停，且出现错误：

```error
Compilation completed successfully, but no node configuration.mofs were generated.
```

#### <a name="cause"></a>原因

如果 DSC 配置中“Node”关键字后面的表达式的计算结果为 `$null`，则不会生成节点配置  。

#### <a name="resolution"></a>解决方法

下述解决方案中的任何一种都可以解决此问题：

* 确保配置定义中 **Node** 关键字旁边的表达式的计算结果不为 $null。
* 如果要在编译配置时传递 ConfigurationData，请确保从 [ConfigurationData](../automation-dsc-compile.md#configurationdata)传递配置需要的预期值。

### <a name="dsc-in-progress"></a>场景：DSC 节点报告卡在了“正在进行”状态

#### <a name="issue"></a>问题

DSC 代理输出：

```error
No instance found with given property values
```

#### <a name="cause"></a>原因

已升级 WMF 版本，已损坏 WMI。

#### <a name="resolution"></a>解决方法

若要解决此问题，请按照 [DSC 已知问题和限制](https://msdn.microsoft.com/powershell/wmf/5.0/limitation_dsc)一文中的说明进行操作。

### <a name="issue-using-credential"></a>场景：无法在 DSC 配置中使用凭据

#### <a name="issue"></a>问题

DSC 编译作业已暂停，且出现错误：

```error
System.InvalidOperationException error processing property 'Credential' of type <some resource name>: Converting and storing an encrypted password as plaintext is allowed only if PSDscAllowPlainTextPassword is set to true.
```

#### <a name="cause"></a>原因

已在配置中使用凭据，但未提供正确的 **ConfigurationData**，从而无法将每个节点配置的 **PSDscAllowPlainTextPassword** 设置为 true。

#### <a name="resolution"></a>解决方法

* 确保传入正确的 **ConfigurationData**，以便将配置中涉及的每个节点配置的 **PSDscAllowPlainTextPassword** 设置为 true。 有关详细信息，请参阅 [Azure 自动化 DSC 中的资产](../automation-dsc-compile.md#assets)。

### <a name="failure-processing-extension"></a>场景：从 dsc 扩展载入时，出现“处理扩展失败”错误

#### <a name="issue"></a>问题

使用 DSC 扩展载入时失败，出现以下错误：

```error
VM has reported a failure when processing extension 'Microsoft.Powershell.DSC'. Error message: \"DSC COnfiguration 'RegistrationMetaConfigV2' completed with error(s). Following are the first few: Registration of the Dsc Agent with the server <url> failed. The underlying error is: The attempt to register Dsc Agent with Agent Id <ID> with the server <url> return unexpected response code BadRequest. .\".
```

#### <a name="cause"></a>原因

当为节点分配了服务中不存在的节点配置名称时，通常会出现此错误。

#### <a name="resolution"></a>解决方法

* 确保要为节点分配的节点配置名称与服务中的名称完全匹配。
* 可以选择不包含节点配置名称，这将导致载入节点，而不分配节点配置

### <a name="failure-linux-temp-noexec"></a>场景：在 Linux 中应用配置时出现故障，并显示一般错误

#### <a name="issue"></a>问题

在 Linux 中应用配置时，会出现包含以下错误的故障：

```error
This event indicates that failure happens when LCM is processing the configuration. ErrorId is 1. ErrorDetail is The SendConfigurationApply function did not succeed.. ResourceId is [resource]name and SourceInfo is ::nnn::n::resource. ErrorMessage is A general error occurred, not covered by a more specific error code..
```

#### <a name="cause"></a>原因

客户已确定，如果 /tmp 位置设置为 noexec，则当前版本的 DSC 将无法应用配置。

#### <a name="resolution"></a>解决方法

* 从 /tmp 位置中删除 noexec 选项。

## <a name="next-steps"></a>后续步骤

如果你的问题未在本文中列出，或者无法解决问题，请访问以下渠道之一获取更多支持：

* 如需更多帮助，可以提交 Azure 支持事件。 请转到 [Azure 支持站点](https://www.azure.cn/support/)并选择“获取支持”。 