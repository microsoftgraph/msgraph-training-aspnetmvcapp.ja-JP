<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9ef7f-101">このデモでは、アプリケーションに Microsoft Graph を組み込みます。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-101">In this demo you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="9ef7f-102">このアプリケーションでは、microsoft [Graph クライアントライブラリ](https://github.com/microsoftgraph/msgraph-sdk-dotnet)を使用して microsoft graph を呼び出すことにします。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="9ef7f-103">Outlook から予定表のイベントを取得する</span><span class="sxs-lookup"><span data-stu-id="9ef7f-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="9ef7f-104">まず、最後の`GraphHelper`モジュールで作成したクラスを拡張します。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-104">Start by extending the `GraphHelper` class you created in the last module.</span></span>

1. <span data-ttu-id="9ef7f-105">次`using`のステートメントを`Helpers/GraphHelper.cs`ファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-105">Add the following `using` statements to the top of the `Helpers/GraphHelper.cs` file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using Microsoft.Identity.Client;
    using System.Collections.Generic;
    using System.Configuration;
    using System.Linq;
    using System.Security.Claims;
    using System.Web;
    ```

1. <span data-ttu-id="9ef7f-106">次のコードを`GraphHelper`クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-106">Add the following code to the `GraphHelper` class.</span></span>

    ```cs
    // Load configuration settings from PrivateSettings.config
    private static string appId = ConfigurationManager.AppSettings["ida:AppId"];
    private static string appSecret = ConfigurationManager.AppSettings["ida:AppSecret"];
    private static string redirectUri = ConfigurationManager.AppSettings["ida:RedirectUri"];
    private static string graphScopes = ConfigurationManager.AppSettings["ida:AppScopes"];

    public static async Task<IEnumerable<Event>> GetEventsAsync()
    {
        var graphClient = GetAuthenticatedClient();

        var events = await graphClient.Me.Events.Request()
            .Select("subject,organizer,start,end")
            .OrderBy("createdDateTime DESC")
            .GetAsync();

        return events.CurrentPage;
    }

    private static GraphServiceClient GetAuthenticatedClient()
    {
        return new GraphServiceClient(
            new DelegateAuthenticationProvider(
                async (requestMessage) =>
                {
                    var idClient = ConfidentialClientApplicationBuilder.Create(appId)
                        .WithRedirectUri(redirectUri)
                        .WithClientSecret(appSecret)
                        .Build();

                    string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
                    var tokenStore = new SessionTokenStore(signedInUserId, HttpContext.Current);
                    tokenStore.Initialize(idClient.UserTokenCache);

                    var accounts = await idClient.GetAccountsAsync();

                    // By calling this here, the token can be refreshed
                    // if it's expired right before the Graph call is made
                    var scopes = graphScopes.Split(' ');
                    var result = await idClient.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                        .ExecuteAsync();

                    requestMessage.Headers.Authorization =
                        new AuthenticationHeaderValue("Bearer", result.AccessToken);
                }));
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="9ef7f-107">このコードの内容を検討してください。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-107">Consider what this code is doing.</span></span>
    >
    > - <span data-ttu-id="9ef7f-108">関数`GetAuthenticatedClient`は、を`GraphServiceClient`呼び出す`AcquireTokenSilent`認証プロバイダーでを初期化します。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-108">The `GetAuthenticatedClient` function initializes a `GraphServiceClient` with an authentication provider that calls `AcquireTokenSilent`.</span></span>
    > - <span data-ttu-id="9ef7f-109">`GetEventsAsync`関数の場合:</span><span class="sxs-lookup"><span data-stu-id="9ef7f-109">In the `GetEventsAsync` function:</span></span>
    >   - <span data-ttu-id="9ef7f-110">呼び出し先の URL は`/v1.0/me/events`になります。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-110">The URL that will be called is `/v1.0/me/events`.</span></span>
    >   - <span data-ttu-id="9ef7f-111">関数`Select`は、各イベントに対して返されるフィールドを、ビューが実際に使用するものだけに制限します。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-111">The `Select` function limits the fields returned for each events to just those the view will actually use.</span></span>
    >   - <span data-ttu-id="9ef7f-112">関数`OrderBy`は、生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-112">The `OrderBy` function sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="9ef7f-113">予定表ビュー用のコントローラーを作成します。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-113">Create a controller for the calendar views.</span></span> <span data-ttu-id="9ef7f-114">ソリューションエクスプローラーで [**コントローラー** ] フォルダーを右クリックし、[ **> コントローラーの追加**] を選択します。[ **MVC 5 コントローラー-空**] を選択し、[**追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-114">Right-click the **Controllers** folder in Solution Explorer and select **Add > Controller...**. Choose **MVC 5 Controller - Empty** and select **Add**.</span></span> <span data-ttu-id="9ef7f-115">コントローラー `CalendarController`の名前を指定して、[**追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-115">Name the controller `CalendarController` and select **Add**.</span></span> <span data-ttu-id="9ef7f-116">新しいファイルの内容全体を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-116">Replace the entire contents of the new file with the following code.</span></span>

    ```cs
    using System;
    using graph_tutorial.Helpers;
    using System.Threading.Tasks;
    using System.Web.Mvc;

    namespace graph_tutorial.Controllers
    {
        public class CalendarController : BaseController
        {
            // GET: Calendar
            [Authorize]
            public async Task<ActionResult> Index()
            {
                var events = await GraphHelper.GetEventsAsync();

                // Change start and end dates from UTC to local time
                foreach (var ev in events)
                {
                    ev.Start.DateTime = DateTime.Parse(ev.Start.DateTime).ToLocalTime().ToString();
                    ev.Start.TimeZone = TimeZoneInfo.Local.Id;
                    ev.End.DateTime = DateTime.Parse(ev.End.DateTime).ToLocalTime().ToString();
                    ev.End.TimeZone = TimeZoneInfo.Local.Id;
                }

                return Json(events, JsonRequestBehavior.AllowGet);
            }
        }
    }
    ```

1. <span data-ttu-id="9ef7f-117">アプリを起動し、サインインして、ナビゲーションバーの [**予定表**] リンクをクリックします。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-117">Start the app, sign in, and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="9ef7f-118">すべてが動作する場合は、ユーザーの予定表にイベントの JSON ダンプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="9ef7f-119">結果を表示する</span><span class="sxs-lookup"><span data-stu-id="9ef7f-119">Display the results</span></span>

<span data-ttu-id="9ef7f-120">これで、ビューを追加して、よりわかりやすい方法で結果を表示することができます。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-120">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="9ef7f-121">ソリューションエクスプローラーで、[**表示]/[予定表**] フォルダーを右クリックし、[ **Add >**] を選択します。ビュー `Index`の名前を指定して、[**追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-121">In Solution Explorer, right-click the **Views/Calendar** folder and select **Add > View...**. Name the view `Index` and select **Add**.</span></span> <span data-ttu-id="9ef7f-122">新しいファイルの内容全体を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-122">Replace the entire contents of the new file with the following code.</span></span>

    ```html
    @model IEnumerable<Microsoft.Graph.Event>

    @{
        ViewBag.Current = "Calendar";
    }

    <h1>Calendar</h1>
    <table class="table">
        <thead>
            <tr>
                <th scope="col">Organizer</th>
                <th scope="col">Subject</th>
                <th scope="col">Start</th>
                <th scope="col">End</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model)
            {
                <tr>
                    <td>@item.Organizer.EmailAddress.Name</td>
                    <td>@item.Subject</td>
                    <td>@Convert.ToDateTime(item.Start.DateTime).ToString("M/d/yy h:mm tt")</td>
                    <td>@Convert.ToDateTime(item.End.DateTime).ToString("M/d/yy h:mm tt")</td>
                </tr>
            }
        </tbody>
    </table>
    ```

    <span data-ttu-id="9ef7f-123">これにより、イベントのコレクションをループ処理して、テーブル行を1つずつ追加します。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-123">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="9ef7f-124">`Index`の`Controllers/CalendarController.cs`関数`return Json(events, JsonRequestBehavior.AllowGet);`から行を削除し、次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-124">Remove the `return Json(events, JsonRequestBehavior.AllowGet);` line from the `Index` function in `Controllers/CalendarController.cs`, and replace it with the following code.</span></span>

    ```cs
    return View(events);
    ```

1. <span data-ttu-id="9ef7f-125">アプリを起動し、サインインして、**予定表**のリンクをクリックします。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-125">Start the app, sign in, and click the **Calendar** link.</span></span> <span data-ttu-id="9ef7f-126">これで、アプリでイベントの表が表示されるはずです。</span><span class="sxs-lookup"><span data-stu-id="9ef7f-126">The app should now render a table of events.</span></span>

    ![イベントの表のスクリーンショット](./images/add-msgraph-01.png)
