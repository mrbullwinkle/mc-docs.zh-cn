---
title: 快速入门：直译文本，Python - 文本翻译 API
titleSuffix: Azure Cognitive Services
description: 本快速入门介绍如何使用 Python 和文本翻译 REST API 将文本从一个脚本直译（转换）为另一个脚本。 在此示例中，日语在经过直译后使用拉丁字母。
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: quickstart
origin.date: 06/04/2019
ms.date: 07/23/2019
ms.author: v-junlch
ms.openlocfilehash: 150d33f87cd9eed7b65dedb32760ee309de11c42
ms.sourcegitcommit: 9a330fa5ee7445b98e4e157997e592a0d0f63f4c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/24/2019
ms.locfileid: "68439958"
---
# <a name="quickstart-use-the-translator-text-api-to-transliterate-text-using-python"></a>快速入门：使用文本翻译 API 通过 Python 对文本进行直译

本快速入门介绍如何使用 Python 和文本翻译 REST API 将文本从一个脚本直译（转换）为另一个脚本。 在提供的示例中，日语在经过直译后使用拉丁字母。

此快速入门需要包含文本翻译资源的 [Azure 认知服务帐户](/cognitive-services/cognitive-services-apis-create-account)。 如果没有帐户，可以使用[试用帐户](https://www.azure.cn/pricing/1rmb-trial/)获取订阅密钥。

>[!TIP]
> 如果你想一次看到所有代码，这个示例的源代码可以在 [GitHub]() 上找到。

## <a name="prerequisites"></a>先决条件

本快速入门需要：

* Python 2.7.x 或 3.x
* 适用于文本翻译的 Azure 订阅密钥

## <a name="create-a-project-and-import-required-modules"></a>创建一个项目并导入必需的模块

使用你喜欢的 IDE 或编辑器创建一个新项目，或者在桌面上创建一个包含名为 `transliterate-text.py` 的文件的新文件夹。 然后将以下代码片段复制到项目/文件中：

```python
# -*- coding: utf-8 -*-
import os
import requests
import uuid
import json
```

> [!NOTE]
> 如果尚未使用这些模块，则需在运行程序之前安装它们。 若要安装这些包，请运行 `pip install requests uuid`。

第一个注释告知 Python 解释器使用 UTF-8 编码。 然后导入必需的模块，以便从环境变量读取订阅密钥、构造 HTTP 请求、创建唯一标识符，以及处理文本翻译 API 返回的 JSON 响应。

## <a name="set-the-subscription-key-base-url-and-path"></a>设置订阅密钥、基 URL 和路径

此示例将尝试从环境变量 `TRANSLATOR_TEXT_KEY` 读取文本翻译订阅密钥。 如果不熟悉环境变量，则可将 `subscriptionKey` 设置为字符串并注释掉条件语句。

将以下代码复制到项目中：

```python
# Checks to see if the Translator Text subscription key is available
# as an environment variable. If you are setting your subscription key as a
# string, then comment these lines out.
if 'TRANSLATOR_TEXT_KEY' in os.environ:
    subscriptionKey = os.environ['TRANSLATOR_TEXT_KEY']
else:
    print('Environment variable for TRANSLATOR_TEXT_KEY is not set.')
    exit()
# If you want to set your subscription key as a string, uncomment the line
# below and add your subscription key.
#subscriptionKey = 'put_your_key_here'
region = 'put_your_region_here'
```

文本翻译全局终结点设置为 `base_url`。 `path` 设置 `transliterate` 路由并确定我们需使用 API 的版本 3。

`params` 用于设置输入语言以及输入和输出脚本。 在此示例中，我们从日语直译为拉丁字母。

>[!NOTE]
> 有关终结点、路由和请求参数的详细信息，请参阅[文本翻译 API 3.0：直译](/cognitive-services/translator/reference/v3-0-transliterate)。

```python
base_url = 'https://api.translator.azure.cn'
path = '/transliterate?api-version=3.0'
params = '&language=ja&fromScript=jpan&toScript=latn'
constructed_url = base_url + path + params
```

## <a name="add-headers"></a>添加标头

若要对请求进行身份验证，最容易的方法是将订阅密钥作为 `Ocp-Apim-Subscription-Key` 标头传入，这是我们在此示例中使用的方法。 替代方法是交换订阅密钥来获取访问令牌，将访问令牌作为 `Authorization` 标头传入，以便对请求进行验证。 有关详细信息，请参阅[身份验证](/cognitive-services/translator/reference/v3-0-reference#authentication)。

将以下代码片段复制到项目中。

```python
headers = {
    'Ocp-Apim-Subscription-Key': subscriptionKey,
    'Ocp-Apim-Subscription-Region': region,
    'Content-type': 'application/json',
    'X-ClientTraceId': str(uuid.uuid4())
}
```

如果使用的是认知服务多服务订阅，则还必须在请求参数中包括 `Ocp-Apim-Subscription-Region`。 [详细了解如何使用多服务订阅进行身份验证](/cognitive-services/translator/reference/v3-0-reference#authentication)。

## <a name="create-a-request-to-transliterate-text"></a>创建文本直译请求

定义要直译的字符串：

```python
# Transliterate "good afternoon" from source Japanese.
# Note: You can pass more than one object in body.
body = [{
    'text': 'こんにちは'
}]
```

接下来，请使用 `requests` 模块创建一个 POST 请求。 它使用三个参数：串联的 URL、请求标头和请求正文：

```python
request = requests.post(constructed_url, headers=headers, json=body)
response = request.json()
```

## <a name="print-the-response"></a>输出响应

最后一步是输出结果。 以下代码片段通过将密钥排序、设置缩进以及声明项和密钥分隔符来美化结果。

```python
print(json.dumps(response, sort_keys=True, indent=4,
                 ensure_ascii=False, separators=(',', ': ')))
```

## <a name="put-it-all-together"></a>将其放在一起

就是这样，你已构建了一个简单的程序。该程序可以调用文本翻译 API 并返回 JSON 响应。 现在，可以运行该程序了：

```console
python transliterate-text.py
```

如果希望将你的代码与我们的进行比较，请查看 [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Python) 上提供的完整示例。

## <a name="sample-response"></a>示例响应

```json
[
    {
        "script": "latn",
        "text": "konnichiwa"
    }
]
```

## <a name="clean-up-resources"></a>清理资源

如果已将订阅密钥硬编码到程序中，请确保在完成本快速入门后删除该订阅密钥。

## <a name="next-steps"></a>后续步骤

查看 API 参考，了解使用文本翻译 API 可以执行的所有操作。

> [!div class="nextstepaction"]
> [API 参考](/cognitive-services/translator/reference/v3-0-reference)

## <a name="see-also"></a>另请参阅

了解如何使用文本翻译 API 执行以下操作：

* [翻译文本](quickstart-python-translate.md)
* [按输入确定语言](quickstart-python-detect.md)
* [获取备用翻译](quickstart-python-dictionary.md)
* [获取支持的语言的列表](quickstart-python-languages.md)
* [根据输入确定句子长度](quickstart-python-sentences.md)

<!-- Update_Description: wording update -->