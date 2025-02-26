---
title: 了解 Azure AD 中的 OAuth 2.0 授权代码流
description: 本文介绍如何使用 Azure Active Directory 和 OAuth 2.0，通过 HTTP 消息来授权访问租户中的 Web 应用程序和 Web API。
services: active-directory
documentationcenter: .net
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
origin.date: 03/05/2019
ms.date: 07/01/2019
ms.author: v-junlch
ms.reviewer: hirsin
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 82dddcbfbbfad97432adabab19bdb2ddf7892713
ms.sourcegitcommit: 5f85d6fe825db38579684ee1b621d19b22eeff57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67568688"
---
# <a name="authorize-access-to-azure-active-directory-web-applications-using-the-oauth-20-code-grant-flow"></a>使用 OAuth 2.0 代码授权流来授权访问 Azure Active Directory Web 应用程序

Azure Active Directory (Azure AD) 使用 OAuth 2.0，使你能够授权访问 Azure AD 租户中的 Web 应用程序和 Web API。 本指南与语言无关，介绍在不使用我们的[开放源代码库](active-directory-authentication-libraries.md)的情况下，如何发送和接收 HTTP 消息。

[OAuth 2.0 规范第 4.1 部分](https://tools.ietf.org/html/rfc6749#section-4.1)描述了 OAuth 2.0 授权代码流。 它用于在大多数应用程序类型（包括 Web 应用和本机安装的应用）中执行身份验证与授权。

[!INCLUDE [active-directory-protocols-getting-started](../../../includes/active-directory-protocols-getting-started.md)]

## <a name="oauth-20-authorization-flow"></a>OAuth 2.0 授权流

概括而言，应用程序的整个授权流看起来有点类似于：

![OAuth 授权代码流](./media/v1-protocols-oauth-code/active-directory-oauth-code-flow-native-app.png)

## <a name="request-an-authorization-code"></a>请求授权代码

授权代码流始于客户端将用户定向到的 `/authorize` 终结点。 在此请求中，客户端指示了用户需要提供的权限。 可以通过在 Azure 门户中选择“应用注册”>“终结点”，获取租户的 OAuth 2.0 授权终结点  。

```
// Line breaks for legibility only

https://login.partner.microsoftonline.cn/{tenant}/oauth2/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%3A12345
&response_mode=query
&resource=https%3A%2F%2Fservice.contoso.com%2F
&state=12345
```

| 参数 |  | 说明 |
| --- | --- | --- |
| tenant |必填 |请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。 允许值为租户标识符，例如独立于租户令牌的 `8eaef023-2b34-4da1-9baa-8bc8c9d6a490`、`contoso.partner.onmschina.cn` 或 `common` |
| client_id |必填 |将应用注册到 Azure AD 时，分配给应用的应用程序 ID。 可在 Azure 门户中找到该值。 单击服务边栏中的“Azure Active Directory”  ，单击“应用注册”  ，然后选择应用程序。 |
| response_type |必填 |必须包括授权代码流的 `code` 。 |
| redirect_uri |建议 |应用的 redirect_uri，应用可向其发送及从其接收身份验证响应。 其必须完全符合在门户中注册的其中一个 redirect_uris，否则必须是编码的 url。 对于本机和移动应用，应使用默认值 `urn:ietf:wg:oauth:2.0:oob`。 |
| response_mode |可选 |指定将生成的令牌送回到应用程序时应该使用的方法。 可以是 `query`、`fragment` 或 `form_post`。 `query` 在重定向 URI 上提供代码作为查询字符串参数。 如果要使用隐式流请求 ID 令牌，则不能使用 [OpenID 规范](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html#Combinations)中指定的 `query`。如果只是请求代码，则可以使用 `query`、`fragment` 或 `form_post`。 `form_post` 对重定向 URI 执行包含代码的 POST。 代码流的默认值为 `query`。  |
| state |建议 |同时随令牌响应返回的请求中所包含的值。 随机生成的唯一值通常用于 [防止跨站点请求伪造攻击](https://tools.ietf.org/html/rfc6749#section-10.12)。 该状态也用于在身份验证请求出现之前，于应用中编码用户的状态信息，例如之前所在的网页或视图。 |
| resource | 建议 |目标 Web API 的应用 ID URI（受保护的资源）。 要查找应用 ID URI，请在 Azure 门户中，依次单击“Azure Active Directory”和“应用程序注册”，打开应用程序的“设置”页面，然后单击“属性”     。 也可能是外部资源，如 `https://microsoftgraph.chinacloudapi.cn`。 这在授权或令牌请求中是必需的。 要确保减少身份验证提示，请将其置于授权请求中以确保获得用户许可。 |
| scope | **ignored** | 对于 v1 Azure AD 应用，必须在 Azure 门户中的应用程序“设置”  、“所需权限”  下静态配置作用域。 |
| prompt |可选 |表示需要的用户交互类型。<p> 有效值是： <p> login  ：应该提示用户重新进行身份验证。 <p> select_account  ：系统会提示用户选择一个帐户，从而中断单一登录。 用户可以选择现有已登录帐户，输入已记忆帐户的凭据，或选择使用其他帐户。 <p> *consent*：用户同意已授予，但需要进行更新。 应该提示用户授予同意。 <p> *admin_consent*：应该提示管理员代表组织中的所有用户授予同意 |
| login_hint |可选 |如果事先知道用户名，可用于预先填充用户登录页的用户名/电子邮件地址字段。 通常，应用在重新身份验证期间使用此参数，并且已经使用 `preferred_username` 声明从前次登录提取用户名。 |
| domain_hint |可选 |提供有关用户应该用于登录的租户或域的提示。 domain_hint 的值是租户的已注册域。 如果该租户与本地目录联合，则 AAD 将重定向到指定的租户联合服务器。 |
| code_challenge_method | 建议    | 用于为 `code_challenge` 参数编码 `code_verifier` 的方法。 可以是 `plain` 或 `S256` 之一。 如果已排除在外，且包含了 `code_challenge`，则假定 `code_challenge` 为纯文本。 Azure AAD v1.0 同时支持 `plain` 和 `S256`。 有关详细信息，请参阅 [PKCE RFC](https://tools.ietf.org/html/rfc7636)。 |
| code_challenge        | 建议    | 用于通过本机客户端或公共客户端的 Proof Key for Code Exchange (PKCE) 保护授权代码授权。 如果包含 `code_challenge_method`，则需要。 有关详细信息，请参阅 [PKCE RFC](https://tools.ietf.org/html/rfc7636)。 |

> [!NOTE]
> 如果用户属于某个组织，则该组织的管理员可以代表该用户许可或拒绝，也可以允许该用户进行许可。 仅当管理员允许时，用户才有权许可。
>
>

此时，会要求用户输入其凭据，并许可 Azure 门户中的应用程序请求的权限。 用户经过身份验证并同意后，Azure AD 会在包含代码的请求的 `redirect_uri` 地址中向应用发送响应。

### <a name="successful-response"></a>成功的响应
成功的响应如下所示：

```
GET  HTTP/1.1 302 Found
Location: http://localhost:12345/?code= AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrqqf_ZT_p5uEAEJJ_nZ3UmphWygRNy2C3jJ239gV_DBnZ2syeg95Ki-374WHUP-i3yIhv5i-7KU2CEoPXwURQp6IVYMw-DjAOzn7C3JCu5wpngXmbZKtJdWmiBzHpcO2aICJPu1KvJrDLDP20chJBXzVYJtkfjviLNNW7l7Y3ydcHDsBRKZc3GuMQanmcghXPyoDg41g8XbwPudVh7uCmUponBQpIhbuffFP_tbV8SNzsPoFz9CLpBCZagJVXeqWoYMPe2dSsPiLO9Alf_YIe5zpi-zY4C3aLw5g9at35eZTfNd0gBRpR5ojkMIcZZ6IgAA&session_state=7B29111D-C220-4263-99AB-6F6E135D75EF&state=D79E5777-702E-4260-9A62-37F75FF22CCE
```

| 参数 | 说明 |
| --- | --- |
| admin_consent |如果管理员同意许可请求提示的内容，则该值为 True。 |
| code |应用程序请求的授权代码。 应用程序可以使用该授权代码请求目标资源的访问令牌。 |
| session_state |一个标识当前用户会话的唯一值。 此值为 GUID，但应将其视为无需检查即可传递的不透明值。 |
| state |如果请求中包含 state 参数，响应中就应该出现相同的值。 应用程序在使用响应之前最好验证请求和响应中的状态值是否完全相同。 这可以帮助检测客户端的[跨网站请求伪造 (CSRF) 攻击](https://tools.ietf.org/html/rfc6749#section-10.12)。 |

### <a name="error-response"></a>错误响应
错误响应也可能发送到 `redirect_uri` ，以便应用程序可以适当地处理。

```
GET http://localhost:12345/?
error=access_denied
&error_description=the+user+canceled+the+authentication
```

| 参数 | 说明 |
| --- | --- |
| error |[OAuth 2.0 授权框架](https://tools.ietf.org/html/rfc6749)第 5.2 部分中定义的错误代码值。 下表描述了 Azure AD 返回的错误代码。 |
| error_description |错误的更详细说明。 此消息不是最终用户友好的。 |
| state |状态值是一个随机生成的不可重用值，它在请求中发送，并在响应中返回，以防止跨网站请求伪造 (CSRF) 攻击。 |

#### <a name="error-codes-for-authorization-endpoint-errors"></a>授权终结点错误的错误代码
下表描述了可在错误响应的 `error` 参数中返回的各个错误代码。

| 错误代码 | 说明 | 客户端操作 |
| --- | --- | --- |
| invalid_request |协议错误，例如，缺少必需的参数。 |修复并重新提交请求。 这通常是在初始测试期间捕获的开发错误。 |
| unauthorized_client |不允许客户端应用程序请求授权代码。 |客户端应用程序未注册到 Azure AD 中或者未添加到用户的 Azure AD 租户时，通常会出现这种情况。 应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |
| access_denied |资源所有者拒绝了许可 |客户端应用程序可以通知用户除非用户许可，否则无法继续。 |
| unsupported_response_type |授权服务器不支持请求中的响应类型。 |修复并重新提交请求。 这通常是在初始测试期间捕获的开发错误。 |
| server_error |服务器遇到意外的错误。 |重试请求。 这些错误可能是临时状况导致的。 客户端应用程序可向用户说明，其响应由于临时错误而延迟。 |
| temporarily_unavailable |服务器暂时繁忙，无法处理请求。 |重试请求。 客户端应用程序可向用户说明，其响应由于临时状况而延迟。 |
| invalid_resource |目标资源无效，原因是它不存在，Azure AD 找不到它，或者未正确配置。 |这表示未在租户中配置该资源（如果存在）。 应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |

## <a name="use-the-authorization-code-to-request-an-access-token"></a>使用授权代码请求访问令牌
已获取授权代码并获得用户授权，现在可以通过将 POST 请求发送到 `/token` 终结点，使用该代码兑换所需资源的访问令牌：

```
// Line breaks for legibility only

POST /{tenant}/oauth2/token HTTP/1.1
Host: https://login.partner.microsoftonline.cn
Content-Type: application/x-www-form-urlencoded
grant_type=authorization_code
&client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrqqf_ZT_p5uEAEJJ_nZ3UmphWygRNy2C3jJ239gV_DBnZ2syeg95Ki-374WHUP-i3yIhv5i-7KU2CEoPXwURQp6IVYMw-DjAOzn7C3JCu5wpngXmbZKtJdWmiBzHpcO2aICJPu1KvJrDLDP20chJBXzVYJtkfjviLNNW7l7Y3ydcHDsBRKZc3GuMQanmcghXPyoDg41g8XbwPudVh7uCmUponBQpIhbuffFP_tbV8SNzsPoFz9CLpBCZagJVXeqWoYMPe2dSsPiLO9Alf_YIe5zpi-zY4C3aLw5g9at35eZTfNd0gBRpR5ojkMIcZZ6IgAA
&redirect_uri=https%3A%2F%2Flocalhost%3A12345
&resource=https%3A%2F%2Fservice.contoso.com%2F
&client_secret=p@ssw0rd

//NOTE: client_secret only required for web apps
```

| 参数 |  | 说明 |
| --- | --- | --- |
| tenant |必填 |请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。 允许值为租户标识符，例如独立于租户令牌的 `8eaef023-2b34-4da1-9baa-8bc8c9d6a490`、`contoso.partner.onmschina.cn` 或 `common` |
| client_id |必填 |将应用注册到 Azure AD 时，分配给应用的应用程序 ID。 可以在 Azure 门户中找到该值。 应用程序 ID 显示在应用注册的设置中。 |
| grant_type |必填 |必须是授权代码流的 `authorization_code` 。 |
| code |必填 |在上一部分中获取的 `authorization_code` |
| redirect_uri |必填 | 一个在客户端应用程序上注册的 `redirect_uri`。 |
| client_secret |对于 Web 应用是必需的，不允许用于公共客户端 |在 Azure 门户的“密钥”  下为应用程序创建的应用程序密码。 它不能在本机应用（公共客户端）中使用，因为设备无法可靠地存储 client_secrets。 Web 应用和 Web API（所有机密客户端）都需要它，能够将 `client_secret` 安全地存储在服务器端。 在发送 client_secret 之前必须先对其进行 URL 编码。 |
| resource | 建议 |目标 Web API 的应用 ID URI（受保护的资源）。 要查找应用 ID URI，请在 Azure 门户中，依次单击“Azure Active Directory”和“应用程序注册”，打开应用程序的“设置”页面，然后单击“属性”     。 也可能是外部资源，如 `https://microsoftgraph.chinacloudapi.cn`。 这在授权或令牌请求中是必需的。 要确保减少身份验证提示，请将其置于授权请求中以确保获得用户许可。 如果同时在授权请求和令牌请求中，资源参数必须匹配。 | 
| code_verifier | 可选 | 即用于获取 authorization_code 的 code_verifier。 如果在授权码授权请求中使用 PKCE，则需要。 有关详细信息，请参阅 [PKCE RFC](https://tools.ietf.org/html/rfc7636)   |

要查找应用 ID URI，请在 Azure 门户中，依次单击“Azure Active Directory”和“应用程序注册”，打开应用程序的“设置”页面，然后单击“属性”     。

### <a name="successful-response"></a>成功的响应
成功响应后，Azure AD 将返回[访问令牌](access-tokens.md)。 为了尽量减少来自客户端应用程序的网络调用及其相关延迟，客户端应用程序应该根据 OAuth 2.0 响应中指定的令牌生存期缓存访问令牌。 若要确定令牌生存期，请使用 `expires_in` 或 `expires_on` 参数值。

如果 Web API 资源返回 `invalid_token` 错误代码，则可能表示该资源确定令牌已过期。 如果客户端和资源时钟时间不同（称为“时间偏差”），该资源可能会将令牌视为已过期，随后从客户端缓存中清除。 如果发生这种情况，请从缓存中清除令牌，即使它仍处于其计算生存期内。

成功的响应如下所示：

```
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ",
  "token_type": "Bearer",
  "expires_in": "3600",
  "expires_on": "1388444763",
  "resource": "https://service.contoso.com/",
  "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4rTfgV29ghDOHRc2B-C_hHeJaJICqjZ3mY2b_YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfcUl4VBbiSHZyd1NVZG5QTIOcbObu3qnLutbpadZGAxqjIbMkQ2bQS09fTrjMBtDE3D6kSMIodpCecoANon9b0LATkpitimVCrl-NyfN3oyG4ZCWu18M9-vEou4Sq-1oMDzExgAf61noxzkNiaTecM-Ve5cq6wHqYQjfV9DOz4lbceuYCAA",
  "scope": "https%3A%2F%2Fmicrosoftgraph.chinacloudapi.cn%2Fmail.read",
  "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83ZmU4MTQ0Ny1kYTU3LTQzODUtYmVjYi02ZGU1N2YyMTQ3N2UvIiwiaWF0IjoxMzg4NDQwODYzLCJuYmYiOjEzODg0NDA4NjMsImV4cCI6MTM4ODQ0NDc2MywidmVyIjoiMS4wIiwidGlkIjoiN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlIiwib2lkIjoiNjgzODlhZTItNjJmYS00YjE4LTkxZmUtNTNkZDEwOWQ3NGY1IiwidXBuIjoiZnJhbmttQGNvbnRvc28uY29tIiwidW5pcXVlX25hbWUiOiJmcmFua21AY29udG9zby5jb20iLCJzdWIiOiJKV3ZZZENXUGhobHBTMVpzZjd5WVV4U2hVd3RVbTV5elBtd18talgzZkhZIiwiZmFtaWx5X25hbWUiOiJNaWxsZXIiLCJnaXZlbl9uYW1lIjoiRnJhbmsifQ."
}

```

| 参数 | 说明 |
| --- | --- |
| access_token |作为签名 JSON Web 令牌 (JWT) 的请求[访问令牌](access-tokens.md)。 应用可以使用此令牌来验证受保护的资源，例如 Web API。 |
| token_type |指示令牌类型值。 Azure AD 唯一支持的类型是 Bearer。 有关持有者令牌的详细信息，请参阅 [OAuth2.0 授权框架：持有者令牌用法 (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) |
| expires_in |访问令牌的有效期（以秒为单位）。 |
| expires_on |访问令牌的过期时间。 该日期表示为自 1970-01-01T0:0:0Z UTC 至过期时间的秒数。 此值用于确定缓存令牌的生存期。 |
| resource |Web API 的应用 ID URI（受保护的资源）。 |
| scope |向客户端应用程序授予的模拟权限。 默认权限为 `user_impersonation`。 受保护资源的所有者可在 Azure AD 中注册其他值。 |
| refresh_token |OAuth 2.0 刷新令牌。 应用可以使用此令牌，在当前访问令牌过期之后获取其他访问令牌。 刷新令牌的生存期很长，而且可以用于延长保留资源访问权限的时间。 |
| id_token |表示 [ID 令牌](id-tokens.md)的未签名 JSON Web令牌 (JWT)。 应用可以 base64Url 解码此令牌的段，以请求已登录用户的相关信息。 应用可以缓存并显示值，但不应依赖于这些值来获取任何授权或安全边界。 |

有关 JSON Web 令牌的详细信息，请参阅 [JWT IETF 草案规范](https://go.microsoft.com/fwlink/?LinkId=392344)。   若要详细了解 `id_tokens`，请参阅 [v1.0 OpenID Connect 流](v1-protocols-openid-connect-code.md)。

### <a name="error-response"></a>错误响应
令牌颁发终结点错误是一些 HTTP 错误代码，因为客户端直接调用令牌颁发终结点。 除了 HTTP 状态代码，Azure AD 令牌颁发终结点还会返回 JSON 文档，其中的对象会对错误进行描述。

下面是一个示例错误响应：

```
{
  "error": "invalid_grant",
  "error_description": "AADSTS70002: Error validating credentials. AADSTS70008: The provided authorization code or refresh token is expired. Send a new interactive authorization request for this user and resource.\r\nTrace ID: 3939d04c-d7ba-42bf-9cb7-1e5854cdce9e\r\nCorrelation ID: a8125194-2dc8-4078-90ba-7b6592a7f231\r\nTimestamp: 2016-04-11 18:00:12Z",
  "error_codes": [
    70002,
    70008
  ],
  "timestamp": "2016-04-11 18:00:12Z",
  "trace_id": "3939d04c-d7ba-42bf-9cb7-1e5854cdce9e",
  "correlation_id": "a8125194-2dc8-4078-90ba-7b6592a7f231"
}
```
| 参数 | 说明 |
| --- | --- |
| error |用于分类发生的错误类型与响应错误的错误码字符串。 |
| error_description |帮助开发人员识别身份验证错误根本原因的特定错误消息。 |
| error_codes |可帮助诊断的 STS 特定错误代码列表。 |
| timestamp |发生错误的时间。 |
| trace_id |可帮助诊断的请求唯一标识符。 |
| correlation_id |可帮助跨组件诊断的请求唯一标识符。 |

#### <a name="http-status-codes"></a>HTTP 状态代码
下表列出了令牌颁发终结点返回的 HTTP 状态代码。 在某些情况下，错误代码足以描述响应，但在发生错误的情况下，则需要分析随附的 JSON 文档并检查其错误代码。

| HTTP 代码 | 说明 |
| --- | --- |
| 400 |默认的 HTTP 代码。 用于大多数情况，原因通常是请求格式不当。 修复并重新提交请求。 |
| 401 |身份验证失败。 例如，请求缺少 client_secret 参数。 |
| 403 |授权失败。 例如，用户没有权限访问该资源。 |
| 500 |服务出现内部错误。 重试请求。 |

#### <a name="error-codes-for-token-endpoint-errors"></a>令牌终结点错误的错误代码
| 错误代码 | 说明 | 客户端操作 |
| --- | --- | --- |
| invalid_request |协议错误，例如，缺少必需的参数。 |修复并重新提交请求。 |
| invalid_grant |授权代码无效或已过期。 |请尝试对 `/authorize` 终结点发出新请求 |
| unauthorized_client |经过身份验证的客户端无权使用此权限授予类型。 |客户端应用程序未注册到 Azure AD 中或者未添加到用户的 Azure AD 租户时，通常会出现这种情况。 应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |
| invalid_client |客户端身份验证失败。 |客户端凭据无效。 若要修复，应用程序管理员应更新凭据。 |
| unsupported_grant_type |授权服务器不支持权限授予类型。 |更改请求中的授权类型。 这种类型的错误应该只在开发过程中发生，并且应该在初始测试过程中检测到。 |
| invalid_resource |目标资源无效，原因是它不存在，Azure AD 找不到它，或者未正确配置。 |这表示未在租户中配置该资源（如果存在）。 应用程序可以提示用户，并说明如何安装应用程序并将其添加到 Azure AD。 |
| interaction_required |请求需要用户交互。 例如，需要额外的身份验证步骤。 | 请对同一资源使用交互式授权请求进行重试，而不是非交互式请求。 |
| temporarily_unavailable |服务器暂时繁忙，无法处理请求。 |重试请求。 客户端应用程序可能向用户说明，其响应由于临时状况而延迟。 |

## <a name="use-the-access-token-to-access-the-resource"></a>使用访问令牌访问资源
已经成功获取 `access_token`，现在可以通过在 `Authorization` 标头中包含令牌，在 Web API 的请求中使用令牌。 [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750.txt) 规范说明了如何使用 HTTP 请求中的持有者令牌访问受保护的资源。

### <a name="sample-request"></a>示例请求
```
GET /data HTTP/1.1
Host: service.contoso.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ
```

### <a name="error-response"></a>错误响应
实现 RFC 6750 的受保护资源会发出 HTTP 状态代码。 如果请求不包含身份验证凭据或缺少令牌，则响应将包含 `WWW-Authenticate` 标头。 当某个请求失败时，资源服务器使用 HTTP 状态代码和错误代码做出响应。

以下是当客户端请求不包含持有者令牌时的不成功响应示例：

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer authorization_uri="https://login.partner.microsoftonline.cn/contoso.com/oauth2/authorize",  error="invalid_token",  error_description="The access token is missing.",
```

#### <a name="error-parameters"></a>错误参数
| 参数 | 说明 |
| --- | --- |
| authorization_uri |授权服务器的 URI（物理终结点）。 此值还用作查找键，可从发现终结点获取有关服务器的详细信息。 <p><p> 客户端必须验证授权服务器是否受信任。 Azure AD 对资源进行保护时，只需验证 URL 是否以 Azure AD 支持的 https://login.partner.microsoftonline.cn 或其他主机名开头即可。 特定于租户的资源应始终返回特定于租户的授权 URI。 |
| error |[OAuth 2.0 授权框架](https://tools.ietf.org/html/rfc6749)第 5.2 部分中定义的错误代码值。 |
| error_description |错误的更详细说明。 此消息不是最终用户友好的。 |
| resource_id |返回资源的唯一标识符。 客户端应用程序在请求资源的令牌时，可以使用此标识符作为 `resource` 参数的值。 <p><p> 对于客户端应用程序，验证此值非常重要，否则恶意服务可能会引发 elevation-of-privileges 攻击  <p><p> 防止攻击的建议策略是验证 `resource_id` 是否与正在访问的 Web API URL 基部分相匹配。 例如，如果正在访问 https://service.contoso.com/data ，`resource_id` 可以是 htttps://service.contoso.com/。 客户端应用程序必须拒绝不以基 URL 开头的 `resource_id` ，除非存在可靠的替代方法来验证该 ID。 |

#### <a name="bearer-scheme-error-codes"></a>持有者方案错误代码
RFC 6750 规范为在响应中使用 WWW-Authenticate 标头和持有者方案的资源定义了以下错误。

| HTTP 状态代码 | 错误代码 | 说明 | 客户端操作 |
| --- | --- | --- | --- |
| 400 |invalid_request |请求格式不正确。 例如，请求可能缺少某个参数或者使用了同一参数两次。 |修复错误并重试请求。 这种类型的错误应该只在开发过程中发生，并且应该在初始测试中检测到。 |
| 401 |invalid_token |访问令牌缺失、无效或被吊销。 error_description 参数的值提供了更多详细信息。 |从授权服务器请求新令牌。 如果新令牌失败，则表示发生了意外的错误。 向用户发送一条错误消息，并在随机延迟后重试。 |
| 403 |insufficient_scope |访问令牌不包含访问资源所需的模拟权限。 |将新的授权请求发送到授权终结点。 如果响应包含 scope 参数，则在对资源的请求中使用 scope 值。 |
| 403 |insufficient_access |令牌的使用者没有访问该资源所需的权限。 |提示用户使用其他帐户或请求对指定资源的权限。 |

## <a name="refreshing-the-access-tokens"></a>刷新访问令牌

访问令牌的生存期很短，必须在其过期后刷新才能继续访问资源。 若要刷新 `access_token`，可以向 `/token` 终结点提交另一个 `POST` 请求，但这次要提供 `refresh_token` 而不是 `code`。  刷新令牌对客户端已被同意访问的所有资源有效 - 因此，对 `resource=https://microsoftgraph.chinacloudapi.cn` 请求发出的刷新令牌可用于请求 `resource=https://contoso.com/api` 的新访问令牌。 

刷新令牌没有指定的生存期。 通常，刷新令牌的生存期相对较长。 但是，在某些情况下，刷新令牌会过期、被吊销，或缺少执行所需操作的足够权限。 应用程序需要正确预期和处理令牌颁发终结点返回的错误。

收到具有刷新令牌错误的响应时，请丢弃当前刷新令牌，并请求新的授权代码或访问令牌。 具体而言，在授权代码授予流中使用刷新令牌时，如果收到具有 `interaction_required` 或 `invalid_grant` 错误代码的响应，请丢弃刷新令牌，并请求新的授权代码。

以下示例使用刷新令牌请求特定于租户的终结点（也可以使用公用终结点）获取新的访问令牌：  

```
// Line breaks for legibility only

POST /{tenant}/oauth2/token HTTP/1.1
Host: https://login.partner.microsoftonline.cn
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&refresh_token=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq...
&grant_type=refresh_token
&resource=https%3A%2F%2Fservice.contoso.com%2F
&client_secret=JqQX2PNo9bpM0uEihUPzyrh    // NOTE: Only required for web apps
```

### <a name="successful-response"></a>成功的响应
成功的令牌响应如下：

```
{
  "token_type": "Bearer",
  "expires_in": "3600",
  "expires_on": "1460404526",
  "resource": "https://service.contoso.com/",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ",
  "refresh_token": "AwABAAAAv YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfcUl4VBbiSHZyd1NVZG5QTIOcbObu3qnLutbpadZGAxqjIbMkQ2bQS09fTrjMBtDE3D6kSMIodpCecoANon9b0LATkpitimVCrl PM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4rTfgV29ghDOHRc2B-C_hHeJaJICqjZ3mY2b_YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfmVCrl-NyfN3oyG4ZCWu18M9-vEou4Sq-1oMDzExgAf61noxzkNiaTecM-Ve5cq6wHqYQjfV9DOz4lbceuYCAA"
}
```
| 参数 | 说明 |
| --- | --- |
| token_type |令牌类型。 唯一支持的值为 **bearer**。 |
| expires_in |令牌的剩余生存期，以秒为单位。 典型值为 3600（1 小时）。 |
| expires_on |令牌过期的日期和时间。 该日期表示为自 1970-01-01T0:0:0Z UTC 至过期时间的秒数。 |
| resource |标识可使用访问令牌访问的受保护资源。 |
| scope |向本机客户端应用程序授予的模拟权限。 默认权限是 user_impersonation。  目标资源的所有者可在 Azure AD 中注册备用值。 |
| access_token |所请求的新访问令牌。 |
| refresh_token |一个新的 OAuth 2.0 refresh_token，可用于在此响应中的令牌过期时请求新的访问令牌。 |

### <a name="error-response"></a>错误响应
下面是一个示例错误响应：

```
{
  "error": "invalid_resource",
  "error_description": "AADSTS50001: The application named https://foo.microsoft.com/mail.read was not found in the tenant named 295e01fc-0c56-4ac3-ac57-5d0ed568f872. This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant. You might have sent your authentication request to the wrong tenant.\r\nTrace ID: ef1f89f6-a14f-49de-9868-61bd4072f0a9\r\nCorrelation ID: b6908274-2c58-4e91-aea9-1f6b9c99347c\r\nTimestamp: 2016-04-11 18:59:01Z",
  "error_codes": [
    50001
  ],
  "timestamp": "2016-04-11 18:59:01Z",
  "trace_id": "ef1f89f6-a14f-49de-9868-61bd4072f0a9",
  "correlation_id": "b6908274-2c58-4e91-aea9-1f6b9c99347c"
}
```

| 参数 | 说明 |
| --- | --- |
| error |用于分类发生的错误类型与响应错误的错误码字符串。 |
| error_description |帮助开发人员识别身份验证错误根本原因的特定错误消息。 |
| error_codes |可帮助诊断的 STS 特定错误代码列表。 |
| timestamp |发生错误的时间。 |
| trace_id |可帮助诊断的请求唯一标识符。 |
| correlation_id |可帮助跨组件诊断的请求唯一标识符。 |

有关错误代码的描述和建议的客户端操作，请参阅 [令牌终结点错误的错误代码](#error-codes-for-token-endpoint-errors)。

<!-- Update_Description: wording update -->