---
title: 快速入门：检测文本语言，Java - 文本翻译 API
titleSuffix: Azure Cognitive Services
description: 本快速入门介绍如何使用 Java 和文本翻译 REST API 来检测所提供文本的语言。
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: quickstart
origin.date: 06/04/2019
ms.date: 07/23/2019
ms.author: v-junlch
ms.openlocfilehash: 0e0f1e9ab57aa9de691b1693677693d56fae0435
ms.sourcegitcommit: 9a330fa5ee7445b98e4e157997e592a0d0f63f4c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/24/2019
ms.locfileid: "68439970"
---
# <a name="quickstart-use-the-translator-text-api-to-detect-text-language-using-java"></a>快速入门：使用文本翻译 API 通过 Java 来检测文本语言

本快速入门介绍如何使用 Java 和文本翻译 REST API 来检测所提供文本的语言。

此快速入门需要包含文本翻译资源的 [Azure 认知服务帐户](/cognitive-services/cognitive-services-apis-create-account)。 如果没有帐户，可以使用[试用帐户](https://www.azure.cn/pricing/1rmb-trial/)获取订阅密钥。

>[!TIP]
> 如果你想一次看到所有代码，这个示例的源代码可以在 [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Java) 上找到。

## <a name="prerequisites"></a>先决条件

* [JDK 7 或更高版本](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* [Gradle](https://gradle.org/install/)
* 适用于文本翻译的 Azure 订阅密钥

## <a name="initialize-a-project-with-gradle"></a>使用 Gradle 初始化项目

首先，创建此项目的工作目录。 从命令行（或终端）中，运行以下命令：

```console
mkdir detect-sample
cd detect-sample
```

接下来，初始化一个 Gradle 项目。 此命令将创建 Gradle 的基本生成文件，最重要的是 `build.gradle.kts`，它在运行时用来创建并配置应用程序。 从工作目录运行以下命令：

```console
gradle init --type basic
```

当提示你选择一个 **DSL** 时，选择 **Kotlin**。

## <a name="configure-the-build-file"></a>配置生成文件

找到 `build.gradle.kts` 并使用你喜欢使用的 IDE 或文本编辑器将其打开。 然后将以下生成配置复制到其中：

```
plugins {
    java
    application
}
application {
    mainClassName = "Detect"
}
repositories {
    mavenCentral()
}
dependencies {
    compile("com.squareup.okhttp:okhttp:2.5.0")
    compile("com.google.code.gson:gson:2.8.5")
}
```

请注意，此示例依赖于 HTTP 请求的 OkHttp 以及 Gson 来处理和分析 JSON。 如果要详细了解生成配置，请参阅[创建新的 Gradle 生成](https://guides.gradle.org/creating-new-gradle-builds/)。

## <a name="create-a-java-file"></a>创建 Java 文件

为示例应用创建一个文件夹。 从工作目录中，运行：

```console
mkdir -p src/main/java
```

接下来，在此文件夹中，创建一个名为 `Detect.java` 的文件。

## <a name="import-required-libraries"></a>导入所需的库

打开 `Detect.java` 并添加以下 import 语句：

```java
import java.io.*;
import java.net.*;
import java.util.*;
import com.google.gson.*;
import com.squareup.okhttp.*;
```


## <a name="define-variables"></a>定义变量

首先，为你的项目创建一个公共类：

```java
public class Detect {
  // All project code goes here...
}
```

将以下行添加到 `Detect` 类：

```java
String subscriptionKey = "YOUR_SUBSCRIPTION_KEY";
String url = "https://api.translator.azure.cn/detect?api-version=3.0";
```

如果使用的是认知服务多服务订阅，则还必须在请求参数中包括 `Ocp-Apim-Subscription-Region`。 [详细了解如何使用多服务订阅进行身份验证](/cognitive-services/translator/reference/v3-0-reference#authentication)。

## <a name="create-a-client-and-build-a-request"></a>创建客户端并生成请求

将以下行添加到 `Detect` 类来实例化 `OkHttpClient`：

```java
// Instantiates the OkHttpClient.
OkHttpClient client = new OkHttpClient();
```

接下来，我们将生成 POST 请求。 可随意更改进行语言检测的文本。

```java
// This function performs a POST request.
public String Post() throws IOException {
    MediaType mediaType = MediaType.parse("application/json");
    RequestBody body = RequestBody.create(mediaType,
            "[{\n\t\"Text\": \"Salve mondo!\"\n}]");
    Request request = new Request.Builder()
            .url(url).post(body)
            .addHeader("Ocp-Apim-Subscription-Key", subscriptionKey)
            .addHeader("Ocp-Apim-Subscription-Region", "your region")
            .addHeader("Content-type", "application/json").build();
    Response response = client.newCall(request).execute();
    return response.body().string();
}
```

## <a name="create-a-function-to-parse-the-response"></a>创建一个函数来分析响应

这个简单的函数分析来自文本翻译服务的 JSON 响应并对其进行美化。

```java
// This function prettifies the json response.
public static String prettify(String json_text) {
    JsonParser parser = new JsonParser();
    JsonElement json = parser.parse(json_text);
    Gson gson = new GsonBuilder().setPrettyPrinting().create();
    return gson.toJson(json);
}
```

## <a name="put-it-all-together"></a>将其放在一起

最后一步是发出请求并获得响应。 将以下行添加你的项目：

```java
public static void main(String[] args) {
    try {
        Detect detectRequest = new Detect();
        String response = detectRequest.Post();
        System.out.println(prettify(response));
    } catch (Exception e) {
        System.out.println(e);
    }
}
```

## <a name="run-the-sample-app"></a>运行示例应用

上述操作完成后，就可以运行示例应用了。 从命令行（或终端会话）导航到工作目录的根，然后运行以下命令：

```console
gradle build
```

当生成完成后，请运行：

```console
gradle run
```

## <a name="sample-response"></a>示例响应

请在此[语言列表](/cognitive-services/translator/language-support)中查找国家/地区缩写。

```json
[
  {
    "language": "it",
    "score": 1.0,
    "isTranslationSupported": true,
    "isTransliterationSupported": false,
    "alternatives": [
      {
        "language": "pt",
        "score": 1.0,
        "isTranslationSupported": true,
        "isTransliterationSupported": false
      },
      {
        "language": "en",
        "score": 1.0,
        "isTranslationSupported": true,
        "isTransliterationSupported": false
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

* [翻译文本](quickstart-java-translate.md)
* [直译文本](quickstart-java-transliterate.md)
* [按输入确定语言](quickstart-java-detect.md)
* [获取备用翻译](quickstart-java-dictionary.md)
* [获取支持的语言的列表](quickstart-java-languages.md)
* [根据输入确定句子长度](quickstart-java-sentences.md)

<!-- Update_Description: wording update -->