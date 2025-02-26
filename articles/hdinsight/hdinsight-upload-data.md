---
title: 在 HDInsight 中上传 Apache Hadoop 作业的数据
description: 了解如何使用 Azure 经典 CLI、Azure 存储资源管理器、Azure PowerShell、Hadoop 命令行或 Sqoop 在 HDInsight 中上传和访问 Apache Hadoop 作业的数据。
keywords: etl hadoop, 将数据引入 hadoop, hadoop 加载数据
services: hdinsight
author: hrasheed-msft
ms.reviewer: jasonh
ms.author: v-yiso
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
origin.date: 06/03/2019
ms.date: 06/22/2019
ms.openlocfilehash: 6bc12a94c8f9928994d790c0af8411be0f2d834c
ms.sourcegitcommit: f4351979a313ac7b5700deab684d1153ae51d725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/12/2019
ms.locfileid: "67845186"
---
# <a name="upload-data-for-apache-hadoop-jobs-in-hdinsight"></a>在 HDInsight 中上传 Apache Hadoop 作业的数据

Azure HDInsight 提供一个基于 Azure 存储和 Azure Data Lake Storage (Gen2) 的功能完备的 Hadoop 分布式文件系统 (HDFS)。 Azure 存储以及 Data Lake Storage Gen2 设计为 HDFS 扩展，为客户提供无缝体验。 它们通过启用 Hadoop 生态系统中的整套组件以直接操作其管理的数据。 Azure 存储、Data Lake Storage Gen1 和 Data lake Storage Gen2 是独特的文件系统，并且已针对数据的存储和计算进行了优化。 若要了解使用 Azure 存储的优点，请参阅[将 Azure 存储与 HDInsight 配合使用][hdinsight-storage]。

## <a name="prerequisites"></a>先决条件

在开始下一步之前，请注意以下要求：

* 一个 Azure HDInsight 群集。 有关说明，请参阅 [Azure HDInsight 入门][hdinsight-get-started]或[创建 HDInsight 群集](hdinsight-hadoop-provision-linux-clusters.md)。
* 了解以下文章：

    - [将 Azure 存储与 HDInsight 配合使用][hdinsight-storage]
    - [将 Data Lake Storage Gen2 与 HDInsight 配合使用](hdinsight-hadoop-use-data-lake-storage-gen2.md)  

## <a name="upload-data-to-azure-storage"></a>将数据上传到 Azure 存储

## <a name="utilities"></a>实用程序
Microsoft 提供以下实用工具用于操作 Azure 存储：

