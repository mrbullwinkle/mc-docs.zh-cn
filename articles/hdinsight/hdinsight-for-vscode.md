---
title: Azure HDInsight 工具 -将 Visual Studio Code 用于 Hive、LLAP 或 pySpark
description: 了解如何使用用于 Visual Studio Code 的 Azure HDInsight 工具来创建、提交查询和脚本。
Keywords: Visual Studio Code,Azure HDInsight Tools,Hive,Python,PySpark,Spark,HDInsight,Hadoop,LLAP,交互式 Hive,交互式查询
services: HDInsight
documentationcenter: ''
author: jejiang
manager: ''
editor: jgao
tags: azure-portal
ms.assetid: ''
ms.service: HDInsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
origin.date: 05/30/2019
ms.date: 07/22/2019
ms.author: v-yiso
ms.openlocfilehash: 25aff627678570d3eb686eaac0f2373d4e807301
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845465"
---
# <a name="use-azure-hdinsight-tools-for-visual-studio-code"></a>使用用于 Visual Studio Code 的 Azure HDInsight 工具

了解如何使用 Visual Studio Code 的 Azure HDInsight 工具，为 Apache Spark 创建和提交 Apache Hive 批处理作业、交互式 Hive 查询和 PySpark 脚本。 本文首先介绍如何在 Visual Studio Code 中安装 HDInsight 工具，然后演练如何向 Hive 和 Spark 提交作业。  

可以在 Visual Studio Code 支持的平台（包括 Windows、Linux 和 macOS）上安装 Azure HDInsight 工具。 可以在下面找到不同平台的必备组件。

## <a name="prerequisites"></a>先决条件

完成本文中的步骤需要具有以下项：

