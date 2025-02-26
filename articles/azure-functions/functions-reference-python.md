---
title: Azure Functions Python 开发人员参考
description: 了解如何使用 Python 开发函数
services: functions
documentationcenter: na
author: ggailey777
manager: cfowler
keywords: Azure Functions, 函数, 事件处理, 动态计算, 无服务体系结构, python
ms.service: azure-functions
ms.devlang: python
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
origin.date: 04/16/2018
ms.date: 07/18/2019
ms.author: v-junlch
ms.openlocfilehash: e4cd6696f5aa522950256d6171fe8a9a264b9bf3
ms.sourcegitcommit: c61b10764d533c32d56bcfcb4286ed0fb2bdbfea
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68331887"
---
# <a name="azure-functions-python-developer-guide"></a>Azure Functions Python 开发人员指南

本文介绍了如何使用 Python 开发 Azure Functions。 以下内容假定你已阅读 [Azure Functions 开发人员指南](functions-reference.md)。

[!INCLUDE [functions-python-preview-note](../../includes/functions-python-preview-note.md)]

## <a name="programming-model"></a>编程模型

Azure 函数应是 Python 脚本中处理输入并生成输出的无状态方法。 默认情况下，运行时期望此方法在 `__init__.py` 文件中作为名为 `main()` 的全局方法实现。

可以通过在 *function.json* 文件中指定 `scriptFile` 和 `entryPoint` 属性来更改默认配置。 例如，下面的 _function.json_ 指示运行时使用 _main.py_ 文件中的 `customentry()` 方法作为 Azure 函数的入口点。

```json
{
  "scriptFile": "main.py",
  "entryPoint": "customentry",
  ...
}
```

来自触发器和绑定的数据使用在 *function.json* 配置文件中定义的 `name` 属性，通过方法特性绑定到函数。 例如，下面的 _function.json_ 描述一个由名为 `req` 的 HTTP 请求触发的简单函数：

```json
{
  "bindings": [
    {
      "name": "req",
      "direction": "in",
      "type": "httpTrigger",
      "authLevel": "anonymous"
    },
    {
      "name": "$return",
      "direction": "out",
      "type": "http"
    }
  ]
}
```

`__init__.py` 文件包含以下函数代码：

```python
def main(req):
    user = req.params.get('user')
    return f'Hello, {user}!'
```

（可选）若要利用代码编辑器提供的 Intellisense 和自动完成功能，还可以使用 Python 类型注释来声明函数中的属性类型和返回类型。 

```python
import azure.functions


def main(req: azure.functions.HttpRequest) -> str:
    user = req.params.get('user')
    return f'Hello, {user}!'
```

使用 [ azure.functions.*](https://docs.microsoft.com/python/api/azure-functions/azure.functions?view=azure-python) 包中附带的 Python 注释将输入和输出绑定到方法。 

## <a name="folder-structure"></a>文件夹结构

Python 函数项目的文件夹结构如下所示：

```
 FunctionApp
 | - MyFirstFunction
 | | - __init__.py
 | | - function.json
 | - MySecondFunction
 | | - __init__.py
 | | - function.json
 | - SharedCode
 | | - myFirstHelperFunction.py
 | | - mySecondHelperFunction.py
 | - host.json
 | - requirements.txt
```

有一个共享的 [host.json](functions-host-json.md) 文件，可用于配置函数应用。 每个函数都有自己的代码文件和绑定配置文件 (function.json)。 

共享代码应保留在单独的文件夹中。 若要引用 SharedCode 文件夹中的模块，可以使用以下语法：

```
from __app__.SharedCode import myFirstHelperFunction
```

在 Azure 中将 Functions 项目部署到函数应用时，*FunctionApp* 文件夹的整个内容应包含在包中，但不包含该文件夹本身。

## <a name="triggers-and-inputs"></a>触发器和输入

在 Azure Functions 中，输入分为两种类别：触发器输入和附加输入。 虽然它们在 `function.json` 文件中并不相同，但它们在 Python 代码中的使用方法却是相同的。  在本地运行时，触发器和输入源的连接字符串或机密应映射到 `local.settings.json` 文件中的值，以及在 Azure 中运行时的应用程序设置。 

例如，以下代码演示了这两个类别之间的差异：

```json
// function.json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "req",
      "direction": "in",
      "type": "httpTrigger",
      "authLevel": "anonymous",
      "route": "items/{id}"
    },
    {
      "name": "obj",
      "direction": "in",
      "type": "blob",
      "path": "samples/{id}",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

```json
// local.settings.json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "AzureWebJobsStorage": "<azure-storage-connection-string>"
  }
}
```

```python
# __init__.py
import azure.functions as func
import logging