| 工具 | Linux | OS X | Windows |
| --- |:---:|:---:|:---:|
| [Azure 门户](../storage/blobs/storage-quickstart-blobs-portal.md) |✔ |✔ |✔ |
| [Azure CLI](../storage/blobs/storage-quickstart-blobs-cli.md) |✔ |✔ |✔ |
| [Azure PowerShell](../storage/blobs/storage-quickstart-blobs-powershell.md) | | |✔ |
| [AzCopy](../storage/common/storage-use-azcopy-v10.md) |✔ | |✔ |
| [Hadoop 命令](#commandline) |✔ |✔ |✔ |


> [!NOTE]  
> Hadoop 命令仅在 HDInsight 群集上可用。 使用该命令只能将数据从本地文件系统载入 Azure 存储。  


## <a id="commandline"></a>Hadoop 命令行
仅当数据已存在于群集头节点中时，才可以使用 Hadoop 命令行将数据存储到 Azure 存储 Blob。

若要使用 Hadoop 命令，必须先通过 [SSH 或 PuTTY](hdinsight-hadoop-linux-use-ssh-unix.md) 连接到头节点。

连接之后，可以使用以下语法将文件上传到存储。

```bash
hadoop -copyFromLocal <localFilePath> <storageFilePath>
```

例如： `hadoop fs -copyFromLocal data.txt /example/data/data.txt`

由于 HDInsight 的默认文件系统在 Azure 存储中，/example/data.txt 实际是在 Azure 存储中。 也可以将该文件表示为：

    wasbs:///example/data/data.txt

或

    wasbs://<ContainerName>@<StorageAccountName>.blob.core.windows.net/example/data/davinci.txt

若要查看可用于文件的其他 Hadoop 命令的列表，请参阅 [https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html)

> [!WARNING]
> 在 Apache HBase 群集上，写入数据为 256 KB 时会使用默认块大小。 虽然在使用 HBase Api 或 REST API 时可良好运行，但使用 `hadoop` 或 `hdfs dfs` 命令编写大于 ~12 GB 的数据会导致错误。 有关详细信息，请参阅本文的[在 Blob 上编写时的存储异常](#storageexception)部分。
>
>

## <a name="graphical-clients"></a>图形客户端
还有一些应用程序可提供用于 Azure 存储的图形界面。 下表是其中一些应用程序的列表：

| 客户端 | Linux | OS X | Windows |
| --- |:---:|:---:|:---:|
| [用于 HDInsight 的 Microsoft Visual Studio Tools](hadoop/apache-hadoop-visual-studio-tools-get-started.md#explore-linked-resources) |✔ |✔ |✔ |
| [Azure 存储空间资源管理器](../storage/blobs/storage-quickstart-blobs-storage-explorer.md) |✔ |✔ |✔ |
| [Cerulea](https://www.cerebrata.com/products/cerulean/features/azure-storage) | | |✔ |
| [CloudXplorer](http://clumsyleaf.com/products/cloudxplorer) | | |✔ |
| [适用于 Microsoft Azure 的 CloudBerry Explorer](https://www.cloudberrylab.com/free-microsoft-azure-explorer.aspx) | | |✔ |
| [Cyberduck](https://cyberduck.io/) | |✔ |✔ |


## <a name="mount-azure-storage-as-local-drive"></a>将 Azure 存储装载为本地驱动器
请参阅[将 Azure 存储装载为本地驱动器](https://blogs.msdn.com/b/bigdatasupport/archive/2014/01/09/mount-azure-blob-storage-as-local-drive.aspx)。

## <a name="upload-using-services"></a>使用服务上传
### <a id="sqoop"></a>Apache Sqoop
Sqoop 是一种专用于在 Hadoop 和关系数据库之间传输数据的工具。 可以使用此工具将数据从关系数据库管理系统 (RDBMS)（如 SQL Server、MySQL 或 Oracle）中导入到 Hadoop 分布式文件系统 (HDFS)，在 Hadoop 中使用 MapReduce 或 Hive 转换数据，然后回过来将数据导出到 RDBMS。

有关详细信息，请参阅[将 Sqoop 与 HDInsight 配合使用][hdinsight-use-sqoop]。

### <a name="development-sdks"></a>开发 SDK
还可以使用 Azure SDK 通过以下编程语言来访问 Azure 存储：

* .NET
* Java
* Node.js
* PHP
* Python
* Ruby

有关安装 Azure SDK 的详细信息，请参阅 [Azure 下载](https://azure.microsoft.com/downloads/)

## <a name="troubleshooting"></a>故障排除
### <a id="storageexception"></a>写入 blob 时的存储异常
**症状**：使用 `hadoop` 或 `hdfs dfs` 命令在 HBase 群集上编写大于或等于 ~12 GB 的文件时，可能会遇到以下错误：

    ERROR azure.NativeAzureFileSystem: Encountered Storage Exception for write on Blob : example/test_large_file.bin._COPYING_ Exception details: null Error Code : RequestBodyTooLarge
    copyFromLocal: java.io.IOException
            at com.microsoft.azure.storage.core.Utility.initIOException(Utility.java:661)
            at com.microsoft.azure.storage.blob.BlobOutputStream$1.call(BlobOutputStream.java:366)
            at com.microsoft.azure.storage.blob.BlobOutputStream$1.call(BlobOutputStream.java:350)
            at java.util.concurrent.FutureTask.run(FutureTask.java:262)
            at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
            at java.util.concurrent.FutureTask.run(FutureTask.java:262)
            at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
            at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
            at java.lang.Thread.run(Thread.java:745)
    Caused by: com.microsoft.azure.storage.StorageException: The request body is too large and exceeds the maximum permissible limit.
            at com.microsoft.azure.storage.StorageException.translateException(StorageException.java:89)
            at com.microsoft.azure.storage.core.StorageRequest.materializeException(StorageRequest.java:307)
            at com.microsoft.azure.storage.core.ExecutionEngine.executeWithRetry(ExecutionEngine.java:182)
            at com.microsoft.azure.storage.blob.CloudBlockBlob.uploadBlockInternal(CloudBlockBlob.java:816)
            at com.microsoft.azure.storage.blob.CloudBlockBlob.uploadBlock(CloudBlockBlob.java:788)
            at com.microsoft.azure.storage.blob.BlobOutputStream$1.call(BlobOutputStream.java:354)
            ... 7 more

**原因**：HBase on HDInsight 群集在写入 Azure 存储时默认阻止 256KB 大小的块。 尽管这对 HBase API 或 REST API 来说可良好运行，但使用 `hadoop` 或 `hdfs dfs` 命令行实用工具时则会导致错误。

**解决方法**：使用 `fs.azure.write.request.size` 指定更大的块大小。 可以使用 `-D` 参数基于使用情况执行此操作。 以下命令是将此参数用于 `hadoop` 命令的示例：

```bash
hadoop -fs -D fs.azure.write.request.size=4194304 -copyFromLocal test_large_file.bin /example/data
```

还可使用 Apache Ambari 以全局方式增加 `fs.azure.write.request.size` 的值。 可以使用以下步骤在 Ambari Web UI 中更改该值：

1. 在浏览器中，转到群集的 Ambari Web UI。 该地址为 https://CLUSTERNAME.azurehdinsight.net ，其中“CLUSTERNAME”是群集名称  。

    出现提示时，输入群集的管理员名称和密码。
2. 在屏幕左侧选择“HDFS”，然后选择“配置”选项卡   。
3. 在“筛选...”字段中输入 `fs.azure.write.request.size`  。 这会在页面中间显示字段和当前值。
4. 将值从 262144 (256KB) 更改为新的值。 例如，4194304 (4MB)。

![通过 Ambari Web UI 更改值的图像](./media/hdinsight-upload-data/hbase-change-block-write-size.png)

有关如何使用 Ambari 的详细信息，请参阅[使用 Apache Ambari Web UI 管理 HDInsight 群集](hdinsight-hadoop-manage-ambari.md)。

## <a name="next-steps"></a>后续步骤
现在，已了解如何将数据导入 HDInsight，请阅读以下文章了解如何执行分析：

* [Azure HDInsight 入门][hdinsight-get-started]
* [以编程方式提交 Apache Hadoop 作业][hdinsight-submit-jobs]
* [将 Apache Hive 和 HDInsight 配合使用][hdinsight-use-hive]
* [将 Apache Pig 和 HDInsight 配合使用][hdinsight-use-pig]

[azure-management-portal]: https://porta.azure.com
[azure-powershell]: https://msdn.microsoft.com/library/windowsazure/jj152841.aspx

[azure-storage-client-library]: https://docs.microsoft.com/develop/net/how-to-guides/blob-storage/
[azure-create-storage-account]:../storage/common/storage-create-storage-account.md
[azure-azcopy-download]:../storage/common/storage-use-azcopy.md
[azure-azcopy]:../storage/common/storage-use-azcopy.md

[hdinsight-use-sqoop]:hadoop/hdinsight-use-sqoop.md

[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-submit-jobs]:hadoop/submit-apache-hadoop-jobs-programmatically.md
[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md

[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-use-pig]:hadoop/hdinsight-use-pig.md

[sqldatabase-create-configure]: ../sql-database-create-configure.md

[apache-sqoop-guide]: https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html

[Powershell-install-configure]: /powershell/azureps-cmdlets-docs

[azurecli]: ../cli-install-nodejs.md


[image-azure-storage-explorer]: ./media/hdinsight-upload-data/HDI.AzureStorageExplorer.png
[image-ase-addaccount]: ./media/hdinsight-upload-data/HDI.ASEAddAccount.png
[image-ase-blob]: ./media/hdinsight-upload-data/HDI.ASEBlob.png
