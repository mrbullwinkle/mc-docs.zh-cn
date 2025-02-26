---
title: 快速入门：识别语音，Python - 语音服务
titleSuffix: Azure Cognitive Services
description: 按照本指南使用适用于 Python 的语音 SDK 创建语音转文本控制台应用程序。 完成后，可以使用计算机的麦克风实时将语音转录为文本。
services: cognitive-services
author: chlandsi
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
origin.date: 2/20/2019
ms.date: 04/01/2019
ms.author: v-biyu
ms.openlocfilehash: 1ce8953a57c07e400f56a3c9c4fd783f23a125ad
ms.sourcegitcommit: edce097f471b6e9427718f0641ee2b421e3c0ed2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/22/2019
ms.locfileid: "58348372"
---
# <a name="quickstart-recognize-speech-with-the-speech-sdk-for-python"></a>快速入门：使用适用于 Python 的语音 SDK 识别语音

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-quickstart-selector.md)]

本文介绍如何通过适用于 Python 的语音 SDK 来使用语音服务。 其中演示了如何识别麦克风输入中的语音。

## <a name="prerequisites"></a>先决条件

* 语音服务的 Azure 订阅密钥。 [获取一个试用版](get-started.md)。
* [Python 3.5 或更高版本](https://www.python.org/downloads/)。
* Python 语音 SDK 包适用于以下操作系统： 
    * Windows：x64 和 x86。
    * Mac：macOS X 10.12 或更高版本。
    * Linux：x64 架构上的 Ubuntu 16.04 或 18.04。
* 在 Ubuntu 上，请运行以下命令安装所需的包：

  ```sh
  sudo apt-get update
  sudo apt-get install build-essential libssl1.0.0 libasound2 wget
  ```

* 在 Windows 上，还需要安装适用于你的平台的 [Microsoft Visual C++ Redistributable for Visual Studio 2017](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads)。

## <a name="install-the-speech-sdk"></a>安装语音 SDK

[!INCLUDE [License Notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

此命令从语音 SDK 的 [PyPI](https://pypi.org/) 安装 Python 包：

```sh
pip install azure-cognitiveservices-speech
```

## <a name="support-and-updates"></a>支持和更新 

语音 SDK Python 包的更新将通过 PyPI 分发，[发行说明](./releasenotes.md)中会发布相关通告。
如果有新版本可用，可以使用 `pip install --upgrade azure-cognitiveservices-speech` 命令进行更新。
通过查看 `azure.cognitiveservices.speech.__version__` 变量来检查当前安装的版本。 

如果遇到问题或者缺少某项功能，请查看[支持和帮助选项](./support.md)。

## <a name="create-a-python-application-that-uses-the-speech-sdk"></a>使用语音 SDK 创建 Python 应用程序

### <a name="run-the-sample"></a>运行示例

可将本快速入门中的[示例代码](#sample-code)复制到源文件 `quickstart.py`，然后在 IDE 或控制台中运行该代码

```sh
python quickstart.py
```

或者，可以从[语音 SDK 示例存储库](https://github.com/Azure-Samples/cognitive-services-speech-sdk/)以 [Jupyter](https://jupyter.org) Notebook 的形式下载本快速入门教程，并将其作为 Notebook 运行。 

### <a name="sample-code"></a>代码示例

```Python
import azure.cognitiveservices.speech as speechsdk

# Creates an instance of a speech config with specified subscription key and service region.
# Replace with your own subscription key and service region.
speech_key, service_region = "YourSubscriptionKey", "YourServiceRegion"
speech_config = speechsdk.SpeechConfig(subscription=speech_key, region=service_region)

# Creates a recognizer with the given settings
speech_recognizer = speechsdk.SpeechRecognizer(speech_config=speech_config)

print("Say something...")


# Starts speech recognition, and returns after a single utterance is recognized. The end of a
# single utterance is determined by listening for silence at the end or until a maximum of 15
# seconds of audio is processed.  The task returns the recognition text as result. 
# Note: Since recognize_once() returns only a single utterance, it is suitable only for single
# shot recognition like command or query. 
# For long-running multi-utterance recognition, use start_continuous_recognition() instead.
result = speech_recognizer.recognize_once()

# Checks result.
if result.reason == speechsdk.ResultReason.RecognizedSpeech:
    print("Recognized: {}".format(result.text))
elif result.reason == speechsdk.ResultReason.NoMatch:
    print("No speech could be recognized: {}".format(result.no_match_details))
elif result.reason == speechsdk.ResultReason.Canceled:
    cancellation_details = result.cancellation_details
    print("Speech Recognition canceled: {}".format(cancellation_details.reason))
    if cancellation_details.reason == speechsdk.CancellationReason.Error:
        print("Error details: {}".format(cancellation_details.error_details))
``` 

### <a name="install-and-use-the-speech-sdk-with-visual-studio-code"></a>通过 Visual Studio Code 安装并使用语音 SDK

1. 在计算机上下载并安装 64 位版本的 [Python](https://www.python.org/downloads/)（3.5 或更高版本）。
1. 下载并安装 [Visual Studio Code](https://code.visualstudio.com/Download)。
1. 打开 Visual Studio Code 并安装 Python 扩展。 在菜单中选择“文件”   >   “首选项” >   “扩展”。 搜索 **Python**。

   ![安装 Python 扩展](media/sdk/qs-python-vscode-python-extension.png)

1. 创建一个文件夹用于存储项目。 例如，使用 Windows 资源管理器。
1. 在 Visual Studio Code 中选择“文件”图标。  然后打开创建的文件夹。

   ![打开文件夹](media/sdk/qs-python-vscode-python-open-folder.png)
   
1. 选择“新建文件”图标创建新的 Python 源文件 `speechsdk.py`。

   ![创建文件](media/sdk/qs-python-vscode-python-newfile.png)

1. 复制 [Python 代码](#sample-code)并将其粘贴到新建的文件，然后保存文件。
1. 插入语音服务订阅信息。
1. 如果已选择 Python 解释器，窗口底部的状态栏左侧会显示它。
   否则，会显示可用 Python 解释器的列表。 打开命令面板 (Ctrl+Shift+P) 并输入 **Python:Select Interpreter**。 选择适当的解释器。
1. 如果尚未为所选的 Python 解释器安装， 可以从 Visual Studio Code 内部安装语音 SDK Python 包。
   若要安装语音 SDK 包，请打开终端。 再次启动命令面板 (Ctrl+Shift+P) 并输入 **Terminal:Create New Integrated Terminal** 来打开终端。
   在打开的终端中，输入命令 `python -m pip install azure-cognitiveservices-speech`，或者输入适用于系统的命令。
1. 若要运行示例代码，请在编辑器中的某个位置单击右键。 选择“在终端中运行 Python 文件”。 
   根据提示讲出几句话。 片刻之后，听录的文本将会显示。

   ![运行示例](media/sdk/qs-python-vscode-python-run.png)

如果在遵照这些说明操作时遇到问题，请参阅内容更全面的 [Visual Studio Code Python 教程](https://code.visualstudio.com/docs/python/python-tutorial)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [浏览 GitHub 上的 Python 示例](https://aka.ms/csspeech/samples)
