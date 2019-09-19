<!-- markdownlint-disable MD002 MD041 -->

この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。 これは、Microsoft Graph API を呼び出すために必要な OAuth アクセストークンを取得するために必要です。 この手順では、アプリケーションに OWIN ミドルウェアと[Microsoft Authentication library](https://www.nuget.org/packages/Microsoft.Identity.Client/)ライブラリを統合します。

1. ソリューションエクスプローラーで**グラフ-チュートリアル**プロジェクトを右クリックし、[**新しいアイテムの追加 >**] を選択します。[ **Web 構成ファイル**] を選択し`PrivateSettings.config` 、ファイルに名前を指定して、[**追加**] を選択します。 すべてのコンテンツを次のコードに置き換えます。

    ```xml
    <appSettings>
        <add key="ida:AppID" value="YOUR APP ID" />
        <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
        <add key="ida:RedirectUri" value="https://localhost:PORT/" />
        <add key="ida:AppScopes" value="User.Read Calendars.Read" />
    </appSettings>
    ```

    を`YOUR_APP_ID_HERE`アプリケーション登録ポータルからのアプリケーション ID に置き換え、生成し`YOUR_APP_PASSWORD_HERE`たクライアントシークレットに置き換えます。 クライアントシークレットにアンパサンド (`&`) が含まれている場合は、をに`&amp;`置き換え`PrivateSettings.config`てください。 また、 `PORT` `ida:RedirectUri`の値をアプリケーションの URL と一致するように変更してください。

    > [!IMPORTANT]
    > Git などのソース管理を使用している場合は、アプリ ID とパスワードを誤っ`PrivateSettings.config`てリークしないように、ソース管理からファイルを除外することをお勧めします。

1. この`Web.config`新しいファイルを読み込むように更新します。 `<appSettings>` (行 7) を次のように置き換えます。

    ```xml
    <appSettings file="PrivateSettings.config">
    ```

## <a name="implement-sign-in"></a>サインインの実装

まず、OWIN ミドルウェアを初期化して、アプリの Azure AD 認証を使用します。

1. ソリューションエクスプローラーで**App_Start**フォルダーを右クリックし、[ **> クラスの追加**] を選択します。ファイル`Startup.Auth.cs`の名前を指定して、[**追加**] を選択します。 すべての内容を次のコードに置き換えます。

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
    > このコードでは、の`PrivateSettings.config`値を使用して OWIN ミドルウェアを構成し`OnAuthenticationFailedAsync` 、 `OnAuthorizationCodeReceivedAsync`2 つのコールバックメソッドとを定義します。 このコールバックメソッドは、サインインプロセスが Azure から戻るときに呼び出されます。

1. ここで、 `Startup.cs`ファイルを更新し`ConfigureAuth`てメソッドを呼び出します。 の`Startup.cs`内容全体を次のコードに置き換えます。

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

1. `HomeController`クラス`message`に`Error`アクションを追加して、パラメーターと`debug`クエリパラメーターを`Alert`オブジェクトに変換します。 を`Controllers/HomeController.cs`開き、次の関数を追加します。

    ```cs
    public ActionResult Error(string message, string debug)
    {
        Flash(message, debug);
        return RedirectToAction("Index");
    }
    ```

1. サインインを処理するコントローラーを追加します。 ソリューションエクスプローラーで [**コントローラー** ] フォルダーを右クリックし、[ **> コントローラーの追加**] を選択します。[ **MVC 5 コントローラー-空**] を選択し、[**追加**] を選択します。 コントローラー `AccountController`の名前を指定して、[**追加**] を選択します。 ファイルの内容全体を次のコードに置き換えます。

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

    これは、 `SignIn` and `SignOut`アクションを定義します。 `SignIn`アクションは、要求が既に認証されているかどうかを確認します。 そうでない場合は、OWIN ミドルウェアを呼び出してユーザーを認証します。 アクション`SignOut`は、OWIN ミドルウェアを呼び出してサインアウトします。

1. 変更を保存し、プロジェクトを開始します。 [サインイン] ボタンをクリックすると、に`https://login.microsoftonline.com`リダイレクトされます。 Microsoft アカウントを使用してログインし、要求されたアクセス許可に同意します。 ブラウザーがアプリにリダイレクトし、トークンが表示されます。

### <a name="get-user-details"></a>ユーザーの詳細を取得する

ユーザーがログインすると、Microsoft Graph から情報を取得できます。

1. ソリューションエクスプローラーで [**グラフ-チュートリアル**] フォルダーを右クリックし、[ **> 新しいフォルダーの追加**] を選択します。 フォルダー `Helpers`に名前を指定します。

1. この新しいフォルダーを右クリックして [ **> クラスの追加**] を選択します。ファイル`GraphHelper.cs`の名前を指定して、[**追加**] を選択します。 このファイルの内容を次のコードに置き換えます。

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
                        async (requestMessage) =>
                        {
                            requestMessage.Headers.Authorization =
                                new AuthenticationHeaderValue("Bearer", accessToken);
                        }));

                return await graphClient.Me.Request().GetAsync();
            }
        }
    }
    ```

    これにより`GetUserDetails` 、関数が実装されます。これは、 `/me` Microsoft Graph SDK を使用してエンドポイントを呼び出し、結果を返します。

1. この関数`OnAuthorizationCodeReceivedAsync`を呼び出す`App_Start/Startup.Auth.cs`には、のメソッドを更新します。 次`using`のステートメントをファイルの先頭に追加します。

    ```cs
    using graph_tutorial.Helpers;
    ```

1. 既存`try`のブロックを次`OnAuthorizationCodeReceivedAsync`のコードで置き換えます。

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

1. 変更を保存し、アプリを開始します。サインイン後、アクセストークンではなく、ユーザーの名前と電子メールアドレスが表示されます。

## <a name="storing-the-tokens"></a>トークンの格納

トークンを入手できるようになったので、これをアプリに保存する方法を実装します。 これはサンプルアプリであるため、セッションを使用してトークンを格納します。 実際のアプリケーションでは、データベースのような、より信頼性の高いセキュリティで保護されたストレージソリューションを使用します。 このセクションでは、次の操作を行います。

- トークンストアクラスを実装して、MSAL トークンキャッシュとユーザーの詳細情報をシリアル化し、ユーザーセッションに格納します。
- トークンストアクラスを使用するように認証コードを更新します。
- ベースコントローラクラスを更新して、保存されたユーザーの詳細をアプリケーションのすべてのビューに公開します。

1. ソリューションエクスプローラーで [**グラフ-チュートリアル**] フォルダーを右クリックし、[ **> 新しいフォルダーの追加**] を選択します。 フォルダー `TokenStorage`に名前を指定します。

1. この新しいフォルダーを右クリックして [ **> クラスの追加**] を選択します。ファイル`SessionTokenStore.cs`の名前を指定して、[**追加**] を選択します。 このファイルの内容を次のコードに置き換えます。

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

1. 次`using`のステートメントを`App_Start/Startup.Auth.cs`ファイルの先頭に追加します。

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    ```

