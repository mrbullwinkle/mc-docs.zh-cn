---
title: 单页应用程序（应用的代码配置）- Microsoft 标识平台
description: 了解如何生成单页应用程序（应用的代码配置）
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
origin.date: 05/07/2019
ms.date: 06/20/2019
ms.author: v-junlch
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 81196e2d72eb3d946278fb4c73f77a63f54aa78a
ms.sourcegitcommit: 9d5fd3184b6a47bf3b60ffdeeee22a08354ca6b1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/21/2019
ms.locfileid: "67305991"
---
# <a name="single-page-application---code-configuration"></a>单页应用程序 - 代码配置

了解如何为单页应用程序 (SPA) 配置代码。

## <a name="msal-libraries-supporting-implicit-flow"></a>支持隐式流的 MSAL 库

Microsoft 标识平台提供了 MSAL.js 库，以业界推荐的安全做法来支持隐式流。  

支持隐式流的库包括：

| MSAL 库 | 说明 |
|--------------|--------------|
| ![MSAL.js](./media/sample-v2-code/logo_js.png) <br/> [MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js)  | 用于任何使用 JavaScript 或 SPA 框架（如 Angular、Vue.js、React.js 等）构建的客户端 Web 应用的纯 JavaScript 库。 |
| ![MSAL Angular](./media/sample-v2-code/logo_angular.png) <br/> [MSAL Angular](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/README.md) | 核心 MSAL.js 库的包装器，用于简化在使用 Angular 框架构建的单页应用中的使用。 此库处于预览状态，并且在某些 Angular 版本和浏览器中存在[已知问题](https://github.com/AzureAD/microsoft-authentication-library-for-js/issues?q=is%3Aopen+is%3Aissue+label%3Aangular)。 |

## <a name="application-code-configuration"></a>应用程序代码配置

在 MSAL 库中，应用程序注册信息在库初始化期间作为配置传递。

### <a name="javascript"></a>Javascript

```javascript
// Configuration object constructed.
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri" //defaults to application start page
    }
}

// create UserAgentApplication instance
const userAgentApplication = new UserAgentApplication(config);
```
有关可用配置选项的更多详细信息，请参阅[使用 MSAL.js 初始化应用程序](msal-js-initializing-client-applications.md)。

### <a name="angular"></a>Angular

```javascript
//In app.module.ts
@NgModule({
  imports: [ MsalModule.forRoot({
                clientId: 'your_app_id'
            })]
         })

  export class AppModule { }
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [登录和注销](scenario-spa-sign-in.md)