def main(req: func.HttpRequest,
         obj: func.InputStream):

    logging.info(f'Python HTTP triggered function processed: {obj.read()}')
```

调用函数时，HTTP 请求作为 `req` 传递给函数。 将基于路由 URL 中的 _ID_ 从 Azure Blob 存储检索一个条目，并在函数体中将其用作 `obj`。  在这里，指定的存储帐户是在 `AzureWebJobsStorage` 中找到的连接字符串，它与函数应用使用的存储帐户相同。


## <a name="outputs"></a>Outputs

输出可以在返回值和输出参数中进行表示。 如果只有一个输出，则建议使用返回值。 对于多个输出，必须使用输出参数。

若要使用函数的返回值作为输出绑定的值，则绑定的 `name` 属性应在 `function.json` 中设置为 `$return`。

若要生成多个输出，请使用 [`azure.functions.Out`](https://docs.microsoft.com/python/api/azure-functions/azure.functions.out?view=azure-python) 接口提供的 `set()` 方法将值分配给绑定。 例如，以下函数可以将消息推送到队列，还可返回 HTTP 响应。

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "req",
      "direction": "in",
      "type": "httpTrigger",
      "authLevel": "anonymous"
    },
    {
      "name": "msg",
      "direction": "out",
      "type": "queue",
      "queueName": "outqueue",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "$return",
      "direction": "out",
      "type": "http"
    }
  ]
}
```

```python
import azure.functions as func


def main(req: func.HttpRequest,
         msg: func.Out[func.QueueMessage]) -> str:

    message = req.params.get('body')
    msg.set(message)
    return message
```

## <a name="async"></a>异步

建议使用 `async def` 语句将 Azure 函数编写为异步协同例程。

```python
# Will be run with asyncio directly


async def main():
    await some_nonblocking_socket_io_op()
```

如果 main() 函数是同步函数（无 `async` 限定符），则我们会在 `asyncio` 线程池中自动运行该函数。

```python
# Would be run in an asyncio thread-pool


def main():
    some_blocking_socket_io()
```

## <a name="context"></a>上下文