1. 既存の `OnAuthorizationCodeReceivedAsync` 関数を、以下の関数で置換します。

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
    > この新しいバージョンでの`OnAuthorizationCodeReceivedAsync`変更点は、次のとおりです。
    >
    > - これで、コードは`ConfidentialClientApplication`、既定のユーザートークンキャッシュを`SessionTokenStore`クラスでラップするようになります。 MSAL ライブラリは、トークンを格納し、必要に応じて更新するロジックを処理します。
    > - このコードでは、Microsoft Graph から取得したユーザーの`SessionTokenStore`詳細を、セッションに格納するオブジェクトに渡します。
    > - 成功した場合、コードはリダイレクトされず、ただ返します。 これにより、OWIN ミドルウェアは認証プロセスを完了することができます。

1. サインアウトする`SignOut`前に、トークンストアをクリアするアクションを更新します。の`Controllers/AccountController.cs`先頭に`using`次のステートメントを追加します。

    ```cs
    using graph_tutorial.TokenStorage;
    ```

1. 既存の `SignOut` 関数を、以下の関数で置換します。

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

1. を`Controllers/BaseController.cs`開き、次`using`のステートメントをファイルの先頭に追加します。

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

1. サーバーを起動し、サインインプロセスを実行します。 ホームページに戻る必要がありますが、UI は、サインインしていることを示すように変更する必要があります。

    ![サインイン後のホームページのスクリーンショット](./images/add-aad-auth-01.png)

1. 右上隅にあるユーザーアバターをクリックして、[**サインアウト**] リンクにアクセスします。 [**サインアウト**] をクリックすると、セッションがリセットされ、ホームページに戻ります。

    ![[サインアウト] リンクがあるドロップダウンメニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>トークンの更新

この時点で、アプリケーションには、API 呼び出しの`Authorization`ヘッダーで送信されるアクセストークンがあります。 これは、アプリがユーザーに代わって Microsoft Graph にアクセスできるようにするトークンです。

ただし、このトークンは存続期間が短くなります。 トークンが発行された後、有効期限が切れる時間になります。 ここでは、更新トークンが有効になります。 更新トークンを使用すると、ユーザーが再度サインインする必要なく、新しいアクセストークンをアプリで要求できます。

アプリは MSAL ライブラリを使用して`TokenCache`オブジェクトをシリアル化しているため、トークン更新ロジックを実装する必要はありません。 メソッド`ConfidentialClientApplication.AcquireTokenSilentAsync`は、すべてのロジックを実行します。 最初に、キャッシュされたトークンをチェックし、有効期限が切れていない場合はそれを返します。 有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しいものを取得します。 このメソッドは、次のモジュールで使用します。
