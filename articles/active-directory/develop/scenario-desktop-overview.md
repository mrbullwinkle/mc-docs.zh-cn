---
title: 调用 Web API 的桌面应用（概述）- Microsoft 标识平台
description: 了解如何构建调用 Web API 的桌面应用（概述）
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
origin.date: 05/07/2019
ms.date: 06/19/2019
ms.author: v-junlch
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 62fb3c672bfb9bfa99344e21d0dee1257432f2c2
ms.sourcegitcommit: 9d5fd3184b6a47bf3b60ffdeeee22a08354ca6b1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/21/2019
ms.locfileid: "67305948"
---
# <a name="scenario-desktop-app-that-calls-web-apis"></a>方案：用于调用 Web API 的 桌面应用

了解构建调用 Web API 的桌面应用所需的一切

## <a name="prerequisites"></a>先决条件

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="getting-started"></a>入门

如果还未创建第一个应用程序，请按 .NET 桌面快速入门或 UWP 快速入门的说明来这样做：

> [!div class="nextstepaction"]
> [快速入门：获取令牌并从 Windows 桌面应用中调用 Microsoft Graph API](./quickstart-v2-windows-desktop.md)


> [!div class="nextstepaction"]
> [快速入门：获取令牌并从 UWP 应用中调用 Microsoft Graph API](./quickstart-v2-uwp.md)

## <a name="overview"></a>概述

你编写桌面应用程序，需要将用户登录到应用程序并调用 Web API，例如 Microsoft Graph、其他 Microsoft API 或你自己的 Web API。 有几种可能性：

- 如果桌面应用程序支持图形控制，例如，它是一个 Windows.Form 应用程序或 WPF 应用程序，则可使用交互式令牌获取。
- 对于 Windows 托管的应用程序，也可让加入到 Windows 域或加入 AAD 的计算机上运行的应用程序使用集成 Windows 身份验证以无提示方式获取令牌。
- 最后，可以在公共客户端应用程序中使用用户名/密码，虽然不建议使用这种方式。 在某些情况下，这仍然是必需的，但请注意，使用它会对应用程序施加约束。 例如，它不能登录需要执行多重身份验证的用户。 另外，应用程序无法利用单一登录 (SSO)。

  它也违反新式身份验证的原则，提供它只是出于历史原因。

  ![桌面应用程序](./media/scenarios/desktop-app.svg)

- 如果编写可移植的命令行工具（可能是一个在 Linux 或 Mac 上运行的 .NET Core），则既不能使用交互式身份验证（因为 .NET Core 不提供 [Web 浏览器](https://aka.ms/msal-net-uses-web-browser)），也不能使用集成 Windows 身份验证。 在这种情况下，最佳选择是使用设备代码流。 该流也用于无浏览器的应用程序，例如 iOT 应用程序

  ![无浏览器应用程序](./media/scenarios/device-code-flow-app.svg)

## <a name="specifics"></a>详情

桌面应用程序有许多具体要求，主要取决于应用程序是否使用交互式身份验证。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [桌面应用 - 应用注册](scenario-desktop-app-registration.md)

