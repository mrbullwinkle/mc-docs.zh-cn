---
title: Xamarin.Forms 应用中的移动应用身份验证入门
description: 了解如何使用移动应用通过各种标识提供者（包括 AAD、Google、Facebook、Twitter 和 Microsoft）对 Xamarin Forms 应用的用户进行身份验证。
services: app-service\mobile
documentationcenter: xamarin
author: elamalani
manager: crdun
editor: ''
ms.assetid: 9c55e192-c761-4ff2-8d88-72260e9f6179
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin
ms.devlang: dotnet
ms.topic: article
origin.date: 08/07/2017
ms.author: v-biyu
ms.date: 07/15/2019
ms.openlocfilehash: d783af5e6bcc22d9d892fdd43bfbb971cc5caba7
ms.sourcegitcommit: a829f1191e40d8940a5bf6074392973128cfe3c0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/04/2019
ms.locfileid: "67560282"
---
# <a name="add-authentication-to-your-xamarin-forms-app"></a>向 Xamarin Forms 应用添加身份验证
[!INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]


## <a name="overview"></a>概述
本主题演示如何从客户端应用程序对应用服务移动应用的用户进行身份验证。 在本教程中，使用应用服务支持的标识提供者向 Xamarin Forms 快速入门项目添加身份验证。 移动应用成功进行身份验证和授权后，将显示用户 ID 值，该用户能够访问受限制的表数据。

## <a name="prerequisites"></a>先决条件
为了使本教程达到最佳效果，建议你先完成[创建 Xamarin Forms 应用][1]教程。 完成此教程后，用户可以获得一个 Xamarin Forms 项目，它是一个多平台 TodoList 应用。

如果不使用下载的快速入门服务器项目，必须将身份验证扩展包添加到项目。 有关服务器扩展包的详细信息，请参阅[使用适用于 Azure 移动应用的 .NET 后端服务器 SDK][2]。