若要在执行过程中获取函数的调用上下文，请在其签名中包含 [`context`](https://docs.microsoft.com/python/api/azure-functions/azure.functions.context?view=azure-python) 参数。 

例如：

```python
import azure.functions


def main(req: azure.functions.HttpRequest,
         context: azure.functions.Context) -> str:
    return f'{context.invocation_id}'
```

[**Context**](https://docs.microsoft.com/python/api/azure-functions/azure.functions.context?view=azure-python) 类具有以下方法：

`function_directory`  
在其中运行函数的目录。

`function_name`  
函数的名称。

`invocation_id`  
当前函数调用的 ID。

## <a name="global-variables"></a>全局变量

不保证应用的状态可保留到将来的执行。 不过，Azure Functions 运行时通常会重复使用同一个进程来多次执行同一个应用。 为了缓存高开销计算的结果，请将其声明为全局变量。 

```python
CACHED_DATA = None


def main(req):
    global CACHED_DATA
    if CACHED_DATA is None:
        CACHED_DATA = load_json()

    # ... use CACHED_DATA in code
```

## <a name="python-version-and-package-management"></a>Python 版本和包管理

当前，Azure Functions 仅支持 Python 3.6.x（CPython 正式分发版）。

在使用 Azure Functions Core Tools 或 Visual Studio Code 进行本地开发时，将所需包的名称和版本添加到 `requirements.txt` 文件并使用 `pip` 安装它们。

例如，可以使用以下要求文件和 pip 命令从 PyPI 安装 `requests` 包。

```txt
requests==2.19.1
```

```bash
pip install -r requirements.txt
```

## <a name="publishing-to-azure"></a>发布到 Azure

准备好进行发布时，请确保所有依赖项都已在 *requirements.txt* 文件（位于项目目录的根目录）中列出。 如果使用的包需要编译器并且不支持从 PyPI 安装 manylinux 兼容滚轮的包，则发布到 Azure 会失败并出现以下错误： 

```
There was an error restoring dependencies.ERROR: cannot install <package name - version> dependency: binary dependencies without wheels are not supported.  
The terminal process terminated with exit code: 1
```

若要自动生成并配置所需二进制文件，请在本地计算机上[安装 Docker](https://docs.docker.com/install/)并运行以下命令以使用 [Azure Functions Core Tools](functions-run-local.md#v2) (func) 进行发布。 请记住将 `<app name>` 替换为 Azure 中的函数应用名称。 

```bash
func azure functionapp publish <app name> --build-native-deps
```

实际上，Core Tools 会使用 docker 将 [mcr.microsoft.com/azure-functions/python](https://hub.docker.com/r/microsoft/azure-functions/) 映像作为本地计算机上的容器来运行。 它随后会使用此环境从源分发版生成并安装所需模块，然后再打包它们以便最终部署到 Azure。

## <a name="unit-testing"></a>单元测试

可以使用标准测试框架，像测试其他 Python 代码一样测试以 Python 编写的函数。 对于大多数绑定，可以通过创建 `azure.functions` 包中相应类的实例来创建模拟输入对象。 由于不能直接使用 [`azure.functions`](https://pypi.org/project/azure-functions/) 包，请务必根据前面的 [Python 版本和包管理](#python-version-and-package-management)部分中所述，通过 `requirements.txt` 文件安装该包。

例如，下面是 HTTP 触发的函数的模拟测试：

```json
{
  "scriptFile": "httpfunc.py",
  "entryPoint": "my_function",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

```python
# myapp/httpfunc.py
import azure.functions as func
import logging

def my_function(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}")
    else:
        return func.HttpResponse(
             "Please pass a name on the query string or in the request body",
             status_code=400
        )
```

```python
# myapp/test_httpfunc.py
import unittest

import azure.functions as func
from httpfunc import my_function


class TestFunction(unittest.TestCase):
    def test_my_function(self):
        # Construct a mock HTTP request.
        req = func.HttpRequest(
            method='GET',
            body=None,
            url='/api/HttpTrigger',
            params={'name': 'Test'})

        # Call the function.
        resp = my_function(req)

        # Check the output.
        self.assertEqual(
            resp.get_body(),
            b'Hello Test',
        )
```

下面是使用队列触发的函数的另一个示例：

```python
# myapp/__init__.py
import azure.functions as func


def my_function(msg: func.QueueMessage) -> str:
    return f'msg body: {msg.get_body().decode()}'
```

```python
# myapp/test_func.py
import unittest

import azure.functions as func
from . import my_function


class TestFunction(unittest.TestCase):
    def test_my_function(self):
        # Construct a mock Queue message.
        req = func.QueueMessage(
            body=b'test')

        # Call the function.
        resp = my_function(req)

        # Check the output.
        self.assertEqual(
            resp,
            'msg body: test',
        )
```

## <a name="known-issues-and-faq"></a>已知问题和常见问题解答

所有已知问题和功能请求都使用 [GitHub 问题](https://github.com/Azure/azure-functions-python-worker/issues)列表进行跟踪。 如果遇到 GitHub 中未列出的问题，请打开“新问题”并提供问题的详细说明。

## <a name="next-steps"></a>后续步骤

有关详细信息，请参阅以下资源：

* [Azure Functions 包 API 文档](https://docs.microsoft.com/python/api/azure-functions/azure.functions?view=azure-python)
* [Azure Functions 最佳实践](functions-best-practices.md)
* [Azure Functions 触发器和绑定](functions-triggers-bindings.md)
* [Blob 存储绑定](functions-bindings-storage-blob.md)
* [HTTP 和 Webhook 绑定](functions-bindings-http-webhook.md)
* [存储绑定](functions-bindings-storage-queue.md)
* [计时器触发器](functions-bindings-timer.md)

<!-- Update_Description: wording update -->