---
title: 应用程序设置
titleSuffix: Azure Cognitive Services
description: 了解语言理解应用的应用程序设置。
services: cognitive-services
author: lingliw
manager: digimobile
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: article
ms.date: 05/29/19
ms.author: v-lingwu
ms.openlocfilehash: 267d13d507e32c53c6e77b3927ff9b49a3e2d10a
ms.sourcegitcommit: 68f7c41974143a8f7bd9b7a54acf41c09893e587
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68332678"
---
# <a name="application-settings"></a>应用程序设置

这些应用程序设置存储在[导出的](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c40)应用中，并使用 REST API 进行[更新](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings)。 更改应用版本设置会将应用训练状态重置为“未训练”。

|设置|默认值|注释|
|--|--|--|
|NormalizePunctuation|True|删除标点。|
|NormalizeDiacritics|True|删除音调符号。|

## <a name="diacritics-normalization"></a>音调符号规范化 

在 `settings` 参数中针对 LUIS JSON 应用文件的音调符号打开话语规范化。

```JSON
"settings": [
    {"name": "NormalizeDiacritics", "value": "true"}
] 
```

以下话语显示了音调符号规范化如何影响话语：

|音调符号设置为 false 时|音调符号设置为 true 时|
|--|--|
|`quiero tomar una piña colada`|`quiero tomar una pina colada`|
|||

### <a name="language-support-for-diacritics"></a>对音调符号的语言支持

#### <a name="brazilian-portuguese-pt-br-diacritics"></a>巴西葡萄牙语 `pt-br` 音调符号

|音调符号设置为 false|音调符号设置为 false|
|-|-|
|`á`|`a`|
|`â`|`a`|
|`ã`|`a`|
|`à`|`a`|
|`ç`|`c`|
|`é`|`e`|
|`ê`|`e`|
|`í`|`i`|
|`ó`|`o`|
|`ô`|`o`|
|`õ`|`o`|
|`ú`|`u`| 
|||

#### <a name="dutch-nl-nl-diacritics"></a>荷兰语 `nl-nl` 音调符号

|音调符号设置为 false|音调符号设置为 false|
|-|-|
|`á`|`a`|
|`à`|`a`|
|`é`|`e`|
|`ë`|`e`|
|`è`|`e`|
|`ï`|`i`|
|`í`|`i`|
|`ó`|`o`|
|`ö`|`o`|
|`ú`|`u`| 
|`ü`|`u`|
|||

#### <a name="french-fr--diacritics"></a>法语 `fr-` 音调符号

这包括法国和加拿大的子区域性。

|音调符号设置为 false|音调符号设置为 false|
|--|--|
|`é`|`e`|
|`à`|`a`|
|`è`|`e`|
|`ù`|`u`|
|`â`|`a`| 
|`ê`|`e`| 
|`î`|`i`| 
|`ô`|`o`| 
|`û`|`u`| 
|`ç`|`c`| 
|`ë`|`e`| 
|`ï`|`i`| 
|`ü`|`u`| 
|`ÿ`|`y`| 

#### <a name="german-de-de-diacritics"></a>德语 `de-de` 音调符号

|音调符号设置为 false|音调符号设置为 false|
|--|--|
|`ä`|`a`|
|`ö`|`o`| 
|`ü`|`u`| 

#### <a name="italian-it-it-diacritics"></a>意大利语 `it-it` 音调符号

|音调符号设置为 false|音调符号设置为 false|
|--|--|
|`à`|`a`|
|`è`|`e`|
|`é`|`e`|
|`ì`|`i`| 
|`í`|`i`| 
|`î`|`i`| 
|`ò`|`o`| 
|`ó`|`o`| 
|`ù`|`u`|
|`ú`|`u`|

#### <a name="spanish-es--diacritics"></a>西班牙语 `es-` 音调符号

这包括西班牙和加拿大墨西哥。

|音调符号设置为 false|音调符号设置为 false|
|-|-|
|`á`|`a`|
|`é`|`e`|
|`í`|`i`| 
|`ó`|`o`| 
|`ú`|`u`|
|`ü`|`u`|
|`ñ`|`u`|


## <a name="punctuation-normalization"></a>标点规范化

在 `settings` 参数中针对 LUIS JSON 应用文件的标点打开话语规范化。

```JSON
"settings": [
    {"name": "NormalizePunctuation", "value": "true"}
] 
```

以下话语显示了音调符号如何影响话语：

|音调符号设置为 False 时|音调符号设置为 True 时|
|--|--|
|`Hmm..... I will take the cappuccino`|`Hmm I will take the cappuccino`|
|||

### <a name="punctuation-removed"></a>标点已删除

将 `NormalizePunctuation` 设置为 true 时，将删除以下标点符号。

|标点|
|--|
|`-`| 
|`.`| 
|`'`|
|`"`|
|`\`|
|`/`|
|`?`|
|`!`|
|`_`|
|`,`|
|`;`|
|`:`|
|`(`|
|`)`|
|`[`|
|`]`|
|`{`|
|`}`|
|`+`|
|`¡`|
