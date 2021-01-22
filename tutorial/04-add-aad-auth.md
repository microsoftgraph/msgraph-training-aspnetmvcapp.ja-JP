<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="be202-101">この演習では、前の演習のアプリケーションを拡張して、Azure AD での認証をサポートします。</span><span class="sxs-lookup"><span data-stu-id="be202-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="be202-102">これは、Microsoft Graph API を呼び出すのに必要な OAuth アクセス トークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="be202-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="be202-103">この手順では、OWIN ミドルウェアと Microsoft [認証](https://www.nuget.org/packages/Microsoft.Identity.Client/) ライブラリ ライブラリをアプリケーションに統合します。</span><span class="sxs-lookup"><span data-stu-id="be202-103">In this step you will integrate the OWIN middleware and the [Microsoft Authentication Library](https://www.nuget.org/packages/Microsoft.Identity.Client/) library into the application.</span></span>

1. <span data-ttu-id="be202-104">ソリューション エクスプローラーで **グラフ チュートリアル プロジェクトを右クリック** し、[新しいアイテムの> **を選択します**。[Web **構成ファイル] を選択し**、ファイルに名前を付 `PrivateSettings.config` け、[追加] を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="be202-104">Right-click the **graph-tutorial** project in Solution Explorer and select **Add > New Item...**. Choose **Web Configuration File**, name the file `PrivateSettings.config` and select **Add**.</span></span> <span data-ttu-id="be202-105">すべてのコンテンツを次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="be202-105">Replace its entire contents with the following code.</span></span>

    ```xml
    <appSettings>
        <add key="ida:AppID" value="YOUR APP ID" />
        <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
        <add key="ida:RedirectUri" value="https://localhost:PORT/" />
        <add key="ida:AppScopes" value="User.Read Calendars.Read" />
    </appSettings>
    ```

    <span data-ttu-id="be202-106">アプリケーション `YOUR_APP_ID_HERE` 登録ポータルのアプリケーション ID に置き換え、生成したクライアント シークレット `YOUR_APP_PASSWORD_HERE` に置き換える。</span><span class="sxs-lookup"><span data-stu-id="be202-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the client secret you generated.</span></span> <span data-ttu-id="be202-107">クライアント シークレットにアンパサンド (`&`) が含まれている場合は、`PrivateSettings.config` で `&amp;` に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="be202-107">If your client secret contains any ampersands (`&`), be sure to replace them with `&amp;` in `PrivateSettings.config`.</span></span> <span data-ttu-id="be202-108">また、`ida:RedirectUri` の `PORT` 値をアプリケーションの URL に合わせて変更してください。</span><span class="sxs-lookup"><span data-stu-id="be202-108">Also be sure to modify the `PORT` value for the `ida:RedirectUri` to match your application's URL.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="be202-109">git などのソース管理を使用している場合は、アプリ ID とパスワードが誤って漏洩しないように、ファイルをソース管理から除外する良い時期です `PrivateSettings.config` 。</span><span class="sxs-lookup"><span data-stu-id="be202-109">If you're using source control such as git, now would be a good time to exclude the `PrivateSettings.config` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="be202-110">この `Web.config` 新しいファイルを読み込む更新。</span><span class="sxs-lookup"><span data-stu-id="be202-110">Update `Web.config` to load this new file.</span></span> <span data-ttu-id="be202-111">`<appSettings>` (7 行目) を以下のように置き換えます</span><span class="sxs-lookup"><span data-stu-id="be202-111">Replace the `<appSettings>` (line 7) with the following</span></span>

    ```xml
    <appSettings file="PrivateSettings.config">
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="be202-112">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="be202-112">Implement sign-in</span></span>

<span data-ttu-id="be202-113">まず OWIN ミドルウェアを初期化して、アプリに Azure AD 認証を使用します。</span><span class="sxs-lookup"><span data-stu-id="be202-113">Start by initializing the OWIN middleware to use Azure AD authentication for the app.</span></span>

1. <span data-ttu-id="be202-114">ソリューション エクスプローラーで **App_Start** フォルダーを右クリックし、[クラスの追加 **>選択します**。ファイルに名前を付 `Startup.Auth.cs` け、[追加] を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="be202-114">Right-click the **App_Start** folder in Solution Explorer and select **Add > Class...**. Name the file `Startup.Auth.cs` and select **Add**.</span></span> <span data-ttu-id="be202-115">すべての内容を、次のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="be202-115">Replace the entire contents with the following code.</span></span>

    ```cs
    using Microsoft.Identity.Client;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.IdentityModel.Tokens;
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.Notifications;
    using Microsoft.Owin.Security.OpenIdConnect;
    using Owin;
    using System.Configuration;
    using System.Threading.Tasks;
    using System.Web;

    namespace graph_tutorial
    {
        public partial class Startup
        {
            // Load configuration settings from PrivateSettings.config
            private static string appId = ConfigurationManager.AppSettings["ida:AppId"];
            private static string appSecret = ConfigurationManager.AppSettings["ida:AppSecret"];
            private static string redirectUri = ConfigurationManager.AppSettings["ida:RedirectUri"];
            private static string graphScopes = ConfigurationManager.AppSettings["ida:AppScopes"];

            public void ConfigureAuth(IAppBuilder app)
            {
                app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

                app.UseCookieAuthentication(new CookieAuthenticationOptions());

                app.UseOpenIdConnectAuthentication(
                    new OpenIdConnectAuthenticationOptions
                    {
                        ClientId = appId,
                        Authority = "https://login.microsoftonline.com/common/v2.0",
                        Scope = $"openid email profile offline_access {graphScopes}",
                        RedirectUri = redirectUri,
                        PostLogoutRedirectUri = redirectUri,
                        TokenValidationParameters = new TokenValidationParameters
                        {
                            // For demo purposes only, see below
                            ValidateIssuer = false

                            // In a real multi-tenant app, you would add logic to determine whether the
                            // issuer was from an authorized tenant
                            //ValidateIssuer = true,
                            //IssuerValidator = (issuer, token, tvp) =>
                            //{
                            //  if (MyCustomTenantValidation(issuer))
                            //  {
                            //    return issuer;
                            //  }
                            //  else
                            //  {
                            //    throw new SecurityTokenInvalidIssuerException("Invalid issuer");
                            //  }
                            //}
                        },
                        Notifications = new OpenIdConnectAuthenticationNotifications
                        {
                            AuthenticationFailed = OnAuthenticationFailedAsync,
                            AuthorizationCodeReceived = OnAuthorizationCodeReceivedAsync
                        }
                    }
                );
            }

            private static Task OnAuthenticationFailedAsync(AuthenticationFailedNotification<OpenIdConnectMessage,
              OpenIdConnectAuthenticationOptions> notification)
            {
                notification.HandleResponse();
                string redirect = $"/Home/Error?message={notification.Exception.Message}";
                if (notification.ProtocolMessage != null && !string.IsNullOrEmpty(notification.ProtocolMessage.ErrorDescription))
                {
                    redirect += $"&debug={notification.ProtocolMessage.ErrorDescription}";
                }
                notification.Response.Redirect(redirect);
                return Task.FromResult(0);
            }

            private async Task OnAuthorizationCodeReceivedAsync(AuthorizationCodeReceivedNotification notification)
            {
                var idClient = ConfidentialClientApplicationBuilder.Create(appId)
                    .WithRedirectUri(redirectUri)
                    .WithClientSecret(appSecret)
                    .Build();

                string message;
                string debug;

                try
                {
                    string[] scopes = graphScopes.Split(' ');

                    var result = await idClient.AcquireTokenByAuthorizationCode(
                        scopes, notification.Code).ExecuteAsync();

                    message = "Access token retrieved.";
                    debug = result.AccessToken;
                }
                catch (MsalException ex)
                {
                    message = "AcquireTokenByAuthorizationCodeAsync threw an exception";
                    debug = ex.Message;
                }

                notification.HandleResponse();
                notification.Response.Redirect($"/Home/Error?message={message}&debug={debug}");
            }
        }
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="be202-116">このコードは、OWIN ミドルウェアの値を使用して構成し、2 つのコールバック `PrivateSettings.config` メソッドを定義 `OnAuthenticationFailedAsync` します `OnAuthorizationCodeReceivedAsync` 。</span><span class="sxs-lookup"><span data-stu-id="be202-116">This code configures the OWIN middleware with the values from `PrivateSettings.config` and defines two callback methods, `OnAuthenticationFailedAsync` and `OnAuthorizationCodeReceivedAsync`.</span></span> <span data-ttu-id="be202-117">これらのコールバック メソッドは、サインイン プロセスが Azure から戻るときに呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="be202-117">These callback methods will be invoked when the sign-in process returns from Azure.</span></span>

1. <span data-ttu-id="be202-118">次に、メソッド `Startup.cs` を呼び出すファイルを更新 `ConfigureAuth` します。</span><span class="sxs-lookup"><span data-stu-id="be202-118">Now update the `Startup.cs` file to call the `ConfigureAuth` method.</span></span> <span data-ttu-id="be202-119">内容全体を次の `Startup.cs` コードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="be202-119">Replace the entire contents of `Startup.cs` with the following code.</span></span>

    ```cs
    using Microsoft.Owin;
    using Owin;

    [assembly: OwinStartup(typeof(graph_tutorial.Startup))]

    namespace graph_tutorial
    {
        public partial class Startup
        {
            public void Configuration(IAppBuilder app)
            {
                ConfigureAuth(app);
            }
        }
    }
    ```

1. <span data-ttu-id="be202-120">`Error` アクションを `HomeController` クラスに追加し、`message` クエリ パラメーターおよび `debug` クエリ パラメーターを `Alert` オブジェクトに変換します。</span><span class="sxs-lookup"><span data-stu-id="be202-120">Add an `Error` action to the `HomeController` class to transform the `message` and `debug` query parameters into an `Alert` object.</span></span> <span data-ttu-id="be202-121">次 `Controllers/HomeController.cs` の関数を開いて追加します。</span><span class="sxs-lookup"><span data-stu-id="be202-121">Open `Controllers/HomeController.cs` and add the following function.</span></span>

    ```cs
    public ActionResult Error(string message, string debug)
    {
        Flash(message, debug);
        return RedirectToAction("Index");
    }
    ```

1. <span data-ttu-id="be202-122">コントローラーを追加してサインインを処理します。</span><span class="sxs-lookup"><span data-stu-id="be202-122">Add a controller to handle sign-in.</span></span> <span data-ttu-id="be202-123">ソリューション エクスプローラーで **[コントローラー]** フォルダーを右クリックし、**[追加] > [コントローラー...]** の順に選択します。**[MVC 5 コントローラー - 空]** を選び、**[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="be202-123">Right-click the **Controllers** folder in Solution Explorer and select **Add > Controller...**. Choose **MVC 5 Controller - Empty** and select **Add**.</span></span> <span data-ttu-id="be202-124">コントローラーの名前を `AccountController` に指定し、**[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="be202-124">Name the controller `AccountController` and select **Add**.</span></span> <span data-ttu-id="be202-125">ファイルのすべての内容を次のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="be202-125">Replace the entire contents of the file with the following code.</span></span>

    ```cs
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.OpenIdConnect;
    using System.Security.Claims;
    using System.Web;
    using System.Web.Mvc;

    namespace graph_tutorial.Controllers
    {
        public class AccountController : Controller
        {
            public void SignIn()
            {
                if (!Request.IsAuthenticated)
                {
                    // Signal OWIN to send an authorization request to Azure
                    Request.GetOwinContext().Authentication.Challenge(
                        new AuthenticationProperties { RedirectUri = "/" },
                        OpenIdConnectAuthenticationDefaults.AuthenticationType);
                }
            }

            public ActionResult SignOut()
            {
                if (Request.IsAuthenticated)
                {
                    Request.GetOwinContext().Authentication.SignOut(
                        CookieAuthenticationDefaults.AuthenticationType);
                }

                return RedirectToAction("Index", "Home");
            }
        }
    }
    ```

    <span data-ttu-id="be202-126">これにより、`SignIn` アクションと `SignOut` アクションが定義されます。</span><span class="sxs-lookup"><span data-stu-id="be202-126">This defines a `SignIn` and `SignOut` action.</span></span> <span data-ttu-id="be202-127">`SignIn` アクションは、要求が既に認証されているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="be202-127">The `SignIn` action checks if the request is already authenticated.</span></span> <span data-ttu-id="be202-128">認証されていなければ、OWIN ミドルウェアを呼び出してユーザーを認証します。</span><span class="sxs-lookup"><span data-stu-id="be202-128">If not, it invokes the OWIN middleware to authenticate the user.</span></span> <span data-ttu-id="be202-129">`SignOut` アクションは、サインアウトする OWIN ミドルウェアを呼び出します。</span><span class="sxs-lookup"><span data-stu-id="be202-129">The `SignOut` action invokes the OWIN middleware to sign out.</span></span>

1. <span data-ttu-id="be202-130">変更を保存してプロジェクトを開始します。</span><span class="sxs-lookup"><span data-stu-id="be202-130">Save your changes and start the project.</span></span> <span data-ttu-id="be202-131">[サインイン] ボタンをクリックすると、`https://login.microsoftonline.com` にリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="be202-131">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="be202-132">Microsoft アカウントでログインし、要求されたアクセス許可に同意します。</span><span class="sxs-lookup"><span data-stu-id="be202-132">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="be202-133">ブラウザーがアプリにリダイレクトし、トークンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="be202-133">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="be202-134">ユーザーの詳細情報を取得する</span><span class="sxs-lookup"><span data-stu-id="be202-134">Get user details</span></span>

<span data-ttu-id="be202-135">ユーザーがログインすると、Microsoft Graph からそのユーザーの情報を入手できます。</span><span class="sxs-lookup"><span data-stu-id="be202-135">Once the user is logged in, you can get their information from Microsoft Graph.</span></span>

1. <span data-ttu-id="be202-136">ソリューション エクスプローラーで **graph-tutorial** フォルダーを右クリックし、**[追加] > [新しいフォルダー]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="be202-136">Right-click the **graph-tutorial** folder in Solution Explorer, and select **Add > New Folder**.</span></span> <span data-ttu-id="be202-137">フォルダーに `Helpers` という名前を指定します。</span><span class="sxs-lookup"><span data-stu-id="be202-137">Name the folder `Helpers`.</span></span>

1. <span data-ttu-id="be202-138">この新しいフォルダーを右クリックし、[ **クラスの追加>選択します**。ファイルに名前を付 `GraphHelper.cs` け、[追加] を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="be202-138">Right click this new folder and select **Add > Class...**. Name the file `GraphHelper.cs` and select **Add**.</span></span> <span data-ttu-id="be202-139">このファイルの内容を次のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="be202-139">Replace the contents of this file with the following code.</span></span>

    ```cs
    using Microsoft.Graph;
    using System.Net.Http.Headers;
    using System.Threading.Tasks;

    namespace graph_tutorial.Helpers
    {
        public static class GraphHelper
        {
            public static async Task<User> GetUserDetailsAsync(string accessToken)
            {
                var graphClient = new GraphServiceClient(
                    new DelegateAuthenticationProvider(
                        (requestMessage) =>
                        {
                            requestMessage.Headers.Authorization =
                                new AuthenticationHeaderValue("Bearer", accessToken);
                            return Task.FromResult(0);
                        }));

                return await graphClient.Me.Request().GetAsync();
            }
        }
    }
    ```

    <span data-ttu-id="be202-140">これにより、`GetUserDetails` 関数が実装されます。これは、Microsoft Graph SDK を使用して `/me` エンドポイントを呼び出して結果を返します。</span><span class="sxs-lookup"><span data-stu-id="be202-140">This implements the `GetUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

1. <span data-ttu-id="be202-141">この関数 `OnAuthorizationCodeReceivedAsync` を呼び出 `App_Start/Startup.Auth.cs` すメソッドを更新します。</span><span class="sxs-lookup"><span data-stu-id="be202-141">Update the `OnAuthorizationCodeReceivedAsync` method in `App_Start/Startup.Auth.cs` to call this function.</span></span> <span data-ttu-id="be202-142">次の `using` ステートメントをファイルの一番上に追加します。</span><span class="sxs-lookup"><span data-stu-id="be202-142">Add the following `using` statement to the top of the file.</span></span>

    ```cs
    using graph_tutorial.Helpers;
    ```

1. <span data-ttu-id="be202-143">`OnAuthorizationCodeReceivedAsync` の既存の `try` ブロックを、以下のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="be202-143">Replace the existing `try` block in `OnAuthorizationCodeReceivedAsync` with the following code.</span></span>

    ```cs
    try
    {
        string[] scopes = graphScopes.Split(' ');

        var result = await idClient.AcquireTokenByAuthorizationCode(
            scopes, notification.Code).ExecuteAsync();

        var userDetails = await GraphHelper.GetUserDetailsAsync(result.AccessToken);

        string email = string.IsNullOrEmpty(userDetails.Mail) ?
            userDetails.UserPrincipalName : userDetails.Mail;

        message = "User info retrieved.";
        debug = $"User: {userDetails.DisplayName}, Email: {email}";
    }
    ```

1. <span data-ttu-id="be202-144">変更を保存してアプリを起動すると、サインイン後、アクセス トークンの代わりにユーザーの名前とメール アドレスが表示されます。</span><span class="sxs-lookup"><span data-stu-id="be202-144">Save your changes and start the app, after sign-in you should see the user's name and email address instead of the access token.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="be202-145">トークンの格納</span><span class="sxs-lookup"><span data-stu-id="be202-145">Storing the tokens</span></span>

<span data-ttu-id="be202-146">トークンを取得できるようになったので、トークンをアプリに格納する手順を実装します。</span><span class="sxs-lookup"><span data-stu-id="be202-146">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="be202-147">これはサンプル アプリです。トークンの保存にはセッションを使用します。</span><span class="sxs-lookup"><span data-stu-id="be202-147">Since this is a sample app, you will use the session to store the tokens.</span></span> <span data-ttu-id="be202-148">実際のアプリでは、データベースのような、より信頼性の高い安全なストレージ ソリューションを使用します。</span><span class="sxs-lookup"><span data-stu-id="be202-148">A real-world app would use a more reliable secure storage solution, like a database.</span></span> <span data-ttu-id="be202-149">このセクションでは、次のことを行います。</span><span class="sxs-lookup"><span data-stu-id="be202-149">In this section, you will:</span></span>

- <span data-ttu-id="be202-150">トークン ストア クラスを実装して、MSAL トークン キャッシュとユーザーの詳細情報をシリアル化し、ユーザー セッションに格納します。</span><span class="sxs-lookup"><span data-stu-id="be202-150">Implement a token store class to serialize and store the MSAL token cache and the user's details in the user session.</span></span>
- <span data-ttu-id="be202-151">認証コードを更新して、トークン ストア クラスを使用します。</span><span class="sxs-lookup"><span data-stu-id="be202-151">Update the authentication code to use the token store class.</span></span>
- <span data-ttu-id="be202-152">ベース コントローラー クラスを更新して、格納されているユーザーの詳細情報をアプリケーション内のすべてのビューに公開します。</span><span class="sxs-lookup"><span data-stu-id="be202-152">Update the base controller class to expose the stored user details to all views in the application.</span></span>

1. <span data-ttu-id="be202-153">ソリューション エクスプローラーで **graph-tutorial** フォルダーを右クリックし、**[追加] > [新しいフォルダー]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="be202-153">Right-click the **graph-tutorial** folder in Solution Explorer, and select **Add > New Folder**.</span></span> <span data-ttu-id="be202-154">フォルダーに `TokenStorage` という名前を指定します。</span><span class="sxs-lookup"><span data-stu-id="be202-154">Name the folder `TokenStorage`.</span></span>

1. <span data-ttu-id="be202-155">この新しいフォルダーを右クリックし、[ **クラスの追加>選択します**。ファイルに名前を付 `SessionTokenStore.cs` け、[追加] を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="be202-155">Right click this new folder and select **Add > Class...**. Name the file `SessionTokenStore.cs` and select **Add**.</span></span> <span data-ttu-id="be202-156">このファイルの内容を次のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="be202-156">Replace the contents of this file with the following code.</span></span>

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT license.

    using Microsoft.Identity.Client;
    using Newtonsoft.Json;
    using System.Security.Claims;
    using System.Threading;
    using System.Web;

    namespace graph_tutorial.TokenStorage
    {
        // Simple class to serialize into the session
        public class CachedUser
        {
            public string DisplayName { get; set; }
            public string Email { get; set; }
            public string Avatar { get; set; }
        }

        public class SessionTokenStore
        {
            private static readonly ReaderWriterLockSlim sessionLock = new ReaderWriterLockSlim(LockRecursionPolicy.NoRecursion);

            private HttpContext httpContext = null;
            private string tokenCacheKey = string.Empty;
            private string userCacheKey = string.Empty;

            public SessionTokenStore(ITokenCache tokenCache, HttpContext context, ClaimsPrincipal user)
            {
                httpContext = context;

                if (tokenCache != null)
                {
                    tokenCache.SetBeforeAccess(BeforeAccessNotification);
                    tokenCache.SetAfterAccess(AfterAccessNotification);
                }

                var userId = GetUsersUniqueId(user);
                tokenCacheKey = $"{userId}_TokenCache";
                userCacheKey = $"{userId}_UserCache";
            }

            public bool HasData()
            {
                return (httpContext.Session[tokenCacheKey] != null &&
                    ((byte[])httpContext.Session[tokenCacheKey]).Length > 0);
            }

            public void Clear()
            {
                sessionLock.EnterWriteLock();

                try
                {
                    httpContext.Session.Remove(tokenCacheKey);
                }
                finally
                {
                    sessionLock.ExitWriteLock();
                }
            }

            private void BeforeAccessNotification(TokenCacheNotificationArgs args)
            {
                sessionLock.EnterReadLock();

                try
                {
                    // Load the cache from the session
                    args.TokenCache.DeserializeMsalV3((byte[])httpContext.Session[tokenCacheKey]);
                }
                finally
                {
                    sessionLock.ExitReadLock();
                }
            }

            private void AfterAccessNotification(TokenCacheNotificationArgs args)
            {
                if (args.HasStateChanged)
                {
                    sessionLock.EnterWriteLock();

                    try
                    {
                        // Store the serialized cache in the session
                        httpContext.Session[tokenCacheKey] = args.TokenCache.SerializeMsalV3();
                    }
                    finally
                    {
                        sessionLock.ExitWriteLock();
                    }
                }
            }

            public void SaveUserDetails(CachedUser user)
            {

                sessionLock.EnterWriteLock();
                httpContext.Session[userCacheKey] = JsonConvert.SerializeObject(user);
                sessionLock.ExitWriteLock();
            }

            public CachedUser GetUserDetails()
            {
                sessionLock.EnterReadLock();
                var cachedUser = JsonConvert.DeserializeObject<CachedUser>((string)httpContext.Session[userCacheKey]);
                sessionLock.ExitReadLock();
                return cachedUser;
            }

            private string GetUsersUniqueId(ClaimsPrincipal user)
            {
                // Combine the user's object ID with their tenant ID

                if (user != null)
                {
                    var userObjectId = user.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value ??
                        user.FindFirst("oid").Value;

                    var userTenantId = user.FindFirst("http://schemas.microsoft.com/identity/claims/tenantid").Value ??
                        user.FindFirst("tid").Value;

                    if (!string.IsNullOrEmpty(userObjectId) && !string.IsNullOrEmpty(userTenantId))
                    {
                        return $"{userObjectId}.{userTenantId}";
                    }
                }

                return null;
            }
        }
    }
    ```

1. <span data-ttu-id="be202-157">次のステートメント `using` をファイルの一番上に追加 `App_Start/Startup.Auth.cs` します。</span><span class="sxs-lookup"><span data-stu-id="be202-157">Add the following `using` statements to the top of the `App_Start/Startup.Auth.cs` file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    ```

1. <span data-ttu-id="be202-158">既存の `OnAuthorizationCodeReceivedAsync` 関数を、以下の関数で置き換えます。</span><span class="sxs-lookup"><span data-stu-id="be202-158">Replace the existing `OnAuthorizationCodeReceivedAsync` function with the following.</span></span>

    ```cs
    private async Task OnAuthorizationCodeReceivedAsync(AuthorizationCodeReceivedNotification notification)
    {
        var idClient = ConfidentialClientApplicationBuilder.Create(appId)
            .WithRedirectUri(redirectUri)
            .WithClientSecret(appSecret)
            .Build();

        var signedInUser = new ClaimsPrincipal(notification.AuthenticationTicket.Identity);
        var tokenStore = new SessionTokenStore(idClient.UserTokenCache, HttpContext.Current, signedInUser);

        try
        {
            string[] scopes = graphScopes.Split(' ');

            var result = await idClient.AcquireTokenByAuthorizationCode(
                scopes, notification.Code).ExecuteAsync();

            var userDetails = await GraphHelper.GetUserDetailsAsync(result.AccessToken);

            var cachedUser = new CachedUser()
            {
                DisplayName = userDetails.DisplayName,
                Email = string.IsNullOrEmpty(userDetails.Mail) ?
                userDetails.UserPrincipalName : userDetails.Mail,
                Avatar = string.Empty
            };

            tokenStore.SaveUserDetails(cachedUser);
        }
        catch (MsalException ex)
        {
            string message = "AcquireTokenByAuthorizationCodeAsync threw an exception";
            notification.HandleResponse();
            notification.Response.Redirect($"/Home/Error?message={message}&debug={ex.Message}");
        }
        catch (Microsoft.Graph.ServiceException ex)
        {
            string message = "GetUserDetailsAsync threw an exception";
            notification.HandleResponse();
            notification.Response.Redirect($"/Home/Error?message={message}&debug={ex.Message}");
        }
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="be202-159">この `OnAuthorizationCodeReceivedAsync` の新しいバージョンでは、次のように変更されます。</span><span class="sxs-lookup"><span data-stu-id="be202-159">The changes in this new version of `OnAuthorizationCodeReceivedAsync` do the following:</span></span>
    >
    > - <span data-ttu-id="be202-160">このコードでは、`ConfidentialClientApplication` の既定のユーザー トークン キャッシュが `SessionTokenStore` クラスでラップされるようになります。</span><span class="sxs-lookup"><span data-stu-id="be202-160">The code now wraps the `ConfidentialClientApplication`'s default user token cache with the `SessionTokenStore` class.</span></span> <span data-ttu-id="be202-161">MSAL ライブラリでは、トークンを格納し、必要に応じて更新するロジックを処理します。</span><span class="sxs-lookup"><span data-stu-id="be202-161">The MSAL library will handle the logic of storing the tokens and refreshing it when needed.</span></span>
    > - <span data-ttu-id="be202-162">このコードでは、Microsoft Graph から取得したユーザーの詳細情報を `SessionTokenStore` オブジェクトに渡して、セッションに格納するようになります。</span><span class="sxs-lookup"><span data-stu-id="be202-162">The code now passes the user details obtained from Microsoft Graph to the `SessionTokenStore` object to store in the session.</span></span>
    > - <span data-ttu-id="be202-163">成功すると、コードはリダイレクトされなくなり、返されるだけです。</span><span class="sxs-lookup"><span data-stu-id="be202-163">On success, the code no longer redirects, it just returns.</span></span> <span data-ttu-id="be202-164">これにより、OWIN ミドルウェアは認証プロセスを完了できます。</span><span class="sxs-lookup"><span data-stu-id="be202-164">This allows the OWIN middleware to complete the authentication process.</span></span>

1. <span data-ttu-id="be202-165">サインアウト `SignOut` する前にトークン ストアをクリアするアクションを更新します。次のステートメント `using` を一番上に追加します `Controllers/AccountController.cs` 。</span><span class="sxs-lookup"><span data-stu-id="be202-165">Update the `SignOut` action to clear the token store before signing out. Add the following `using` statement to the top of `Controllers/AccountController.cs`.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    ```

1. <span data-ttu-id="be202-166">既存の `SignOut` 関数を、以下の関数で置き換えます。</span><span class="sxs-lookup"><span data-stu-id="be202-166">Replace the existing `SignOut` function with the following.</span></span>

    ```cs
    public ActionResult SignOut()
    {
        if (Request.IsAuthenticated)
        {
            var tokenStore = new SessionTokenStore(null,
                System.Web.HttpContext.Current, ClaimsPrincipal.Current);

            tokenStore.Clear();

            Request.GetOwinContext().Authentication.SignOut(
                CookieAuthenticationDefaults.AuthenticationType);
        }

        return RedirectToAction("Index", "Home");
    }
    ```

1. <span data-ttu-id="be202-167">次 `Controllers/BaseController.cs` のステートメントを `using` 開き、ファイルの一番上に追加します。</span><span class="sxs-lookup"><span data-stu-id="be202-167">Open `Controllers/BaseController.cs` and add the following `using` statements to the top of the file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    using System.Web;
    using Microsoft.Owin.Security.Cookies;
    ```

1. <span data-ttu-id="be202-168">次の関数を追加します。</span><span class="sxs-lookup"><span data-stu-id="be202-168">Add the following function.</span></span>

    ```cs
    protected override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        if (Request.IsAuthenticated)
        {
            // Get the user's token cache
            var tokenStore = new SessionTokenStore(null,
                System.Web.HttpContext.Current, ClaimsPrincipal.Current);

            if (tokenStore.HasData())
            {
                // Add the user to the view bag
                ViewBag.User = tokenStore.GetUserDetails();
            }
            else
            {
                // The session has lost data. This happens often
                // when debugging. Log out so the user can log back in
                Request.GetOwinContext().Authentication.SignOut(CookieAuthenticationDefaults.AuthenticationType);
                filterContext.Result = RedirectToAction("Index", "Home");
            }
        }

        base.OnActionExecuting(filterContext);
    }
    ```

1. <span data-ttu-id="be202-169">サーバーを起動し、サインイン プロセスを実行します。</span><span class="sxs-lookup"><span data-stu-id="be202-169">Start the server and go through the sign-in process.</span></span> <span data-ttu-id="be202-170">ホーム ページに戻る必要がありますが、サインイン中を示すために UI が変更される必要があります。</span><span class="sxs-lookup"><span data-stu-id="be202-170">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![サインイン後のホーム ページのスクリーンショット](./images/add-aad-auth-01.png)

1. <span data-ttu-id="be202-172">右上隅にあるユーザー アバターをクリックして、[サインアウト **] リンクにアクセス** します。</span><span class="sxs-lookup"><span data-stu-id="be202-172">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="be202-173">**[サインアウト]** をクリックすると、セッションがリセットされ、ホーム ページに戻ります。</span><span class="sxs-lookup"><span data-stu-id="be202-173">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![[サインアウト] リンクのドロップダウン メニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="be202-175">トークンの更新</span><span class="sxs-lookup"><span data-stu-id="be202-175">Refreshing tokens</span></span>

<span data-ttu-id="be202-176">この時点で、アプリケーションはアクセス トークンを持ち、API 呼び出しのヘッダー `Authorization` で送信されます。</span><span class="sxs-lookup"><span data-stu-id="be202-176">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="be202-177">これは、アプリが Microsoft Graph にユーザーの代わりにアクセスできるようにするトークンです。</span><span class="sxs-lookup"><span data-stu-id="be202-177">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="be202-178">ただし、このトークンは一時的なものです。</span><span class="sxs-lookup"><span data-stu-id="be202-178">However, this token is short-lived.</span></span> <span data-ttu-id="be202-179">トークンは発行後 1 時間で期限切れになります。</span><span class="sxs-lookup"><span data-stu-id="be202-179">The token expires an hour after it is issued.</span></span> <span data-ttu-id="be202-180">ここで、更新トークンが役に立ちます。</span><span class="sxs-lookup"><span data-stu-id="be202-180">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="be202-181">更新トークンを使用すると、ユーザーが再度サインインしなくても、アプリは新しいアクセス トークンを要求できます。</span><span class="sxs-lookup"><span data-stu-id="be202-181">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="be202-182">アプリは MSAL ライブラリを使用し、オブジェクトをシリアル化するために、トークン更新ロジックを `TokenCache` 実装する必要があります。</span><span class="sxs-lookup"><span data-stu-id="be202-182">Because the app is using the MSAL library and serializing the `TokenCache` object, you do not have to implement any token refresh logic.</span></span> <span data-ttu-id="be202-183">`ConfidentialClientApplication.AcquireTokenSilentAsync` メソッドが、すべてのロジックを実行します。</span><span class="sxs-lookup"><span data-stu-id="be202-183">The `ConfidentialClientApplication.AcquireTokenSilentAsync` method does all of the logic for you.</span></span> <span data-ttu-id="be202-184">最初にキャッシュされたトークンをチェックし、有効期限が切れていない場合はトークンを返します。</span><span class="sxs-lookup"><span data-stu-id="be202-184">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="be202-185">有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しい更新トークンを取得します。</span><span class="sxs-lookup"><span data-stu-id="be202-185">If it is expired, it uses the cached refresh token to obtain a new one.</span></span> <span data-ttu-id="be202-186">このメソッドは次のモジュールで使用します。</span><span class="sxs-lookup"><span data-stu-id="be202-186">You'll use this method in the following module.</span></span>