## <a name="register-your-app-for-authentication-and-configure-app-services"></a>注册应用以进行身份验证并配置应用服务
[!INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

## <a name="redirecturl"></a>将应用添加到允许的外部重定向 URL

安全身份验证要求为应用定义新的 URL 方案。 此方案允许在完成身份验证过程后，身份验证系统重定向到应用。 在本教程中，我们自始至终使用 URL 方案 _appname_ 。 但是，可以使用任何你所选的 URL 方案。 对于移动应用程序而言，它应是唯一的。 在服务器端启用重定向：

1. 在 [Azure 门户][8]中，选择应用服务。

2. 单击“身份验证/授权”  菜单选项。

3. 在“允许的外部重定向 URL”  中，输入 `url_scheme_of_your_app://easyauth.callback`。  此字符串中的 **url_scheme_of_your_app** 是移动应用程序的 URL 方案。  它应该遵循协议的正常 URL 规范（仅使用字母和数字，并以字母开头）。  应记下此字符串，因为在一些地方需要使用此 URL 方案调整移动应用代码。

4. 单击 **“确定”** 。

5. 单击“保存”  。

## <a name="restrict-permissions-to-authenticated-users"></a>将权限限制给已经过身份验证的用户
[!INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

## <a name="add-authentication-to-the-portable-class-library"></a>向可移植类库添加身份验证
移动应用使用 [LoginAsync][3] extension method on the [MobileServiceClient][4] to sign in a user with App Service authentication. This sample
uses a server-managed authentication flow that displays the provider's sign-in interface in the app. For more information, see [Server-managed authentication][5]。 若要在生产应用中提供更好的用户体验，应考虑改用[客户端托管的身份验证][6]。

若要使用 Xamarin Forms 项目进行身份验证，请在可移植类库中为应用定义 **IAuthenticate** 接口。 然后，将“登录”按钮添加到可移植类库中定义的用户界面，用户单击此按钮即可开始进行身份验证  。 身份验证成功后，将从移动应用后端加载数据。

为应用支持的每个平台实现 **IAuthenticate** 接口。

1. 在 Visual Studio 或 Xamarin Studio 中，从名称中包含 Portable 的项目（该项目是可移植类库项目）中打开 App.cs，然后添加以下 `using` 语句  ：

    ```
    using System.Threading.Tasks;
    ```
2. 在 App.cs 中，在 `App` 类定义前添加以下 `IAuthenticate` 接口定义。

    ```
    public interface IAuthenticate
    {
        Task<bool> Authenticate();
    }
    ```

3. 若要使用平台特定的实现初始化接口，可向 **App** 类添加以下静态成员。

    ```
    public static IAuthenticate Authenticator { get; private set; }

    public static void Init(IAuthenticate authenticator)
    {
        Authenticator = authenticator;
    }
    ```

4. 从可移植类库项目中打开 TodoList.xaml，在 **buttonsPanel** 布局元素中现有按钮之后添加以下 *Button* 元素： 

    ```
      <Button x:Name="loginButton" Text="Sign-in" MinimumHeightRequest="30" 
        Clicked="loginButton_Clicked"/>
    ```

    此按钮通过移动应用后端触发服务器托管的身份验证。

5. 从可移植类库项目中打开 TodoList.xaml.cs，并将以下字段添加到 `TodoList` 类：

    ```
    // Track whether the user has authenticated. 
    bool authenticated = false;
    ```

6. 将 **OnAppearing** 方法替换为以下代码：

    ```
    protected override async void OnAppearing()
    {
        base.OnAppearing();

        // Refresh items only when authenticated.
        if (authenticated == true)
        {
            // Set syncItems to true in order to synchronize the data 
            // on startup when running in offline mode.
            await RefreshItems(true, syncItems: false);

            // Hide the Sign-in button.
            this.loginButton.IsVisible = false;
        }
    }
    ```

    该代码可确保仅在用户经过身份验证后，才从服务刷新数据。
7. 在 TodoList 类中为 Clicked 事件添加以下处理程序   ：

    ```
    async void loginButton_Clicked(object sender, EventArgs e)
    {
        if (App.Authenticator != null)
            authenticated = await App.Authenticator.Authenticate();

        // Set syncItems to true to synchronize the data on startup when offline is enabled.
        if (authenticated == true)
            await RefreshItems(true, syncItems: false);
    }
    ```

8. 保存更改，重新生成可移植类库项目，并验证没有错误。

## <a name="add-authentication-to-the-android-app"></a>向 Android 应用添加身份验证
本部分演示如何在 Android 应用项目中实现 **IAuthenticate** 接口。 如果不要支持 Android 设备，请跳过本部分。

1. 在 Visual Studio 或 Xamarin Studio 中，右键单击 droid 项目，然后单击“设为启动项目”   。
2. 按 F5 在调试器中启动项目，然后验证启动该应用后，是否会引发状态代码为 401（“未授权”）的未处理异常。 因为后端上的访问仅限于授权用户，因此会生成 401 代码。
3. 在 Android 项目中打开 MainActivity.cs，并添加以下 `using` 语句：

    ```
    using Microsoft.WindowsAzure.MobileServices;
    using System.Threading.Tasks;
    ```

4. 更新 MainActivity 类，实现 IAuthenticate 接口，如下所示   ：

    ```
    public class MainActivity : global::Xamarin.Forms.Platform.Android.FormsApplicationActivity, IAuthenticate
    ```

5. 通过添加 MobileServiceUser 字段和 IAuthenticate 接口所需的 Authenticate 方法，更新 MainActivity 类，如下所示     ：

    ```
    // Define a authenticated user.
    private MobileServiceUser user;

    public async Task<bool> Authenticate()
    {
        var success = false;
        var message = string.Empty;
        try
        {
                // Sign in with MicrosoftAccount login using a server-managed flow.
            user = await TodoItemManager.DefaultManager.CurrentClient.LoginAsync(this,
                    MobileServiceAuthenticationProvider.MicrosoftAccount, "{url_scheme_of_your_app}");
            if (user != null)
            {
                message = string.Format("you are now signed-in as {0}.",
                    user.UserId);
                success = true;
            }
        }
        catch (Exception ex)
        {
            message = ex.Message;
        }

        // Display the success or failure message.
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.SetMessage(message);
        builder.SetTitle("Sign-in result");
        builder.Create().Show();

        return success;
    }
    ```

    如果使用的是 MicrosoftAccount 以外的其他标识提供者，请为 [MobileServiceAuthenticationProvider] 选择不同的值。

6. 在 `<application>` 元素内添加以下 XML，更新 **AndroidManifest.xml** 文件：

    ```xml
    <activity android:name="com.microsoft.windowsazure.mobileservices.authentication.RedirectUrlActivity" android:launchMode="singleTop" android:noHistory="true">
      <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="{url_scheme_of_your_app}" android:host="easyauth.callback" />
      </intent-filter>
    </activity>
    ```
    将 `{url_scheme_of_your_app}` 替换为 URL 方案。
7. 调用 `LoadApplication()` 之前，向 MainActivity 类的 OnCreate 方法添加以下代码   ：

        // Initialize the authenticator before loading the app.
        App.Init((IAuthenticate)this);

    此代码可确保验证器在应用加载前进行初始化。
8. 重新生成应用，运行它，使用所选的身份验证提供者登录，并验证是否能够以经过身份验证的用户身份访问数据。

### <a name="troubleshooting"></a>故障排除

**应用程序崩溃并显示 `Java.Lang.NoSuchMethodError: No static method startActivity`**

在某些情况下，支持包中的冲突在 Visual Studio 中仅显示为警告，但应用程序在运行时会崩溃并显示此异常。 在这种情况下，你需要确保在项目中引用的所有支持包都具有相同的版本。 对于 Android 平台，[Azure 移动应用 NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client/)具有 `Xamarin.Android.Support.CustomTabs` 依赖项，因此，如果你的项目使用较新的支持包，则你需要直接安装具有所需版本的此包以避免冲突。

## <a name="add-authentication-to-the-ios-app"></a>向 iOS 应用添加身份验证
本部分演示如何在 iOS 应用项目中实现 **IAuthenticate** 接口。 如果不要支持 iOS 设备，请跳过本部分。

1. 在 Visual Studio 或 Xamarin Studio 中，右键单击 iOS 项目，然后单击“设为启动项目”   。
2. 按 F5 在调试器中启动项目，然后验证启动该应用后，是否会引发状态代码为 401（“未授权”）的未处理异常。 之所以会生成 401 响应，是因为对后端的访问仅限于授权用户。
3. 打开 iOS 项目中的 AppDelegate.cs，并添加以下 `using` 语句：

    ```
    using Microsoft.WindowsAzure.MobileServices;
    using System.Threading.Tasks;
    ```
4. 更新 AppDelegate 类，实现 IAuthenticate 接口，如下所示   ：

    ```
    public partial class AppDelegate : global::Xamarin.Forms.Platform.iOS.FormsApplicationDelegate, IAuthenticate
    ```
5. 通过添加 MobileServiceUser 字段和 IAuthenticate 接口所需的 Authenticate 方法，更新 AppDelegate 类，如下所示     ：

    ```
    // Define a authenticated user.
    private MobileServiceUser user;

    public async Task<bool> Authenticate()
    {
        var success = false;
        var message = string.Empty;
        try
        {
            // Sign in with MicrosoftAccount login using a server-managed flow.
            if (user == null)
            {
                user = await TodoItemManager.DefaultManager.CurrentClient
                    .LoginAsync(UIApplication.SharedApplication.KeyWindow.RootViewController,
                        MobileServiceAuthenticationProvider.MicrosoftAccount, "{url_scheme_of_your_app}");
                if (user != null)
                {
                    message = string.Format("You are now signed-in as {0}.", user.UserId);
                    success = true;                        
                }
            }        
        }
        catch (Exception ex)
        {
           message = ex.Message;
        }

        // Display the success or failure message.
        UIAlertView avAlert = new UIAlertView("Sign-in result", message, null, "OK", null);
        avAlert.Show();         

        return success;
    }
    ```

    如果使用的是 MicrosoftAccount 以外的其他标识提供者，请为 [MobileServiceAuthenticationProvider] 选择不同的值。

6. 通过添加 **OpenUrl** 方法重载更新 **AppDelegate** 类，如下所示：

        public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
        {
            return TodoItemManager.DefaultManager.CurrentClient.ResumeWithURL(url);
        }

6. 调用 `LoadApplication()` 之前，向 FinishedLaunching 方法添加以下代码行  ：

    ```
    App.Init(this);
    ```

    该代码可确保验证器在应用加载前进行初始化。

8. 打开 Info.plist 并添加 **URL 类型**。 将“标识符”设置  为所选的名称，将“URL 方案”  设置为应用的 URL 方案，将“角色”  设置为“无”。

7. 重新生成应用，运行它，使用所选的身份验证提供者登录，并验证是否能够以经过身份验证的用户身份访问数据。

## <a name="add-authentication-to-windows-10-including-phone-app-projects"></a>向 Windows 10（包括 Phone）应用项目添加身份验证
本部分演示如何在 Windows 10 应用项目中实现“IAuthenticate”接口。  相同的步骤适用于通用 Windows 平台 (UWP) 项目，但使用的是 UWP 项目（具有已注明的更改）  。 如果不要支持 Windows 设备，请跳过本部分。

1. 在 Visual Studio 中，右键单击“UWP”项目，然后单击“设为启动项目”。  
2. 按 F5 在调试器中启动项目，然后验证启动该应用后，是否会引发状态代码为 401（“未授权”）的未处理异常。 因为后端上的访问仅限于授权用户，因此会发生 401 响应。
3. 打开 Windows 应用项目的 MainPage.xaml.cs，并添加以下 `using` 语句：

    ```
    using Microsoft.WindowsAzure.MobileServices;
    using System.Threading.Tasks;
    using Windows.UI.Popups;
    using <your_Portable_Class_Library_namespace>;
    ```

    将 `<your_Portable_Class_Library_namespace>` 替换为可移植类库的命名空间。

4. 更新 MainPage 类，实现 IAuthenticate 接口，如下所示   ：

    ```
    public sealed partial class MainPage : IAuthenticate
    ```
5. 通过添加 MobileServiceUser 字段和 IAuthenticate 接口所需的 Authenticate 方法，更新 MainPage 类，如下所示     ：

    ```
    // Define a authenticated user.
    private MobileServiceUser user;

    public async Task<bool> Authenticate()
    {
        string message = string.Empty;
        var success = false;

        try
        {
            // Sign in with MicrosoftAccount login using a server-managed flow.
            if (user == null)
            {
                user = await TodoItemManager.DefaultManager.CurrentClient
                        .LoginAsync(MobileServiceAuthenticationProvider.MicrosoftAccount, "{url_scheme_of_your_app}");
                if (user != null)
                {
                    success = true;
                    message = string.Format("You are now signed-in as {0}.", user.UserId);
                }
            }

        }
        catch (Exception ex)
        {
            message = string.Format("Authentication Failed: {0}", ex.Message);
        }

        // Display the success or failure message.
        await new MessageDialog(message, "Sign-in result").ShowAsync();

        return success;
    }
    ```

    如果使用的是 MicrosoftAccount 以外的其他标识提供者，请为 [MobileServiceAuthenticationProvider] 选择不同的值。

6. 调用 `LoadApplication()` 之前，在 MainPage 类的构造函数中添加以下代码行  ：

    ```
    // Initialize the authenticator before loading the app.
    <your_Portable_Class_Library_namespace>.App.Init(this);
    ```

    将 `<your_Portable_Class_Library_namespace>` 替换为可移植类库的命名空间。

3. 如果使用的是“UWP”，则将以下“OnActivated”方法重写添加到“App”类：   

       protected override void OnActivated(IActivatedEventArgs args)
       {
           base.OnActivated(args);

            if (args.Kind == ActivationKind.Protocol)
            {
                ProtocolActivatedEventArgs protocolArgs = args as ProtocolActivatedEventArgs;
                MobileServiceClientExtensions.ResumeWithURL(TodoItemManager.DefaultManager.CurrentClient,protocolArgs.Uri);
            }
       }

3. 打开 Package.appxmanifest，添加 **Protocol** 声明。 将“显示名称”设置  为所选的名称，将“名称”  设置为应用的 URL 方案。

4. 重新生成应用，运行它，使用所选的身份验证提供者登录，并验证是否能够以经过身份验证的用户身份访问数据。

## <a name="next-steps"></a>后续步骤
完成此基本身份验证教程后，请考虑继续学习以下教程之一：

* [向应用添加推送通知](app-service-mobile-xamarin-forms-get-started-push.md)

  了解如何为应用添加推送通知支持，以及如何将移动应用后端配置为使用 Azure 通知中心发送推送通知。
* [为应用启用脱机同步](app-service-mobile-xamarin-forms-get-started-offline-data.md)

  了解如何使用移动应用后端向应用添加脱机支持。 脱机同步允许最终用户与移动应用交互（查看、添加或修改数据），即使在没有网络连接时也是如此。

<!-- Images. -->

<!-- URLs. -->
[1]: ./app-service-mobile-xamarin-forms-get-started.md
[2]: ./app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[3]: https://msdn.microsoft.com/zh-cn/library/azure/dn268341(v=azure.10).aspx
[4]: https://msdn.microsoft.com/zh-cn/library/azure/JJ553674(v=azure.10).aspx
[5]: ./app-service-mobile-dotnet-how-to-use-client-library.md#serverflow
[6]: ./app-service-mobile-dotnet-how-to-use-client-library.md#clientflow
[7]: https://msdn.microsoft.com/zh-cn/library/azure/jj730936(v=azure.10).aspx
[8]: https://portal.azure.cn

<!--Update_Description:add the section of "Add app to the Allowed External Redirect URLs" and update some code-->