- HDInsight 群集。 若要创建群集，请参阅 [HDInsight 入门](hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md)。
- [Visual Studio Code](https://code.visualstudio.com/)。
- [Mono](https://www.mono-project.com/docs/getting-started/install/)。 只有 Linux 和 macOS 需要 Mono。
- 适用于 Visual Studio Code 的 [Azure 帐户扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)。
- [为 Visual Studio Code 设置 PySpark 交互式环境](set-up-pyspark-interactive-environment.md)。
- 名为 **HDexample** 的本地目录。  本文使用 **C:\HD\HDexample**。

## <a name="install-azure-hdinsight-tools"></a>安装 Azure HDInsight 工具

满足先决条件后，可以安装 Visual Studio Code 的 Azure HDInsight 工具。  完成以下步骤来安装 Azure HDInsight 工具：

1. 打开 Visual Studio Code。

2. 在菜单栏中，导航到“视图” > “扩展”。  

3. 在搜索框中输入 **HDInsight**。

4. 从搜索结果中选择“Azure HDInsight 工具”，然后选择“安装”。    

   ![用于 Visual Studio Code 的 HDInsight - Python 安装](./media/hdinsight-for-vscode/install-hdInsight-plugin.png)

5. 选择“重载”以激活安装的“Azure HDInsight 工具”扩展。  


## <a name="open-hdinsight-work-folder"></a>打开 HDInsight 工作文件夹

完成以下步骤打开工作文件夹，并在 Visual Studio Code 中创建一个文件：

1. 在菜单栏中，导航到“文件” > “打开文件夹...” > “C:\HD\HDexample”，然后选择“选择文件夹”按钮。     该文件夹将显示在左侧的“资源管理器”视图中。 

2. 在“资源管理器”视图中选择文件夹“HDexample”，然后选择工作文件夹旁边的“新建文件”图标。   

   ![新建文件](./media/hdinsight-for-vscode/new-file.png)

3. 为新文件命名，以 .hql（Hive 查询）或 .py（Spark 脚本）作为文件扩展名。  本示例使用 **HelloWorld.hql**。

## <a name="set-the-azure-environment"></a>设置 Azure 环境

1. [连接](#connect-to-azure-account)到 Azure 帐户或链接某个群集（如果尚未这样做）。

2. 在菜单栏中，导航到“视图” > “命令面板...”，然后输入 **HDInsight:  ** 设置 Azure 环境”。

3. 选择一个环境作为默认登录入口。

4. 同时，工具已将你的默认登录入口保存到 **.VSCode\settings.json** 中。 还可以在此配置文件中直接更新默认登录入口。 

   ![设置默认登录入口配置](./media/hdinsight-for-vscode/set-default-login-entry-configuration.png)

## <a name="connect-to-azure-account"></a>连接到 Azure 帐户

将脚本从 Visual Studio Code 提交到 HDInsight 群集之前，需要连接到 Azure 帐户，或链接群集（使用 Ambari 用户名/密码或已加入域的帐户）。  完成以下步骤以连接到 Azure：

1. 在菜单栏中，导航到“视图” > “命令面板...”，然后输入 **HDInsight:  ** Login”。

    ![用于 Visual Studio Code 的 HDInsight 工具 - 登录](./media/hdinsight-for-vscode/hdinsight-for-vscode-extension-login.png)

2. 遵照“输出”窗格中的登录说明操作。 
 
    ![针对其他环境的登录说明](./media/hdinsight-for-vscode/hdi-azure-hdinsight-hdinsight-signin.png)

   连接后，Visual Studio Code 窗口左下角的状态栏上会显示 Azure 帐户名称。  
  

## <a name="link-a-cluster"></a>链接群集

### <a name="link-azure-hdinsight"></a>链接：Azure HDInsight

可以使用 [Apache Ambari](https://ambari.apache.org/) 管理的用户名链接标准群集，也可以使用域用户名（例如：user1@contoso.com）链接 Enterprise Security Pack 安全 Hadoop 群集。

1. 在菜单栏中，导航到“视图” > “命令面板...”，然后输入 **HDInsight:  ** 链接群集”。

   ![链接群集命令](./media/hdinsight-for-vscode/link-cluster-command.png)

2. 选择链接的群集类型“Azure HDInsight”。 

3. 输入 HDInsight 群集 URL。

4. 输入 Ambari 用户名，默认为 **admin**。

5. 输入 Ambari 密码。

6. 选择群集类型。

7. 查看“输出”视图以进行验证。 

   > [!NOTE]  
   > 如果群集已登录到 Azure 订阅中并且已链接群集，则使用链接用户名和密码。  



### <a name="link-generic-livy-endpoint"></a>链接：通用 Livy 终结点

1. 在菜单栏中，导航到“视图” > “命令面板...”，然后输入 **HDInsight:  ** 链接群集”。

2. 选择链接的群集类型“通用 Livy 终结点”。 

3. 输入通用 Livy 终结点，例如 http\://10.172.41.42:18080。

4. 选择授权类型“基本”或“无”。    如果选择“基本”，则：   
    &emsp;a. 输入 Ambari 用户名，默认为 **admin**。  
    &emsp;b. 输入 Ambari 密码。

5. 查看“输出”视图以进行验证。 

## <a name="list-hdinsight-clusters"></a>列出 HDInsight 群集

1. 在菜单栏中，导航到“视图” > “命令面板...”，然后输入 **HDInsight:   List Cluster**。

2. 选择所需的订阅。

3. 查看“输出”视图。   该视图将显示链接的群集，以及 Azure 订阅下的所有群集。

    ![设置默认群集配置](./media/hdinsight-for-vscode/list-cluster-result.png)

## <a name="set-default-cluster"></a>设置默认群集

1. 重新打开[前面](#open-hdinsight-work-folder)创建的文件夹 **HDexample**（如果已关闭）。  

2. 选择[前面](#open-hdinsight-work-folder)创建的文件 **HelloWorld.hql**，它将在脚本编辑器中打开。

3. 右键单击脚本编辑器并选择“HDInsight:  设置默认群集”。  

4. [连接](#connect-to-azure-account)到 Azure 帐户或链接某个群集（如果尚未这样做）。

5. 选择一个群集作为当前脚本文件的默认群集。 工具将自动更新配置文件 **.VSCode\settings.json**。 

   ![设置默认群集配置](./media/hdinsight-for-vscode/set-default-cluster-configuration.png)


## <a name="submit-interactive-hive-queries-hive-batch-scripts"></a>提交交互式 Hive 查询、Hive 批处理脚本

通过 Visual Studio Code 的 HDInsight 工具，可将交互式 Hive 查询和 Hive 批处理脚本提交到 HDInsight 群集。

1. 重新打开[前面](#open-hdinsight-work-folder)创建的文件夹 **HDexample**（如果已关闭）。  

2. 选择[前面](#open-hdinsight-work-folder)创建的文件 **HelloWorld.hql**，它将在脚本编辑器中打开。


3. 将以下代码复制并粘贴到 Hive 文件中，然后保存该文件。

    ```hiveql
    SELECT * FROM hivesampletable;
    ```

4. [连接](#connect-to-azure-account)到 Azure 帐户或链接某个群集（如果尚未这样做）。

5. 右键单击脚本编辑器，选择“HDInsight:  Hive 交互式”提交查询，或使用快捷方式 Ctrl + Alt + I  。选择“HDInsight:  Hive 批处理”以提交脚本，或使用快捷键 Ctrl + Alt + H  。  

6. 如果尚未指定默认群集，请选择群集。 工具还允许使用上下文菜单提交代码块而非整个脚本文件。 不久之后，查询结果将显示在新选项卡中。

   ![交互式 Hive 结果](./media/hdinsight-for-vscode/interactive-hive-result.png)

    - “结果”  面板：可以将整个结果作为 CSV、JSON、Excel 保存到本地路径，也可以只选择多个行。

    - “消息”  面板：选择“行”  号会跳转到运行的脚本的第一行。

## <a name="submit-interactive-pyspark-queries"></a>提交交互式 PySpark 查询

可遵循以下步骤提交交互式 PySpark 查询：

1. 重新打开[前面](#open-hdinsight-work-folder)创建的文件夹 **HDexample**（如果已关闭）。  

2. 遵循[前面](#open-hdinsight-work-folder)所述的步骤创建新文件 **HelloWorld.py**。

3. 将以下代码复制并粘贴到脚本文件中：

   ```python
   from operator import add
   lines = spark.read.text("/HdiSamples/HdiSamples/FoodInspectionData/README").rdd.map(lambda r: r[0])
   counters = lines.flatMap(lambda x: x.split(' ')) \
                .map(lambda x: (x, 1)) \
                .reduceByKey(add)

   coll = counters.collect()
   sortedCollection = sorted(coll, key = lambda r: r[1], reverse = True)

   for i in range(0, 5):
        print(sortedCollection[i])
   ```

4. [连接](#connect-to-azure-account)到 Azure 帐户或链接某个群集（如果尚未这样做）。

5. 选择所有代码，右键单击脚本编辑器并选择“HDInsight:  PySpark Interactive”提交查询，或使用快捷键 **Ctrl + Alt + I**。

   ![pyspark interactive - 右键单击](./media/hdinsight-for-vscode/pyspark-interactive-right-click.png)

6. 如果尚未指定默认群集，请选择群集。 不久之后，新选项卡中会显示“Python Interactive 结果”。  工具还允许使用上下文菜单提交代码块而非整个脚本文件。 

   ![pyspark interactive - python interactive 窗口](./media/hdinsight-for-vscode/pyspark-interactive-python-interactive-window.png) 

7. 输入 **"%%info"** ，然后按 **Shift + Enter** 查看作业信息。 (可选)

   ![查看作业信息](./media/hdinsight-for-vscode/pyspark-interactive-view-job-information.png)

8. 工具还支持 **Spark SQL** 查询。

   ![Pyspark Interactive - 查看结果](./media/hdinsight-for-vscode/pyspark-ineteractive-select-result.png)

   运行查询时，提交状态将显示在底部状态栏的左侧。 当状态为“PySpark 内核(忙)”时，请不要提交其他查询。   

   > [!NOTE] 
   >
   > 如果设置中的“已启用 Python 扩展”处于未选中状态（默认设置已选中），则提交的 pyspark 交互结果将使用旧窗口。 
   >
   > ![已禁用 pyspark interactive python 扩展](./media/hdinsight-for-vscode/pyspark-interactive-python-extension-disabled.png)


## <a name="submit-pyspark-batch-job"></a>提交 PySpark 批处理作业

1. 重新打开[前面](#open-hdinsight-work-folder)创建的文件夹 **HDexample**（如果已关闭）。  

2. 遵循[前面](#open-hdinsight-work-folder)所述的步骤创建新文件 **BatchFile.py**。

3. 将以下代码复制并粘贴到脚本文件中：

    ```python
    from __future__ import print_function
    import sys
    from operator import add
    from pyspark.sql import SparkSession
    if __name__ == "__main__":
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
    
        lines = spark.read.text('/HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv').rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' '))\
                    .map(lambda x: (x, 1))\
                    .reduceByKey(add)
        output = counts.collect()
        for (word, count) in output:
            print("%s: %i" % (word, count))
        spark.stop()
    ```

4. [连接](#connect-to-azure-account)到 Azure 帐户或链接某个群集（如果尚未这样做）。

5. 右键单击脚本编辑器，然后选择  “HDInsight:** PySpark 批处理”，或使用快捷键 Ctrl + Alt + H  。 

6. 选择要将 PySpark 作业提交到的群集。 

   ![提交 python 作业结果](./media/hdinsight-for-vscode/submit-pythonjob-result.png) 

提交 Python 作业后，提交日志会显示在 Visual Studio Code 的“输出”窗口中。  同时还会显示 **Spark UI URL** 和 **Yarn UI URL**。 可以在 Web 浏览器中打开 URL 来跟踪作业状态。

## <a name="apache-livy-configuration"></a>Apache Livy 配置

支持 [Apache Livy](https://livy.incubator.apache.org/) 配置，可在工作空间文件夹中的 **.VSCode\settings.json** 内设置此配置。 目前，livy 配置仅支持 Python 脚本。 详细信息请参阅 [Livy README](https://github.com/cloudera/livy/blob/master/README.rst )。

<a id="triggerlivyconf"></a>**如何触发 livy 配置**

方法 1  
1. 在菜单栏中，导航到“文件” > “首选项” > “设置”。     
2. 在“搜索设置”文本框中，输入“HDInsight 作业提交：   Livy 配置”。  
3. 在相关搜索结果中选择“在 settings.json 中编辑”。 

方法 2   
提交一个文件，然后观察 .vscode 文件夹是否已自动添加到工作文件夹。 可以通过单击 **.vscode\settings.json** 来查找 livy 配置。

+ 项目设置：

    ![Livy 配置](./media/hdinsight-for-vscode/hdi-livyconfig.png)

>[!NOTE]
>对于 **driverMomory** 和 **executorMomry** 设置，请设置带单位的值，例如 1g 或 1024m。 

+ 支持的 Livy 配置：   

    **POST /batches**   
    请求正文

    | name | 说明 | type | 
    | :- | :- | :- | 
    | file | 包含要执行的应用程序的文件 | path（必需） | 
    | proxyUser | 运行作业时要模拟的用户 | string | 
    | className | 应用程序 Java/Spark main 类 | string |
    | args | 应用程序的命令行参数 | 字符串列表 | 
    | jars | 此会话中要使用的 jars | 字符串列表 | 
    | pyFiles | 此会话中要使用的 Python 文件 | 字符串列表 |
    | 文件 | 此会话中要使用的文件 | 字符串列表 |
    | driverMemory | 要用于驱动程序进程的内存量 | string |
    | driverCores | 要用于驱动程序进程的内核数 | int |
    | executorMemory | 每个执行程序进程要使用的内存量 | string |
    | executorCores | 用于每个执行程序的内核数 | int |
    | numExecutors | 为此会话启动的执行程序数量 | int |
    | archives | 此会话中要使用的存档 | 字符串列表 |
    | 队列 | 要提交的 YARN 队列的名称 | string |
    | name | 此会话的名称 | string |
    | conf | Spark 配置属性 | key=val 的映射 |

    响应正文   
    创建的批处理对象。

    | name | 说明 | type | 
    | :- | :- | :- | 
    | id | 会话 id | int | 
    | appId | 此会话的应用程序 id |  String |
    | appInfo | 应用程序详细信息 | key=val 的映射 |
    | log | 日志行 | 字符串列表 |
    | state |   批处理状态 | string |

>[!NOTE]
>当提交脚本时，已分配的 livy 配置将显示在输出窗格中。

## <a name="integrate-with-azure-hdinsight-from-explorer"></a>通过资源管理器与 Azure HDInsight 集成

**Azure HDInsight** 已添加到“资源管理器”视图中。 可以直接通过 **Azure HDInsight** 浏览和管理群集。

1. [连接](#connect-to-azure-account)到 Azure 帐户或链接某个群集（如果尚未这样做）。

2. 在菜单栏中，导航到“视图” > “资源管理器”。  

3. 在左窗格中，展开“AZURE HDINSIGHT”。   将列出可用的订阅和群集（支持 Spark、Hadoop 和 HBase）。 

   ![Azure HDInsight 订阅](./media/hdinsight-for-vscode/hdi-azure-hdinsight-subscription.png)

5. 展开群集以查看 hive 元数据数据库和表架构。

   ![Azure HDInsight 群集](./media/hdinsight-for-vscode/hdi-azure-hdinsight-cluster.png)


## <a name="preview-hive-table"></a>预览 Hive 表
可以通过 **Azure HDInsight** 资源管理器直接在群集中预览 Hive 表。
1. [连接](#connect-to-azure-account)到 Azure 帐户（如果尚未这样做）。

2. 单击最左侧列中的“Azure”图标。 

3. 在左窗格中，展开“AZURE HDINSIGHT”。 此时会列出可用的订阅和群集。

4. 展开群集以查看 hive 元数据数据库和表架构。

5. 右键单击 Hive 表，例如 hivesampletable。 选择“预览”。  

   ![HDInsight for vscode - 预览 hive 表](./media/hdinsight-for-vscode/hdinsight-for-vscode-preview-hive-table.png)

6. 此时会打开“预览结果”窗口。 

   ![HDInsight for vscode - 预览结果窗口](./media/hdinsight-for-vscode/hdinsight-for-vscode-preview-results-window.png)
   
- “结果”面板 

   可以将整个结果作为 CSV、JSON、Excel 保存到本地路径，也可以只选择多个行。

- “消息”面板 
   1. 如果表中的行数大于 100，消息将显示：“显示了 Hive 表的前 100 行”。 
   2. 如果表中的行数小于或等于 100，消息将显示：“显示了 Hive 表的 60 行”。 
   3. 如果表中没有任何内容，消息将显示：“显示了 Hive 表的 0 行”。 

>[!NOTE]
>
>在 Linux 中，安装 xclip 用于复制表数据。
>
>![linux 中的 HDInsight for vscode](./media/hdinsight-for-vscode/hdinsight-for-vscode-preview-linux-install-xclip.png)
## <a name="additional-features"></a>其他功能

Visual Studio Code 的 HDInsight 支持以下功能：

- **IntelliSense 自动完成**。 弹出关键字、方法、变量等相关建议。 不同图标表示不同类型的对象。

    ![用于 Visual Studio Code 的 HDInsight 工具 IntelliSense 对象类型](./media/hdinsight-for-vscode/hdinsight-for-vscode-auto-complete-objects.png)
- **IntelliSense 错误标记**。 语言服务会为 Hive 脚本的编辑错误添加下划线。     
- **语法突出显示**。 语言服务使用不同颜色来区分变量、关键字、数据类型、函数等等。 

    ![用于 Visual Studio Code 的 HDInsight 工具语法突出显示](./media/hdinsight-for-vscode/hdinsight-for-vscode-syntax-highlights.png)

## <a name="reader-only-role"></a>仅限读取者角色

具有群集“仅限读取者”角色的用户不再可以将作业提交到 HDInsight 群集，也不可以查看 Hive 数据库。    需要在 [Azure 门户](https://ms.portal.azure.com/)中联系群集管理员将你的角色升级到“HDInsight 群集操作员”。[    ](https://docs.microsoft.com/azure/hdinsight/hdinsight-migrate-granular-access-cluster-configurations#add-the-hdinsight-cluster-operator-role-assignment-to-a-user) 如果你知道 Ambari 凭据，可遵照以下说明手动链接群集。

### <a name="browse-hdinsight-cluster"></a>浏览 HDInsight 群集  

单击 Azure HDInsight 资源管理器展开 HDInsight 群集时，如果你对群集拥有“仅限读取者”角色，系统会提示你链接群集。 遵循以下步骤通过 Ambari 凭据链接到群集。 

### <a name="submit-job-to-hdinsight-cluster"></a>将作业提交到 HDInsight 群集

将作业提交到 HDInsight 群集时，如果你对群集拥有“仅限读取者”角色，系统会提示你链接群集。 遵循以下步骤通过 Ambari 凭据链接到群集。 

### <a name="link-to-cluster"></a>链接到群集

1.  输入 Ambari 用户名 
2.  输入 Ambari 用户密码。

   ![HDInsight Tools for Visual Studio Code 用户名](./media/hdinsight-for-vscode/hdi-azure-hdinsight-azure-username.png)

   ![HDInsight Tools for Visual Studio Code 密码](./media/hdinsight-for-vscode/hdi-azure-hdinsight-azure-password.png)

> [!NOTE]
>
>可以使用“HDInsight:列出群集”检查已链接的群集。
>
>![HDInsight Tools for Visual Studio Code 读取者已链接](./media/hdinsight-for-vscode/hdi-azure-hdinsight-azure-reader-linked.png)

## <a name="azure-data-lake-storage-gen2-adls-gen2"></a>Azure Data Lake Storage Gen2 (ADLS Gen2)

### <a name="browse-an-adls-gen2-account"></a>浏览 ADLS Gen2 帐户

单击 Azure HDInsight 资源管理器展开 ADLS Gen2 帐户时，如果你的 Azure 帐户对 Gen2 存储没有访问权限，系统会提示你输入存储**访问密钥**。 成功验证访问密钥后，ADLS Gen2 帐户会自动展开。 

### <a name="submit-jobs-to-hdinsight-cluster-with-adls-gen2"></a>将作业提交到包含 ADLS Gen2 的 HDInsight 群集

将作业提交到包含 ADLS Gen2 的 HDInsight 群集时，如果你的 Azure 帐户对 Gen2 存储没有写入访问权限，系统会提示你输入存储**访问密钥**。  成功验证访问密钥后，作业将成功提交。 

![HDInsight Tools for Visual Studio Code 访问密钥](./media/hdinsight-for-vscode/hdi-azure-hdinsight-azure-accesskey.png)   

> [!NOTE]
> 
>可以在 Azure 门户中获取存储帐户的访问密钥。 有关信息，请参阅[查看和复制访问密钥](https://docs.microsoft.com/azure/storage/common/storage-account-manage#view-and-copy-access-keys)。

## <a name="unlink-cluster"></a>取消链接群集

1. 在菜单栏中，导航到“视图” > “命令面板...”，然后输入 **HDInsight:   Unlink a Cluster**。  

2. 选择要取消链接的群集。  

3. 查看“输出”视图以进行验证。   

## <a name="sign-out"></a>注销  

在菜单栏中，导航到“视图” > “命令面板...”，然后输入 **HDInsight:   Logout** 命令。  右下角会弹出一个窗口，指出“注销成功!”。 


## <a name="next-steps"></a>后续步骤
有关使用 Visual Studio Code 的 HDInsight 的演示视频，请参阅 [Visual Studio Code 的 HDInsight](https://go.microsoft.com/fwlink/?linkid=858706)
