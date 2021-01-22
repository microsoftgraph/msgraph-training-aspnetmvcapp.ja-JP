<!-- markdownlint-disable MD002 MD041 -->

この演習では、前の演習のアプリケーションを拡張して、Azure AD での認証をサポートします。 これは、Microsoft Graph API を呼び出すのに必要な OAuth アクセス トークンを取得するために必要です。 この手順では、OWIN ミドルウェアと Microsoft [認証](https://www.nuget.org/packages/Microsoft.Identity.Client/) ライブラリ ライブラリをアプリケーションに統合します。

1. ソリューション エクスプローラーで **グラフ チュートリアル プロジェクトを右クリック** し、[新しいアイテムの> **を選択します**。[Web **構成ファイル] を選択し**、ファイルに名前を付 `PrivateSettings.config` け、[追加] を **選択します**。 すべてのコンテンツを次のコードに置き換えます。

    ```xml
    <appSettings>
        <add key="ida:AppID" value="YOUR APP ID" />
        <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
        <add key="ida:RedirectUri" value="https://localhost:PORT/" />
        <add key="ida:AppScopes" value="User.Read Calendars.Read" />
    </appSettings>
    ```

    アプリケーション `YOUR_APP_ID_HERE` 登録ポータルのアプリケーション ID に置き換え、生成したクライアント シークレット `YOUR_APP_PASSWORD_HERE` に置き換える。 クライアント シークレットにアンパサンド (`&`) が含まれている場合は、`PrivateSettings.config` で `&amp;` に置き換えてください。 また、`ida:RedirectUri` の `PORT` 値をアプリケーションの URL に合わせて変更してください。

    > [!IMPORTANT]
    > git などのソース管理を使用している場合は、アプリ ID とパスワードが誤って漏洩しないように、ファイルをソース管理から除外する良い時期です `PrivateSettings.config` 。

1. この `Web.config` 新しいファイルを読み込む更新。 `<appSettings>` (7 行目) を以下のように置き換えます

    ```xml
    <appSettings file="PrivateSettings.config">
    ```

## <a name="implement-sign-in"></a>サインインの実装

まず OWIN ミドルウェアを初期化して、アプリに Azure AD 認証を使用します。

1. ソリューション エクスプローラーで **App_Start** フォルダーを右クリックし、[クラスの追加 **>選択します**。ファイルに名前を付 `Startup.Auth.cs` け、[追加] を **選択します**。 すべての内容を、次のコードで置き換えます。

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
    > このコードは、OWIN ミドルウェアの値を使用して構成し、2 つのコールバック `PrivateSettings.config` メソッドを定義 `OnAuthenticationFailedAsync` します `OnAuthorizationCodeReceivedAsync` 。 これらのコールバック メソッドは、サインイン プロセスが Azure から戻るときに呼び出されます。

1. 次に、メソッド `Startup.cs` を呼び出すファイルを更新 `ConfigureAuth` します。 内容全体を次の `Startup.cs` コードに置き換えます。

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

1. `Error` アクションを `HomeController` クラスに追加し、`message` クエリ パラメーターおよび `debug` クエリ パラメーターを `Alert` オブジェクトに変換します。 次 `Controllers/HomeController.cs` の関数を開いて追加します。

    ```cs
    public ActionResult Error(string message, string debug)
    {
        Flash(message, debug);
        return RedirectToAction("Index");
    }
    ```

1. コントローラーを追加してサインインを処理します。 ソリューション エクスプローラーで **[コントローラー]** フォルダーを右クリックし、**[追加] > [コントローラー...]** の順に選択します。**[MVC 5 コントローラー - 空]** を選び、**[追加]** を選択します。 コントローラーの名前を `AccountController` に指定し、**[追加]** を選択します。 ファイルのすべての内容を次のコードで置き換えます。

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

    これにより、`SignIn` アクションと `SignOut` アクションが定義されます。 `SignIn` アクションは、要求が既に認証されているかどうかを確認します。 認証されていなければ、OWIN ミドルウェアを呼び出してユーザーを認証します。 `SignOut` アクションは、サインアウトする OWIN ミドルウェアを呼び出します。

1. 変更を保存してプロジェクトを開始します。 [サインイン] ボタンをクリックすると、`https://login.microsoftonline.com` にリダイレクトされます。 Microsoft アカウントでログインし、要求されたアクセス許可に同意します。 ブラウザーがアプリにリダイレクトし、トークンが表示されます。

### <a name="get-user-details"></a>ユーザーの詳細情報を取得する

ユーザーがログインすると、Microsoft Graph からそのユーザーの情報を入手できます。

1. ソリューション エクスプローラーで **graph-tutorial** フォルダーを右クリックし、**[追加] > [新しいフォルダー]** を選択します。 フォルダーに `Helpers` という名前を指定します。

1. この新しいフォルダーを右クリックし、[ **クラスの追加>選択します**。ファイルに名前を付 `GraphHelper.cs` け、[追加] を **選択します**。 このファイルの内容を次のコードで置き換えます。

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

    これにより、`GetUserDetails` 関数が実装されます。これは、Microsoft Graph SDK を使用して `/me` エンドポイントを呼び出して結果を返します。

1. この関数 `OnAuthorizationCodeReceivedAsync` を呼び出 `App_Start/Startup.Auth.cs` すメソッドを更新します。 次の `using` ステートメントをファイルの一番上に追加します。

    ```cs
    using graph_tutorial.Helpers;
    ```

1. `OnAuthorizationCodeReceivedAsync` の既存の `try` ブロックを、以下のコードで置き換えます。

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

1. 変更を保存してアプリを起動すると、サインイン後、アクセス トークンの代わりにユーザーの名前とメール アドレスが表示されます。

## <a name="storing-the-tokens"></a>トークンの格納

トークンを取得できるようになったので、トークンをアプリに格納する手順を実装します。 これはサンプル アプリです。トークンの保存にはセッションを使用します。 実際のアプリでは、データベースのような、より信頼性の高い安全なストレージ ソリューションを使用します。 このセクションでは、次のことを行います。

- トークン ストア クラスを実装して、MSAL トークン キャッシュとユーザーの詳細情報をシリアル化し、ユーザー セッションに格納します。
- 認証コードを更新して、トークン ストア クラスを使用します。
- ベース コントローラー クラスを更新して、格納されているユーザーの詳細情報をアプリケーション内のすべてのビューに公開します。

1. ソリューション エクスプローラーで **graph-tutorial** フォルダーを右クリックし、**[追加] > [新しいフォルダー]** を選択します。 フォルダーに `TokenStorage` という名前を指定します。

1. この新しいフォルダーを右クリックし、[ **クラスの追加>選択します**。ファイルに名前を付 `SessionTokenStore.cs` け、[追加] を **選択します**。 このファイルの内容を次のコードで置き換えます。

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

1. 次のステートメント `using` をファイルの一番上に追加 `App_Start/Startup.Auth.cs` します。

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    ```

1. 既存の `OnAuthorizationCodeReceivedAsync` 関数を、以下の関数で置き換えます。

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
    > この `OnAuthorizationCodeReceivedAsync` の新しいバージョンでは、次のように変更されます。
    >
    > - このコードでは、`ConfidentialClientApplication` の既定のユーザー トークン キャッシュが `SessionTokenStore` クラスでラップされるようになります。 MSAL ライブラリでは、トークンを格納し、必要に応じて更新するロジックを処理します。
    > - このコードでは、Microsoft Graph から取得したユーザーの詳細情報を `SessionTokenStore` オブジェクトに渡して、セッションに格納するようになります。
    > - 成功すると、コードはリダイレクトされなくなり、返されるだけです。 これにより、OWIN ミドルウェアは認証プロセスを完了できます。

1. サインアウト `SignOut` する前にトークン ストアをクリアするアクションを更新します。次のステートメント `using` を一番上に追加します `Controllers/AccountController.cs` 。

    ```cs
    using graph_tutorial.TokenStorage;
    ```

1. 既存の `SignOut` 関数を、以下の関数で置き換えます。

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

1. 次 `Controllers/BaseController.cs` のステートメントを `using` 開き、ファイルの一番上に追加します。

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    using System.Web;
    using Microsoft.Owin.Security.Cookies;
    ```

1. 次の関数を追加します。

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

1. サーバーを起動し、サインイン プロセスを実行します。 ホーム ページに戻る必要がありますが、サインイン中を示すために UI が変更される必要があります。

    ![サインイン後のホーム ページのスクリーンショット](./images/add-aad-auth-01.png)

1. 右上隅にあるユーザー アバターをクリックして、[サインアウト **] リンクにアクセス** します。 **[サインアウト]** をクリックすると、セッションがリセットされ、ホーム ページに戻ります。

    ![[サインアウト] リンクのドロップダウン メニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>トークンの更新

この時点で、アプリケーションはアクセス トークンを持ち、API 呼び出しのヘッダー `Authorization` で送信されます。 これは、アプリが Microsoft Graph にユーザーの代わりにアクセスできるようにするトークンです。

ただし、このトークンは一時的なものです。 トークンは発行後 1 時間で期限切れになります。 ここで、更新トークンが役に立ちます。 更新トークンを使用すると、ユーザーが再度サインインしなくても、アプリは新しいアクセス トークンを要求できます。

アプリは MSAL ライブラリを使用し、オブジェクトをシリアル化するために、トークン更新ロジックを `TokenCache` 実装する必要があります。 `ConfidentialClientApplication.AcquireTokenSilentAsync` メソッドが、すべてのロジックを実行します。 最初にキャッシュされたトークンをチェックし、有効期限が切れていない場合はトークンを返します。 有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しい更新トークンを取得します。 このメソッドは次のモジュールで使用します。
