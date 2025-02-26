---
title: 快速入门：检测文本语言，C# - 文本翻译 API
titleSuffix: Azure Cognitive Services
description: 本快速入门介绍如何使用 .NET Core 和文本翻译 REST API 来检测所提供文本的语言。
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: quickstart
origin.date: 06/13/2019
ms.date: 07/23/2019
ms.author: v-junlch
ms.openlocfilehash: f09276be958f23438ac2ec4b20776c9f2bbc883c
ms.sourcegitcommit: 9a330fa5ee7445b98e4e157997e592a0d0f63f4c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/24/2019
ms.locfileid: "68439943"
---
# <a name="quickstart-use-the-translator-text-api-to-detect-text-language-using-c"></a>快速入门：使用文本翻译 API 通过 C# 来检测文本语言

本快速入门介绍如何使用 .NET Core、C# 7.1 或更高版本和文本翻译 REST API 来检测所提供文本的语言。

此快速入门需要包含文本翻译资源的 [Azure 认知服务帐户](/cognitive-services/cognitive-services-apis-create-account)。 如果没有帐户，可以使用[试用帐户](https://www.azure.cn/pricing/1rmb-trial/)获取订阅密钥。

>[!TIP]
> 如果你想一次看到所有代码，这个示例的源代码可以在 [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-C-Sharp) 上找到。

## <a name="prerequisites"></a>先决条件

* C# 7.1 或更高版本
* [.NET SDK](https://www.microsoft.com/net/learn/dotnet/hello-world-tutorial)
* [Json.NET NuGet 包](https://www.nuget.org/packages/Newtonsoft.Json/)
* [Visual Studio](https://visualstudio.microsoft.com/downloads/)、[Visual Studio Code](https://code.visualstudio.com/download) 或你喜欢用的文本编辑器
* 适用于文本翻译的 Azure 订阅密钥

## <a name="create-a-net-core-project"></a>创建 .NET Core 项目

打开新的命令提示符（或终端会话）并运行以下命令：

```console
dotnet new console -o detect-sample
cd detect-sample
```

第一个命令执行两项操作。 它创建新的 .NET 控制台应用程序，并创建名为 `detect-sample` 的目录。 第二个目录转到项目的目录。

接下来需安装 Json.Net。 在项目的目录中，运行以下命令：

```console
dotnet add package Newtonsoft.Json --version 11.0.2
```

## <a name="select-the-c-language-version"></a>选择 C# 语言版本

本快速入门需要 C# 7.1 或更高版本。 可以通过多种方式来更改项目的 C# 版本。 本指南将介绍如何调整 `detect-sample.csproj` 文件。 如需所有可用选项，例如在 Visual Studio 中更改语言的选项，请参阅[选择 C# 语言版本](https://docs.microsoft.com/dotnet/csharp/language-reference/configure-language-version)。

打开项目，然后打开 `detect-sample.csproj`。 确保将 `LangVersion` 设置为 7.1 或更高版本。 如果没有用于语言版本的属性组，请添加以下行：

```xml
<PropertyGroup>
   <LangVersion>7.1</LangVersion>
</PropertyGroup>
```

## <a name="add-required-namespaces-to-your-project"></a>将所需命名空间添加到项目

此前运行的 `dotnet new console` 命令创建了一个项目（包括 `Program.cs`）。 此文件是需放置应用程序代码的位置。 打开 `Program.cs`，替换现有的 using 语句。 这些语句可确保你有权访问生成并运行示例应用所需的所有类型。

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
// Install Newtonsoft.Json with NuGet
using Newtonsoft.Json;
```

## <a name="create-classes-for-the-json-response"></a>为 JSON 响应创建类

接下来，我们将创建一个在反序列化由文本翻译 API 返回的 JSON 响应时使用的类。

```csharp
/// <summary>
/// The C# classes that represents the JSON returned by the Translator Text API.
/// </summary>
public class DetectResult
{
    public string Language { get; set; }
    public float Score { get; set; }
    public bool IsTranslationSupported { get; set; }
    public bool IsTransliterationSupported { get; set; }
    public AltTranslations[] Alternatives { get; set; }
}
public class AltTranslations
{
    public string Language { get; set; }
    public float Score { get; set; }
    public bool IsTranslationSupported { get; set; }
    public bool IsTransliterationSupported { get; set; }
}
```

## <a name="create-a-function-to-detect-the-source-texts-language"></a>创建源文本语言检测函数

在 `Program` 类中创建名为 `DetectTextRequest()` 的函数。 该类封装用于调用 Detect 资源的代码，并将结果输出到控制台。

```csharp
static public async Task DetectTextRequest(string subscriptionKey, string host, string route, string inputText)
{
  /*
   * The code for your call to the translation service will be added to this
   * function in the next few sections.
   */
}
```

## <a name="serialize-the-detect-request"></a>序列化检测请求

接下来需创建 JSON 对象并将其序列化，其中包含要进行语言检测的文本。

```csharp
System.Object[] body = new System.Object[] { new { Text = inputText } };
var requestBody = JsonConvert.SerializeObject(body);
```

## <a name="instantiate-the-client-and-make-a-request"></a>实例化客户端并发出请求

以下行实例化 `HttpClient` 和 `HttpRequestMessage`：

```csharp
using (var client = new HttpClient())
using (var request = new HttpRequestMessage())
{
  // In the next few sections you'll add code to construct the request.
}
```

## <a name="construct-the-request-and-print-the-response"></a>构造请求并输出响应

在 `HttpRequestMessage` 中，需执行以下操作：

* 声明 HTTP 方法
* 构造请求 URI
* 插入请求正文（序列化的 JSON 对象）
* 添加必需的标头
* 发出异步请求
* 输出响应

向 `HttpRequestMessage` 添加以下代码：

```csharp
// Build the request.
request.Method = HttpMethod.Post;
// Construct the URI and add headers.
request.RequestUri = new Uri(host + route);
request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
request.Headers.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
request.Headers.Add("Ocp-Apim-Subscription-Region", "your region");

// Send the request and get response.
HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
// Read response as a string.
string result = await response.Content.ReadAsStringAsync();
// Deserialize the response using the classes created earlier.
DetectResult[] deserializedOutput = JsonConvert.DeserializeObject<DetectResult[]>(result);
// Iterate over the deserialized response.
foreach (DetectResult o in deserializedOutput)
{
    Console.WriteLine("The detected language is '{0}'. Confidence is: {1}.\nTranslation supported: {2}.\nTransliteration supported: {3}.\n",
        o.Language, o.Score, o.IsTranslationSupported, o.IsTransliterationSupported);
    // Create a counter
    int counter = 0;
    // Iterate over alternate translations.
    foreach (AltTranslations a in o.Alternatives)
    {
        counter++;
        Console.WriteLine("Alternative {0}", counter);
        Console.WriteLine("The detected language is '{0}'. Confidence is: {1}.\nTranslation supported: {2}.\nTransliteration supported: {3}.\n",
            a.Language, a.Score, a.IsTranslationSupported, a.IsTransliterationSupported);
    }
}
```

如果使用的是认知服务多服务订阅，则还必须在请求参数中包括 `Ocp-Apim-Subscription-Region`。 [详细了解如何使用多服务订阅进行身份验证](/cognitive-services/translator/reference/v3-0-reference#authentication)。

## <a name="put-it-all-together"></a>将其放在一起

最后一步是在 `Main` 函数中调用 `DetectTextRequest()`。 找到 `static void Main(string[] args)` 并将其替换为以下代码：

```csharp
static async Task Main(string[] args)
{
    // This is our main function.
    // Output languages are defined in the route.
    // For a complete list of options, see API reference.
    string subscriptionKey = "YOUR_TRANSLATOR_TEXT_KEY_GOES_HERE";
    string host = "https://api.translator.azure.cn";
    string route = "/detect?api-version=3.0";
    string breakSentenceText = @"How are you doing today? The weather is pretty pleasant. Have you been to the movies lately?";
    await DetectTextRequest(subscriptionKey, host, route, breakSentenceText);
}
```
## <a name="run-the-sample-app"></a>运行示例应用

上述操作完成后，就可以运行示例应用了。 从命令行（或终端会话）导航到项目目录，然后运行以下命令：

```console
dotnet run
```

## <a name="sample-response"></a>示例响应

运行示例后，应当会看到输出到终端的以下内容：

> [!NOTE]
> 请在此[语言列表](/cognitive-services/translator/language-support)中查找国家/地区缩写。

```bash
The detected language is 'en'. Confidence is: 1.
Translation supported: True.
Transliteration supported: False.

Alternative 1
The detected language is 'fil'. Confidence is: 0.82.
Translation supported: True.
Transliteration supported: False.

Alternative 2
The detected language is 'ro'. Confidence is: 1.
Translation supported: True.
Transliteration supported: False.
```

此消息是从原始 JSON 构建的，如下所示：

```json
[  
    {  
        "language":"en",
        "score":1.0,
        "isTranslationSupported":true,
        "isTransliterationSupported":false,
        "alternatives":[  
            {  
                "language":"fil",
                "score":0.82,
                "isTranslationSupported":true,
                "isTransliterationSupported":false
            },
            {  
                "language":"ro",
                "score":1.0,
                "isTranslationSupported":true,
                "isTransliterationSupported":false
            }
        ]
    }
]
```

## <a name="clean-up-resources"></a>清理资源

请务必删除示例应用的源代码中的机密信息，例如订阅密钥。

## <a name="next-steps"></a>后续步骤

查看 API 参考，了解使用文本翻译 API 可以执行的所有操作。

> [!div class="nextstepaction"]
> [API 参考](/cognitive-services/translator/reference/v3-0-reference)

## <a name="see-also"></a>另请参阅

* [翻译文本](quickstart-csharp-translate.md)
* [直译文本](quickstart-csharp-transliterate.md)
* [获取备用翻译](quickstart-csharp-dictionary.md)
* [获取支持的语言的列表](quickstart-csharp-languages.md)
* [根据输入确定句子长度](quickstart-csharp-sentences.md)

<!-- Update_Description: wording update -->