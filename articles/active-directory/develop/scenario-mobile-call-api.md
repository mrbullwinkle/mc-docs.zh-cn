---
title: 调用 Web API 的移动应用 - 调用 Web API | Microsoft 标识平台
description: 了解如何构建调用 Web API 的移动应用（调用 Web API）
services: active-directory
documentationcenter: dev-center-name
author: danieldobalian
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
ms.reviwer: brandwe
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0ea43458a9af60a3b8ec8210b58063fb7abe7f1b
ms.sourcegitcommit: 9d5fd3184b6a47bf3b60ffdeeee22a08354ca6b1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/21/2019
ms.locfileid: "67305992"
---
# <a name="mobile-app-that-calls-web-apis---call-a-web-api"></a>调用 Web API 的移动应用 - 调用 Web API

在应用登录用户并接收令牌以后，MSAL 会公开有关用户、用户的环境以及所颁发令牌的多项信息。 应用可以使用这些值来调用 Web API 或向用户显示欢迎消息。

首先，我们将查看 MSAL 结果。 接下来，我们看看如何使用 `AuthenticationResult` 或 `result` 中的访问令牌来调用受保护的 Web API。

## <a name="msal-result"></a>MSAL 结果
MSAL 提供以下值： 

- `AccessToken`：用于在 HTTP 持有者请求中调用受保护的 Web API。
- `IdToken`：包含已登录用户的有用信息，例如用户的名称、主租户，以及存储的唯一标识符。
- `ExpiresOn`：令牌的过期时间。 MSAL 处理应用的自动刷新。
- `TenantId`：用户登录时使用的租户的标识符。 
- `Scopes`：通过令牌授予的作用域。 授予的作用域可能是请求的作用域的子集。

MSAL 也提供 `Account` 的抽象。 `Account` 表示当前用户的已登录帐户。

- `HomeAccountIdentifier`：用户的主租户的标识符。
- `UserName`：用户的首选用户名。 Azure Active Directory B2C 用户的此项可能为空。
- `AccountIdentifier`：已登录用户的标识符。 大多数情况下，此值会与 `HomeAccountIdentifier` 值相同，除非用户是另一租户中的来宾。

## <a name="call-an-api"></a>调用 API

有了访问令牌以后，即可轻松调用 Web API。 应用将使用此令牌构造一个 HTTP 请求，然后运行该请求。

### <a name="android"></a>Android

```Java
        RequestQueue queue = Volley.newRequestQueue(this);
        JSONObject parameters = new JSONObject();

        try {
            parameters.put("key", "value");
        } catch (Exception e) {
            // Error when constructing.
        }
        JsonObjectRequest request = new JsonObjectRequest(Request.Method.GET, MSGRAPH_URL,
                parameters,new Response.Listener<JSONObject>() {
            @Override
            public void onResponse(JSONObject response) {
                // Successfully called Graph. Process data and send to UI.
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                // Error.
            }
        }) {
            @Override
            public Map<String, String> getHeaders() throws AuthFailureError {
                Map<String, String> headers = new HashMap<>();
                
                // Put access token in HTTP request.
                headers.put("Authorization", "Bearer " + authResult.getAccessToken());
                return headers;
            }
        };

        request.setRetryPolicy(new DefaultRetryPolicy(
                3000,
                DefaultRetryPolicy.DEFAULT_MAX_RETRIES,
                DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        queue.add(request);
```

### <a name="ios"></a>iOS

```swift
        let url = URL(string: kGraphURI)
        var request = URLRequest(url: url!)

        // Put access token in HTTP request.
        request.setValue("Bearer \(self.accessToken)", forHTTPHeaderField: "Authorization")

        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                self.updateLogging(text: "Couldn't get graph result: \(error)")
                return
            }
            guard let result = try? JSONSerialization.jsonObject(with: data!, options: []) else {
                self.updateLogging(text: "Couldn't deserialize result JSON")
                return
            }

            // Successfully got data from Graph.
            self.updateLogging(text: "Result from Graph: \(result))")
        }.resume()
```

### <a name="xamarin"></a>Xamarin

```CSharp
httpClient = new HttpClient();

// Put access token in HTTP request.
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

// Call Graph.
HttpResponseMessage response = await _httpClient.GetAsync(apiUri);
...
}
```

## <a name="making-several-api-requests"></a>发出多个 API 请求

如需调用同一 API 多次，或者需要调用多个 API，请在生成应用时考虑以下事项：

- **增量许可**：Microsoft 标识平台允许应用在需要权限的时候获取用户许可，而不是在开始时统一这样做。 应用在每次准备调用 API 时，应该只请求需要使用的作用域。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [移到生产环境](scenario-mobile-production.md)

