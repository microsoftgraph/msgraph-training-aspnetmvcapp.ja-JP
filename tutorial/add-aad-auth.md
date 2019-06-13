<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="63f32-101">この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。</span><span class="sxs-lookup"><span data-stu-id="63f32-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="63f32-102">これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="63f32-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="63f32-103">この手順では、アプリケーションに OWIN ミドルウェアと[Microsoft Authentication library](https://www.nuget.org/packages/Microsoft.Identity.Client/)ライブラリを統合します。</span><span class="sxs-lookup"><span data-stu-id="63f32-103">In this step you will integrate the OWIN middleware and the [Microsoft Authentication Library](https://www.nuget.org/packages/Microsoft.Identity.Client/) library into the application.</span></span>

<span data-ttu-id="63f32-104">ソリューションエクスプローラーで**グラフのチュートリアル**プロジェクトを右クリックし、[**新しいアイテムの追加 >**] を選択します。[ **Web 構成ファイル**] を選択し`PrivateSettings.config` 、ファイルに名前を指定して、[**追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="63f32-104">Right-click the **graph-tutorial** project in Solution Explorer and choose **Add > New Item...**. Choose **Web Configuration File**, name the file `PrivateSettings.config` and choose **Add**.</span></span> <span data-ttu-id="63f32-105">すべてのコンテンツを次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="63f32-105">Replace its entire contents with the following code.</span></span>

```xml
<appSettings>
    <add key="ida:AppID" value="YOUR APP ID" />
    <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
    <add key="ida:RedirectUri" value="http://localhost:PORT/" />
    <add key="ida:AppScopes" value="User.Read Calendars.Read" />
</appSettings>
```

<span data-ttu-id="63f32-106">を`YOUR_APP_ID_HERE`アプリケーション登録ポータルからのアプリケーション ID に置き換え、生成し`YOUR_APP_PASSWORD_HERE`たクライアントシークレットに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="63f32-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the client secret you generated.</span></span> <span data-ttu-id="63f32-107">クライアントシークレットにアンパサンド (`&`) が含まれている場合は、をに`&amp;`置き換え`PrivateSettings.config`てください。</span><span class="sxs-lookup"><span data-stu-id="63f32-107">If your client secret contains any ampersands (`&`), be sure to replace them with `&amp;` in `PrivateSettings.config`.</span></span> <span data-ttu-id="63f32-108">また、 `PORT` `ida:RedirectUri`の値をアプリケーションの URL と一致するように変更してください。</span><span class="sxs-lookup"><span data-stu-id="63f32-108">Also be sure to modify the `PORT` value for the `ida:RedirectUri` to match your application's URL.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="63f32-109">Git などのソース管理を使用している場合は、アプリ ID とパスワードを誤っ`PrivateSettings.config`てリークしないように、ソース管理からファイルを除外することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="63f32-109">If you're using source control such as git, now would be a good time to exclude the `PrivateSettings.config` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

<span data-ttu-id="63f32-110">この`Web.config`新しいファイルを読み込むように更新します。</span><span class="sxs-lookup"><span data-stu-id="63f32-110">Update `Web.config` to load this new file.</span></span> <span data-ttu-id="63f32-111">`<appSettings>` (行 7) を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="63f32-111">Replace the `<appSettings>` (line 7) with the following</span></span>

```xml
<appSettings file="PrivateSettings.config">
```

## <a name="implement-sign-in"></a><span data-ttu-id="63f32-112">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="63f32-112">Implement sign-in</span></span>

<span data-ttu-id="63f32-113">まず、OWIN ミドルウェアを初期化して、アプリの Azure AD 認証を使用します。</span><span class="sxs-lookup"><span data-stu-id="63f32-113">Start by initializing the OWIN middleware to use Azure AD authentication for the app.</span></span> <span data-ttu-id="63f32-114">ソリューションエクスプローラーで [ **App_Start** ] フォルダーを右クリックし、[ **> クラスの追加**] を選択します。ファイル`Startup.Auth.cs`の名前を指定して、[**追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="63f32-114">Right-click the **App_Start** folder in Solution Explorer and choose **Add > Class...**. Name the file `Startup.Auth.cs` and choose **Add**.</span></span> <span data-ttu-id="63f32-115">すべての内容を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="63f32-115">Replace the entire contents with the following code.</span></span>

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

<span data-ttu-id="63f32-116">このコードでは、の`PrivateSettings.config`値を使用して OWIN ミドルウェアを構成し`OnAuthenticationFailedAsync` 、 `OnAuthorizationCodeReceivedAsync`2 つのコールバックメソッドとを定義します。</span><span class="sxs-lookup"><span data-stu-id="63f32-116">This code configures the OWIN middleware with the values from `PrivateSettings.config` and defines two callback methods, `OnAuthenticationFailedAsync` and `OnAuthorizationCodeReceivedAsync`.</span></span> <span data-ttu-id="63f32-117">このコールバックメソッドは、サインインプロセスが Azure から戻るときに呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="63f32-117">These callback methods will be invoked when the sign-in process returns from Azure.</span></span>

<span data-ttu-id="63f32-118">ここで、 `Startup.cs`ファイルを更新し`ConfigureAuth`てメソッドを呼び出します。</span><span class="sxs-lookup"><span data-stu-id="63f32-118">Now update the `Startup.cs` file to call the `ConfigureAuth` method.</span></span> <span data-ttu-id="63f32-119">の`Startup.cs`内容全体を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="63f32-119">Replace the entire contents of `Startup.cs` with the following code.</span></span>

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

<span data-ttu-id="63f32-120">`HomeController`クラス`message`に`Error`アクションを追加して、パラメーターと`debug`クエリパラメーターを`Alert`オブジェクトに変換します。</span><span class="sxs-lookup"><span data-stu-id="63f32-120">Add an `Error` action to the `HomeController` class to transform the `message` and `debug` query parameters into an `Alert` object.</span></span> <span data-ttu-id="63f32-121">を`Controllers/HomeController.cs`開き、次の関数を追加します。</span><span class="sxs-lookup"><span data-stu-id="63f32-121">Open `Controllers/HomeController.cs` and add the following function.</span></span>

```cs
public ActionResult Error(string message, string debug)
{
    Flash(message, debug);
    return RedirectToAction("Index");
}
```

<span data-ttu-id="63f32-122">サインインを処理するコントローラーを追加します。</span><span class="sxs-lookup"><span data-stu-id="63f32-122">Add a controller to handle sign-in.</span></span> <span data-ttu-id="63f32-123">ソリューションエクスプローラーで [**コントローラー** ] フォルダーを右クリックし、[ **> コントローラーの追加**] を選択します。[ **MVC 5 コントローラー-空**] を選択し、[**追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="63f32-123">Right-click the **Controllers** folder in Solution Explorer and choose **Add > Controller...**. Choose **MVC 5 Controller - Empty** and choose **Add**.</span></span> <span data-ttu-id="63f32-124">コントローラー `AccountController`の名前を指定して、[**追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="63f32-124">Name the controller `AccountController` and choose **Add**.</span></span> <span data-ttu-id="63f32-125">ファイルの内容全体を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="63f32-125">Replace the entire contents of the file with the following code.</span></span>

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

<span data-ttu-id="63f32-126">これは、 `SignIn` and `SignOut`アクションを定義します。</span><span class="sxs-lookup"><span data-stu-id="63f32-126">This defines a `SignIn` and `SignOut` action.</span></span> <span data-ttu-id="63f32-127">`SignIn`アクションは、要求が既に認証されているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="63f32-127">The `SignIn` action checks if the request is already authenticated.</span></span> <span data-ttu-id="63f32-128">そうでない場合は、OWIN ミドルウェアを呼び出してユーザーを認証します。</span><span class="sxs-lookup"><span data-stu-id="63f32-128">If not, it invokes the OWIN middleware to authenticate the user.</span></span> <span data-ttu-id="63f32-129">アクション`SignOut`は、OWIN ミドルウェアを呼び出してサインアウトします。</span><span class="sxs-lookup"><span data-stu-id="63f32-129">The `SignOut` action invokes the OWIN middleware to sign out.</span></span>

<span data-ttu-id="63f32-130">変更を保存し、プロジェクトを開始します。</span><span class="sxs-lookup"><span data-stu-id="63f32-130">Save your changes and start the project.</span></span> <span data-ttu-id="63f32-131">[サインイン] ボタンをクリックすると、に`https://login.microsoftonline.com`リダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="63f32-131">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="63f32-132">Microsoft アカウントを使用してログインし、要求されたアクセス許可に同意します。</span><span class="sxs-lookup"><span data-stu-id="63f32-132">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="63f32-133">ブラウザーがアプリにリダイレクトし、トークンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="63f32-133">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="63f32-134">ユーザーの詳細を取得する</span><span class="sxs-lookup"><span data-stu-id="63f32-134">Get user details</span></span>

<span data-ttu-id="63f32-135">最初に、Microsoft Graph のすべての呼び出しを保持する新しいファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="63f32-135">Start by creating a new file to hold all of your Microsoft Graph calls.</span></span> <span data-ttu-id="63f32-136">ソリューションエクスプローラーで [**グラフ-チュートリアル**] フォルダーを右クリックし、[ **> 新しいフォルダーの追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="63f32-136">Right-click the **graph-tutorial** folder in Solution Explorer, and choose **Add > New Folder**.</span></span> <span data-ttu-id="63f32-137">フォルダー `Helpers`に名前を指定します。</span><span class="sxs-lookup"><span data-stu-id="63f32-137">Name the folder `Helpers`.</span></span> <span data-ttu-id="63f32-138">この新しいフォルダーを右クリックして、[ **> クラスの追加**] を選択します。ファイル`GraphHelper.cs`の名前を指定して、[**追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="63f32-138">Right click this new folder and choose **Add > Class...**. Name the file `GraphHelper.cs` and choose **Add**.</span></span> <span data-ttu-id="63f32-139">このファイルの内容を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="63f32-139">Replace the contents of this file with the following code.</span></span>

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

<span data-ttu-id="63f32-140">これにより`GetUserDetails` 、関数が実装されます。これは、 `/me` Microsoft Graph SDK を使用してエンドポイントを呼び出し、結果を返します。</span><span class="sxs-lookup"><span data-stu-id="63f32-140">This implements the `GetUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

<span data-ttu-id="63f32-141">この関数`OnAuthorizationCodeReceivedAsync`を呼び出す`App_Start/Startup.Auth.cs`には、のメソッドを更新します。</span><span class="sxs-lookup"><span data-stu-id="63f32-141">Update the `OnAuthorizationCodeReceivedAsync` method in `App_Start/Startup.Auth.cs` to call this function.</span></span> <span data-ttu-id="63f32-142">最初に、次`using`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="63f32-142">First, add the following `using` statement to the top of the file.</span></span>

```cs
using graph_tutorial.Helpers;
```

<span data-ttu-id="63f32-143">既存`try`のブロックを次`OnAuthorizationCodeReceivedAsync`のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="63f32-143">Replace the existing `try` block in `OnAuthorizationCodeReceivedAsync` with the following code.</span></span>

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

<span data-ttu-id="63f32-144">変更を保存してアプリを開始すると、サインイン後に、アクセストークンではなく、ユーザーの名前と電子メールアドレスが表示されます。</span><span class="sxs-lookup"><span data-stu-id="63f32-144">Now if you save your changes and start the app, after sign-in you should see the user's name and email address instead of the access token.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="63f32-145">トークンの格納</span><span class="sxs-lookup"><span data-stu-id="63f32-145">Storing the tokens</span></span>

<span data-ttu-id="63f32-146">トークンを入手できるようになったので、これをアプリに保存する方法を実装します。</span><span class="sxs-lookup"><span data-stu-id="63f32-146">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="63f32-147">これはサンプルアプリであるため、セッションを使用してトークンを格納します。</span><span class="sxs-lookup"><span data-stu-id="63f32-147">Since this is a sample app, we'll use the session to store the tokens.</span></span> <span data-ttu-id="63f32-148">実際のアプリケーションでは、データベースのような、より信頼性の高いセキュリティで保護されたストレージソリューションを使用します。</span><span class="sxs-lookup"><span data-stu-id="63f32-148">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="63f32-149">ソリューションエクスプローラーで [**グラフ-チュートリアル**] フォルダーを右クリックし、[ **> 新しいフォルダーの追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="63f32-149">Right-click the **graph-tutorial** folder in Solution Explorer, and choose **Add > New Folder**.</span></span> <span data-ttu-id="63f32-150">フォルダー `TokenStorage`に名前を指定します。</span><span class="sxs-lookup"><span data-stu-id="63f32-150">Name the folder `TokenStorage`.</span></span> <span data-ttu-id="63f32-151">この新しいフォルダーを右クリックして、[ **> クラスの追加**] を選択します。ファイル`SessionTokenStore.cs`の名前を指定して、[**追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="63f32-151">Right click this new folder and choose **Add > Class...**. Name the file `SessionTokenStore.cs` and choose **Add**.</span></span> <span data-ttu-id="63f32-152">このファイルの内容を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="63f32-152">Replace the contents of this file with the following code.</span></span>

```cs
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

    // Adapted from https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-v2
    public class SessionTokenStore
    {
        private static readonly ReaderWriterLockSlim sessionLock = new ReaderWriterLockSlim(LockRecursionPolicy.NoRecursion);

        private readonly string userId = string.Empty;
        private readonly string cacheId = string.Empty;
        private readonly string cachedUserId = string.Empty;
        private HttpContext httpContext = null;
        private ITokenCache tokenCache;

        public SessionTokenStore(string userId, HttpContext httpcontext)
        {
            this.userId = userId;
            this.cacheId = $"{userId}_TokenCache";
            this.cachedUserId = $"{userId}_UserCache";
            this.httpContext = httpcontext;
        }

        public void Initialize(ITokenCache tokenCache)
        {
            this.tokenCache = tokenCache;
            this.tokenCache.SetBeforeAccess(BeforeAccessNotification);
            this.tokenCache.SetAfterAccess(AfterAccessNotification);
            Load();
        }

        public bool HasData()
        {
            return (httpContext.Session[cacheId] != null && ((byte[])httpContext.Session[cacheId]).Length > 0);
        }

        public void Clear()
        {
            httpContext.Session.Remove(cacheId);
        }

        private void Load()
        {
            sessionLock.EnterReadLock();
            tokenCache.DeserializeMsalV3((byte[])httpContext.Session[cacheId]);
            sessionLock.ExitReadLock();
        }

        private void Persist()
        {
            sessionLock.EnterReadLock();
            httpContext.Session[cacheId] = tokenCache.SerializeMsalV3();
            sessionLock.ExitReadLock();
        }

        private void BeforeAccessNotification(TokenCacheNotificationArgs args)
        {
            Load();
        }

        private void AfterAccessNotification(TokenCacheNotificationArgs args)
        {
            if (args.HasStateChanged)
            {
                Persist();
            }
        }

        public void SaveUserDetails(CachedUser user)
        {
            sessionLock.EnterReadLock();
            httpContext.Session[cachedUserId] = JsonConvert.SerializeObject(user);
            sessionLock.ExitReadLock();
        }

        public CachedUser GetUserDetails()
        {
            sessionLock.EnterReadLock();
            var cachedUser = JsonConvert.DeserializeObject<CachedUser>((string)httpContext.Session[cachedUserId]);
            sessionLock.ExitReadLock();
            return cachedUser;
        }
    }
}
```

<span data-ttu-id="63f32-153">このコードでは`SessionTokenStore` 、msal ライブラリの`TokenCache`クラスを処理するクラスを作成します。</span><span class="sxs-lookup"><span data-stu-id="63f32-153">This code creates a `SessionTokenStore` class that works with the MSAL library's `TokenCache` class.</span></span> <span data-ttu-id="63f32-154">ここでのコードのほとんどは、 `TokenCache`をシリアル化し、セッションに逆シリアル化することです。</span><span class="sxs-lookup"><span data-stu-id="63f32-154">Most of the code here involves serializing and deserializing the `TokenCache` to the session.</span></span> <span data-ttu-id="63f32-155">また、ユーザーの詳細をシリアル化し、セッションに逆シリアル化するためのクラスとメソッドも提供します。</span><span class="sxs-lookup"><span data-stu-id="63f32-155">It also provides a class and methods to serialize and deserialize the user's details to the session.</span></span>

<span data-ttu-id="63f32-156">この時点で、次`using`のステートメントを`App_Start/Startup.Auth.cs`ファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="63f32-156">Now, add the following `using` statement to the top of the `App_Start/Startup.Auth.cs` file.</span></span>

```cs
using graph_tutorial.TokenStorage;
using System.IdentityModel.Claims;
```

<span data-ttu-id="63f32-157">次に、 `OnAuthorizationCodeReceivedAsync`関数を更新して、 `SessionTokenStore`クラスのインスタンスを作成し、 `ConfidentialClientApplication`オブジェクトのコンストラクターにそれを提供します。</span><span class="sxs-lookup"><span data-stu-id="63f32-157">Now update the `OnAuthorizationCodeReceivedAsync` function to create an instance of the `SessionTokenStore` class and provide that to the constructor for the `ConfidentialClientApplication` object.</span></span> <span data-ttu-id="63f32-158">これにより、MSAL は、トークンを格納するためにキャッシュ実装を使用します。</span><span class="sxs-lookup"><span data-stu-id="63f32-158">That will cause MSAL to use your cache implementation for storing tokens.</span></span> <span data-ttu-id="63f32-159">既存の `OnAuthorizationCodeReceivedAsync` 関数を、以下の関数で置換します。</span><span class="sxs-lookup"><span data-stu-id="63f32-159">Replace the existing `OnAuthorizationCodeReceivedAsync` function with the following.</span></span>

```cs
private async Task OnAuthorizationCodeReceivedAsync(AuthorizationCodeReceivedNotification notification)
{
    var idClient = ConfidentialClientApplicationBuilder.Create(appId)
        .WithRedirectUri(redirectUri)
        .WithClientSecret(appSecret)
        .Build();

    var signedInUserId = notification.AuthenticationTicket.Identity.FindFirst(ClaimTypes.NameIdentifier).Value;
    var tokenStore = new SessionTokenStore(signedInUserId, HttpContext.Current);
    tokenStore.Initialize(idClient.UserTokenCache);

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

<span data-ttu-id="63f32-160">変更内容を要約するには、次のようにします。</span><span class="sxs-lookup"><span data-stu-id="63f32-160">To summarize the changes:</span></span>

- <span data-ttu-id="63f32-161">コードによって、 `ConfidentialClientApplication`既定のユーザートークンキャッシュがのインスタンスに置き換え`SessionTokenStore`られました。</span><span class="sxs-lookup"><span data-stu-id="63f32-161">The code now replaces the `ConfidentialClientApplication`'s default user token cache with an instance of the  `SessionTokenStore`.</span></span> <span data-ttu-id="63f32-162">MSAL ライブラリは、トークンを格納し、必要に応じて更新するロジックを処理します。</span><span class="sxs-lookup"><span data-stu-id="63f32-162">The MSAL library will handle the logic of storing the tokens and refreshing it when needed.</span></span>
- <span data-ttu-id="63f32-163">このコードでは、Microsoft Graph から取得したユーザーの`SessionTokenStore`詳細を、セッションに格納するオブジェクトに渡します。</span><span class="sxs-lookup"><span data-stu-id="63f32-163">The code now passes the user details obtained from Microsoft Graph to the `SessionTokenStore` object to store in the session.</span></span>
- <span data-ttu-id="63f32-164">成功した場合、コードはリダイレクトされず、ただ返します。</span><span class="sxs-lookup"><span data-stu-id="63f32-164">On success, the code no longer redirects, it just returns.</span></span> <span data-ttu-id="63f32-165">これにより、OWIN ミドルウェアは認証プロセスを完了することができます。</span><span class="sxs-lookup"><span data-stu-id="63f32-165">This allows the OWIN middleware to complete the authentication process.</span></span>

<span data-ttu-id="63f32-166">トークンキャッシュはセッションに格納されているため、 `SignOut`の`Controllers/AccountController.cs`アクションを更新して、サインアウトする前にトークンストアをクリアします。最初に、次`using`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="63f32-166">Since the token cache is stored in the session, update the `SignOut` action in `Controllers/AccountController.cs` to clear the token store before signing out. First, add the following `using` statement to the top of the file.</span></span>

```cs
using graph_tutorial.TokenStorage;
```

<span data-ttu-id="63f32-167">次に、既存`SignOut`の関数を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="63f32-167">Then, replace the existing `SignOut` function with the following.</span></span>

```cs
public ActionResult SignOut()
{
    if (Request.IsAuthenticated)
    {
        string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
        SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId, System.Web.HttpContext.Current);

        tokenStore.Clear();

        Request.GetOwinContext().Authentication.SignOut(
            CookieAuthenticationDefaults.AuthenticationType);
    }

    return RedirectToAction("Index", "Home");
}
```

<span data-ttu-id="63f32-168">キャッシュされたユーザーの詳細は、アプリケーションのすべてのビューで必要になる`BaseController`ため、この情報をセッションから読み込むようにクラスを更新します。</span><span class="sxs-lookup"><span data-stu-id="63f32-168">The cached user details are something that every view in the application will need, so update the `BaseController` class to load this information from the session.</span></span> <span data-ttu-id="63f32-169">を`Controllers/BaseController.cs`開き、次`using`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="63f32-169">Open `Controllers/BaseController.cs` and add the following `using` statements to the top of the file.</span></span>

```cs
using graph_tutorial.TokenStorage;
using System.Security.Claims;
using System.Web;
using Microsoft.Owin.Security.Cookies;
```

<span data-ttu-id="63f32-170">次に、次の関数を追加します。</span><span class="sxs-lookup"><span data-stu-id="63f32-170">Then add the following function.</span></span>

```cs
protected override void OnActionExecuting(ActionExecutingContext filterContext)
{
    if (Request.IsAuthenticated)
    {
        // Get the signed in user's id and create a token cache
        string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
        SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId, System.Web.HttpContext.Current);

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

<span data-ttu-id="63f32-171">サーバーを起動し、サインインプロセスを実行します。</span><span class="sxs-lookup"><span data-stu-id="63f32-171">Start the server and go through the sign-in process.</span></span> <span data-ttu-id="63f32-172">ホームページに戻る必要がありますが、UI は、サインインしていることを示すように変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="63f32-172">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![サインイン後のホームページのスクリーンショット](./images/add-aad-auth-01.png)

<span data-ttu-id="63f32-174">右上隅にあるユーザーアバターをクリックして、[**サインアウト**] リンクにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="63f32-174">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="63f32-175">[**サインアウト**] をクリックすると、セッションがリセットされ、ホームページに戻ります。</span><span class="sxs-lookup"><span data-stu-id="63f32-175">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![[サインアウト] リンクがあるドロップダウンメニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="63f32-177">トークンの更新</span><span class="sxs-lookup"><span data-stu-id="63f32-177">Refreshing tokens</span></span>

<span data-ttu-id="63f32-178">この時点で、アプリケーションには、API 呼び出しの`Authorization`ヘッダーで送信されるアクセストークンがあります。</span><span class="sxs-lookup"><span data-stu-id="63f32-178">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="63f32-179">これは、アプリがユーザーに代わって Microsoft Graph にアクセスできるようにするトークンです。</span><span class="sxs-lookup"><span data-stu-id="63f32-179">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="63f32-180">ただし、このトークンは存続期間が短くなります。</span><span class="sxs-lookup"><span data-stu-id="63f32-180">However, this token is short-lived.</span></span> <span data-ttu-id="63f32-181">トークンが発行された後、有効期限が切れる時間になります。</span><span class="sxs-lookup"><span data-stu-id="63f32-181">The token expires an hour after it is issued.</span></span> <span data-ttu-id="63f32-182">ここでは、更新トークンが有効になります。</span><span class="sxs-lookup"><span data-stu-id="63f32-182">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="63f32-183">更新トークンを使用すると、ユーザーが再度サインインする必要なく、新しいアクセストークンをアプリで要求できます。</span><span class="sxs-lookup"><span data-stu-id="63f32-183">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="63f32-184">アプリは MSAL ライブラリと`TokenCache`オブジェクトを使用しているため、トークン更新ロジックを実装する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="63f32-184">Because the app is using the MSAL library and a `TokenCache` object, you do not have to implement any token refresh logic.</span></span> <span data-ttu-id="63f32-185">メソッド`ConfidentialClientApplication.AcquireTokenSilentAsync`は、すべてのロジックを実行します。</span><span class="sxs-lookup"><span data-stu-id="63f32-185">The `ConfidentialClientApplication.AcquireTokenSilentAsync` method does all of the logic for you.</span></span> <span data-ttu-id="63f32-186">最初に、キャッシュされたトークンをチェックし、有効期限が切れていない場合はそれを返します。</span><span class="sxs-lookup"><span data-stu-id="63f32-186">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="63f32-187">有効期限が切れている場合は、キャッシュされた更新トークンを使用して新しいものを取得します。</span><span class="sxs-lookup"><span data-stu-id="63f32-187">If it is expired, it uses the cached refresh token to obtain a new one.</span></span> <span data-ttu-id="63f32-188">このメソッドは、次のモジュールで使用します。</span><span class="sxs-lookup"><span data-stu-id="63f32-188">You'll use this method in the following module.</span></span>