---
title: 快速入门：使用双语字典、Go 查找字词 - 文本翻译 API
titleSuffix: Azure Cognitive Services
description: 在本快速入门中，你将使用文本翻译 API 和 Go 查找字词的备用翻译和示例。
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: quickstart
origin.date: 06/04/2019
ms.date: 07/23/2019
ms.author: v-junlch
ms.openlocfilehash: 9d1d02382d3eed70afc3ec78c88dda7a2ae0e687
ms.sourcegitcommit: 9a330fa5ee7445b98e4e157997e592a0d0f63f4c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/24/2019
ms.locfileid: "68439969"
---
# <a name="quickstart-look-up-words-with-bilingual-dictionary-using-go"></a>快速入门：通过 Go 使用双语字典查找字词

本快速入门介绍如何使用 Go 和文本翻译 REST API 查找指定文本的备用翻译和用法示例。

此快速入门需要包含文本翻译资源的 [Azure 认知服务帐户](/cognitive-services/cognitive-services-apis-create-account)。 如果没有帐户，可以使用[试用帐户](https://www.azure.cn/pricing/1rmb-trial/)获取订阅密钥。

>[!TIP]
> 如果你想一次看到所有代码，这个示例的源代码可以在 [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Go) 上找到。

## <a name="prerequisites"></a>先决条件

本快速入门需要：

* [Go](https://golang.org/doc/install)
* 适用于文本翻译的 Azure 订阅密钥

## <a name="create-a-project-and-import-required-modules"></a>创建一个项目并导入必需的模块

使用喜欢的 IDE/编辑器创建新的 Go 项目，或在桌面上创建新的文件夹。 然后，将此代码片段复制到项目/文件夹的 `alt-translations.go` 文件中。

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "net/url"
    "os"
)
```

## <a name="create-the-main-function"></a>创建 main 函数

此示例将尝试从环境变量 `TRANSLATOR_TEXT_KEY` 读取文本翻译订阅密钥。 如果不熟悉环境变量，则可将 `subscriptionKey` 设置为字符串并注释掉条件语句。

将以下代码复制到项目中：

```go
func main() {
    /*
     * Read your subscription key from an env variable.
     * Please note: You can replace this code block with
     * var subscriptionKey = "YOUR_SUBSCRIPTION_KEY" if you don't
     * want to use env variables. Then, be sure to delete the "os" import.
     */
    subscriptionKey := os.Getenv("TRANSLATOR_TEXT_KEY")
    if subscriptionKey == "" {
       log.Fatal("Environment variable TRANSLATOR_TEXT_KEY is not set.")
    }
    /*
     * This calls our altTranslations function, which we'll
     * create in the next section. It takes a single argument,
     * the subscription key.
     */
    altTranslations(subscriptionKey)
}
```

## <a name="create-a-function-to-get-alternate-translations"></a>创建获取备用翻译的函数

让我们创建获取备用翻译的函数。 此函数将接受一个参数，即文本翻译订阅密钥。

```go
func altTranslations(subscriptionKey string) {
    /*  
     * In the next few sections, we'll add code to this
     * function to make a request and handle the response.
     */
}
```

接下来，构造 URL。 使用 `Parse()` 和 `Query()` 方法生成 URL。 你将注意到参数是通过 `Add()` 方法添加的。 在此示例中，我们将从英语翻译为西班牙语。

将此代码复制到 `altTranslations` 函数中。

```go
// Build the request URL. See: https://golang.org/pkg/net/url/#example_URL_Parse
u, _ := url.Parse("https://api.translator.azure.cn/dictionary/lookup?api-version=3.0")
q := u.Query()
q.Add("from", "en")
q.Add("to", "es")
u.RawQuery = q.Encode()
```

>[!NOTE]
> 有关终结点、路由和请求参数的详细信息，请参阅[文本翻译 API 3.0：字典查找](/cognitive-services/translator/reference/v3-0-dictionary-lookup)。

## <a name="create-a-struct-for-your-request-body"></a>创建请求正文的结构

接下来，创建请求正文的匿名结构，并使用 `json.Marshal()` 将其编码为 JSON。 将此代码添加到 `altTranslations` 函数中。

```go
// Create an anonymous struct for your request body and encode it to JSON
body := []struct {
    Text string
}{
    {Text: "Pineapples"},
}
b, _ := json.Marshal(body)
```

## <a name="build-the-request"></a>生成请求

将请求正文编码为 JSON 后，可以生成 POST 请求并调用文本翻译 API。

```go
// Build the HTTP POST request
req, err := http.NewRequest("POST", u.String(), bytes.NewBuffer(b))
if err != nil {
    log.Fatal(err)
}
// Add required headers to the request
req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
req.Header.Add("Ocp-Apim-Subscription-Region", "your region")
req.Header.Add("Content-Type", "application/json")

// Call the Translator Text API
res, err := http.DefaultClient.Do(req)
if err != nil {
    log.Fatal(err)
}
```

如果使用的是认知服务多服务订阅，则还必须在请求参数中包括 `Ocp-Apim-Subscription-Region`。 [详细了解如何使用多服务订阅进行身份验证](/cognitive-services/translator/reference/v3-0-reference#authentication)。

## <a name="handle-and-print-the-response"></a>处理并输出响应

将此代码添加到 `altTranslations` 函数以解码 JSON 响应，然后格式化并输出结果。

```go
// Decode the JSON response
var result interface{}
if err := json.NewDecoder(res.Body).Decode(&result); err != nil {
    log.Fatal(err)
}
// Format and print the response to terminal
prettyJSON, _ := json.MarshalIndent(result, "", "  ")
fmt.Printf("%s\n", prettyJSON)
```

## <a name="put-it-all-together"></a>将其放在一起

就是这样，你已构建了一个简单的程序。该程序可以调用文本翻译 API 并返回 JSON 响应。 现在，可以运行该程序了：

```console
go run alt-translations.go
```

如果希望将你的代码与我们的进行比较，请查看 [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Go) 上提供的完整示例。

## <a name="sample-response"></a>示例响应

```json
[
  {
    "displaySource": "pineapples",
    "normalizedSource": "pineapples",
    "translations": [
      {
        "backTranslations": [
          {
            "displayText": "pineapples",
            "frequencyCount": 158,
            "normalizedText": "pineapples",
            "numExamples": 5
          },
          {
            "displayText": "cones",
            "frequencyCount": 13,
            "normalizedText": "cones",
            "numExamples": 5
          },
          {
            "displayText": "piña",
            "frequencyCount": 5,
            "normalizedText": "piña",
            "numExamples": 3
          },
          {
            "displayText": "ganks",
            "frequencyCount": 3,
            "normalizedText": "ganks",
            "numExamples": 2
          }
        ],
        "confidence": 0.7016,
        "displayTarget": "piñas",
        "normalizedTarget": "piñas",
        "posTag": "NOUN",
        "prefixWord": ""
      },
      {
        "backTranslations": [
          {
            "displayText": "pineapples",
            "frequencyCount": 16,
            "normalizedText": "pineapples",
            "numExamples": 2
          }
        ],
        "confidence": 0.2984,
        "displayTarget": "ananás",
        "normalizedTarget": "ananás",
        "posTag": "NOUN",
        "prefixWord": ""
      }
    ]
  }
]
```

## <a name="next-steps"></a>后续步骤

查看 API 参考，了解使用文本翻译 API 可以执行的所有操作。

> [!div class="nextstepaction"]
> [API 参考](/cognitive-services/translator/reference/v3-0-reference)

## <a name="see-also"></a>另请参阅

了解如何使用文本翻译 API 执行以下操作：

* [翻译文本](quickstart-go-translate.md)
* [直译文本](quickstart-go-transliterate.md)
* [按输入确定语言](quickstart-go-detect.md)
* [获取支持的语言的列表](quickstart-go-languages.md)
* [根据输入确定句子长度](quickstart-go-sentences.md)

<!-- Update_Description: wording update -->