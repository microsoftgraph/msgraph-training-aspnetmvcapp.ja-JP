# <a name="how-to-run-the-completed-project"></a>完了したプロジェクトを実行する方法

## <a name="prerequisites"></a>前提条件

このフォルダーで完了したプロジェクトを実行するには、次のものが必要です。

- 開発用コンピューターにインストールされている[Visual Studio](https://visualstudio.microsoft.com/vs/) 。 Visual Studio を持っていない場合は、「ダウンロードオプション」の前のリンクにアクセスしてください。 (**注:** このチュートリアルは、Visual Studio 2019 version 16.2.3 を使用して作成されています。 このガイドの手順は、他のバージョンでは動作しますが、テストされていません。)
- Outlook.com 上のメールボックスを持つ個人の Microsoft アカウント、または Microsoft 職場または学校のアカウントのいずれか。

Microsoft アカウントを持っていない場合は、無料のアカウントを取得するためのオプションがいくつかあります。

- [新しい個人用 Microsoft アカウントにサインアップ](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)することができます。
- [Office 365 開発者プログラムにサインアップ](https://developer.microsoft.com/office/dev-program)して、無料の office 365 サブスクリプションを取得することができます。

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a>Web アプリケーションを Azure Active Directory 管理センターに登録する

1. ASP.NET アプリケーションの SSL URL を決定します。 Visual Studio のソリューションエクスプローラーで、**グラフ-チュートリアル**プロジェクトを選択します。 [**プロパティ**] ウィンドウで、[ **SSL URL**] の値を探します。 この値をコピーします。

    ![Visual Studio の [プロパティ] ウィンドウのスクリーンショット](/tutorial/images/vs-project-url.png)

1. ブラウザーを開き、[Azure Active Directory 管理センター](https://aad.portal.azure.com)に移動します。 **個人用アカウント** (別名: Microsoft アカウント)、または**職場/学校アカウント**を使用してログインします。

1. 左側のナビゲーションで [ **Azure Active Directory** ] を選択し、[**管理**] の下にある [**アプリの登録**] を選択します。

    ![アプリの登録のスクリーンショット ](/tutorial/images/aad-portal-app-registrations.png)

1. **[新規登録]** を選択します。 **[アプリケーションを登録]** ページで、次のように値を設定します。

    - `ASP.NET Graph Tutorial` に **[名前]** を設定します。
    - [**サポートされているアカウントの種類**] を [**任意の組織のディレクトリ内のアカウントと個人用の Microsoft アカウント**] に設定します。
    - [**リダイレクト URI**] で、最初のドロップダウン リストを `Web` に設定し、値を手順 1 でコピーした ASP.NET アプリ URL に設定します。

    ![[アプリケーションの登録] ページのスクリーンショット](/tutorial/images/aad-register-an-app.png)

1. [**登録**] を選択します。 [ **ASP.NET Graph のチュートリアル**] ページで、**アプリケーション (クライアント) ID**の値をコピーして保存します。次の手順で必要になります。

    ![新しいアプリの登録のアプリケーション ID のスクリーンショット](/tutorial/images/aad-application-id.png)

1. [**管理**] の下の [**認証**] を選択します。 **[暗黙的な許可]** セクションを検索し、**[ID トークン]** を有効にします。 **[保存]** を選択します。

    ![暗黙的な grant セクションのスクリーンショット](/tutorial/images/aad-implicit-grant.png)

1. **[管理]** で **[証明書とシークレット]** を選択します。 **[新しいクライアント シークレット]** ボタンを選択します。 **[説明]** に値を入力して、**[有効期限]** のオプションのいずれかを選び、**[追加]** を選択します。

    ![[クライアントシークレットの追加] ダイアログのスクリーンショット](/tutorial/images/aad-new-client-secret.png)

1. クライアント シークレットの値をコピーしてから、このページから移動します。 次の手順で行います。

    > [!IMPORTANT]
    > このクライアント シークレットは今後表示されないため、今必ずコピーするようにしてください。

    ![新しく追加されたクライアントシークレットのスクリーンショット](/tutorial/images/aad-copy-client-secret.png)

## <a name="configure-the-sample"></a>サンプルを構成する

1. ファイルの`PrivateSettings.config.example`名前を`PrivateSettings.config`に変更します。
1. `PrivateSettings.config`ファイルを編集し、次のように変更します。
    1. を`YOUR_APP_ID_HERE`アプリ登録ポータルで取得した**アプリケーション Id**に置き換えます。
    1. を`YOUR_APP_PASSWORD_HERE`アプリ登録ポータルから取得した**アプリケーションシークレット**に置き換えます。
1. Visual `graph-tutorial.sln` Studio で開きます。 ソリューションエクスプローラーで、**グラフチュートリアル**ソリューションを右クリックして、[ **NuGet パッケージの復元**] を選択します。

## <a name="run-the-sample"></a>サンプルを実行する

Visual Studio で、 **F5**キーを押すか、デバッグ > 選択して**デバッグを開始**します。
