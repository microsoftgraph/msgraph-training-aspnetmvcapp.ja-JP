<!-- markdownlint-disable MD002 MD041 -->

このデモでは、Microsoft Graph をアプリケーションに組み込みます。 このアプリケーションでは、microsoft [Graph クライアントライブラリ](https://github.com/microsoftgraph/msgraph-sdk-dotnet)を使用して microsoft graph を呼び出すことにします。

## <a name="get-calendar-events-from-outlook"></a>Outlook から予定表のイベントを取得する

まず、最後の`GraphHelper`モジュールで作成したクラスを拡張します。 最初に、次`using`のステートメントを`Helpers/GraphHelper.cs`ファイルの先頭に追加します。

```cs
using graph_tutorial.TokenStorage;
using Microsoft.Identity.Client;
using System.Collections.Generic;
using System.Configuration;
using System.Linq;
using System.Security.Claims;
using System.Web;
```

その後、次のコードを`GraphHelper`クラスに追加します。

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

このコードの内容を検討してください。

- 関数`GetAuthenticatedClient`は、を`GraphServiceClient`呼び出す`AcquireTokenSilent`認証プロバイダーでを初期化します。
- `GetEventsAsync`関数の場合:
  - 呼び出し先の URL は`/v1.0/me/events`になります。
  - 関数`Select`は、各イベントに対して返されるフィールドを、ビューが実際に使用するものだけに制限します。
  - 関数`OrderBy`は、生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。

ここで、予定表ビュー用のコントローラーを作成します。 ソリューションエクスプローラーで [**コントローラー** ] フォルダーを右クリックし、[ **> コントローラーの追加**] を選択します。[ **MVC 5 コントローラー-空**] を選択し、[**追加**] を選択します。 コントローラー `CalendarController`の名前を指定して、[**追加**] を選択します。 新しいファイルの内容全体を次のコードに置き換えます。

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

これで、これをテストできます。 アプリを起動し、サインインして、ナビゲーションバーの [**予定表**] リンクをクリックします。 すべてが動作する場合は、ユーザーの予定表にイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果を表示する

これで、ビューを追加して、よりわかりやすい方法で結果を表示することができます。 ソリューションエクスプローラーで、[**表示]/[予定表**] フォルダーを右クリックし、[ **Add > View...**] を選択します。ビュー `Index`の名前を指定して、[**追加**] を選択します。 新しいファイルの内容全体を次のコードに置き換えます。

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

これにより、イベントのコレクションをループ処理して、テーブル行を1つずつ追加します。 `Index`の`Controllers/CalendarController.cs`関数`return Json(events, JsonRequestBehavior.AllowGet);`から行を削除し、次のコードに置き換えます。

```cs
return View(events);
```

アプリを起動し、サインインして、**予定表**のリンクをクリックします。 これで、アプリでイベントの表が表示されるはずです。

![イベントの表のスクリーンショット](./images/add-msgraph-01.png)