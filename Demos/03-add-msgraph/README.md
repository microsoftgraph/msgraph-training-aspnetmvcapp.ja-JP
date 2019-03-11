# <a name="how-to-run-the-completed-project"></a><span data-ttu-id="586de-101">完了したプロジェクトを実行する方法</span><span class="sxs-lookup"><span data-stu-id="586de-101">How to run the completed project</span></span>

## <a name="prerequisites"></a><span data-ttu-id="586de-102">前提条件</span><span class="sxs-lookup"><span data-stu-id="586de-102">Prerequisites</span></span>

<span data-ttu-id="586de-103">このフォルダーで完了したプロジェクトを実行するには、次のものが必要です。</span><span class="sxs-lookup"><span data-stu-id="586de-103">To run the completed project in this folder, you need the following:</span></span>

- <span data-ttu-id="586de-104">開発用コンピューターにインストールされている[Visual Studio](https://visualstudio.microsoft.com/vs/) 。</span><span class="sxs-lookup"><span data-stu-id="586de-104">[Visual Studio](https://visualstudio.microsoft.com/vs/) installed on your development machine.</span></span> <span data-ttu-id="586de-105">Visual Studio を持っていない場合は、「ダウンロードオプション」の前のリンクにアクセスしてください。</span><span class="sxs-lookup"><span data-stu-id="586de-105">If you do not have Visual Studio, visit the previous link for download options.</span></span> <span data-ttu-id="586de-106">(**注:** このチュートリアルは、Visual Studio 2017 バージョン15.81 で記述されています。</span><span class="sxs-lookup"><span data-stu-id="586de-106">(**Note:** This tutorial was written with Visual Studio 2017 version 15.81.</span></span> <span data-ttu-id="586de-107">このガイドの手順は、他のバージョンでは動作しますが、テストされていません。)</span><span class="sxs-lookup"><span data-stu-id="586de-107">The steps in this guide may work with other versions, but that has not been tested.)</span></span>
- <span data-ttu-id="586de-108">Outlook.com 上のメールボックスを持つ個人の Microsoft アカウント、または microsoft 職場または学校のアカウントのいずれか。</span><span class="sxs-lookup"><span data-stu-id="586de-108">Either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span>

<span data-ttu-id="586de-109">Microsoft アカウントを持っていない場合は、無料のアカウントを取得するためのオプションがいくつかあります。</span><span class="sxs-lookup"><span data-stu-id="586de-109">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="586de-110">[新しい個人用 Microsoft アカウントにサインアップ](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)することができます。</span><span class="sxs-lookup"><span data-stu-id="586de-110">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="586de-111">[office 365 開発者プログラムにサインアップ](https://developer.microsoft.com/office/dev-program)して、無料の office 365 サブスクリプションを取得することができます。</span><span class="sxs-lookup"><span data-stu-id="586de-111">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

## <a name="register-a-web-application-with-the-application-registration-portal"></a><span data-ttu-id="586de-112">web アプリケーションをアプリケーション登録ポータルに登録する</span><span class="sxs-lookup"><span data-stu-id="586de-112">Register a web application with the Application Registration Portal</span></span>

1. <span data-ttu-id="586de-113">ブラウザーを開き、[アプリケーション登録ポータル](https://apps.dev.microsoft.com)に移動します。</span><span class="sxs-lookup"><span data-stu-id="586de-113">Open a browser and navigate to the [Application Registration Portal](https://apps.dev.microsoft.com).</span></span> <span data-ttu-id="586de-114">**個人アカウント**(別名: Microsoft アカウント) または**職場または学校のアカウント**を使用してログインします。</span><span class="sxs-lookup"><span data-stu-id="586de-114">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="586de-115">ページの上部にある [**アプリの追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="586de-115">Select **Add an app** at the top of the page.</span></span>

    > <span data-ttu-id="586de-116">**注:** ページに [**アプリの追加**] ボタンが複数表示されている場合は、[収束した**アプリ**] リストに対応するボタンを選択します。</span><span class="sxs-lookup"><span data-stu-id="586de-116">**Note:** If you see more than one **Add an app** button on the page, select the one that corresponds to the **Converged apps** list.</span></span>

1. <span data-ttu-id="586de-117">[**アプリケーションの登録**] ページで、[**アプリケーション名**] を「 **ASP.NET Graph チュートリアル**」に設定し、[**作成**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="586de-117">On the **Register your application** page, set the **Application Name** to **ASP.NET Graph Tutorial** and select **Create**.</span></span>

    ![アプリ登録ポータル web サイトで新しいアプリを作成するスクリーンショット](/tutorial/images/arp-create-app-01.png)

1. <span data-ttu-id="586de-119">[ **ASP.NET Graph のチュートリアル登録**] ページの [**プロパティ**] セクションで、後で必要になるように**アプリケーション Id**をコピーします。</span><span class="sxs-lookup"><span data-stu-id="586de-119">On the **ASP.NET Graph Tutorial Registration** page, under the **Properties** section, copy the **Application Id** as you will need it later.</span></span>

    ![新しく作成されたアプリケーションの ID のスクリーンショット](/tutorial/images/arp-create-app-02.png)

1. <span data-ttu-id="586de-121">[**アプリケーションシークレット**] セクションまで下にスクロールします。</span><span class="sxs-lookup"><span data-stu-id="586de-121">Scroll down to the **Application Secrets** section.</span></span>

    1. <span data-ttu-id="586de-122">[**新しいパスワードの生成**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="586de-122">Select **Generate New Password**.</span></span>
    1. <span data-ttu-id="586de-123">[**新しいパスワードが生成さ**れました] ダイアログで、後で必要になるように、ボックスの内容をコピーします。</span><span class="sxs-lookup"><span data-stu-id="586de-123">In the **New password generated** dialog, copy the contents of the box as you will need it later.</span></span>

        > <span data-ttu-id="586de-124">**重要:** このパスワードは今後表示されないため、ここでコピーしてください。</span><span class="sxs-lookup"><span data-stu-id="586de-124">**Important:** This password is never shown again, so make sure you copy it now.</span></span>

    ![新しく作成されたアプリケーションのパスワードのスクリーンショット](/tutorial/images/arp-create-app-03.png)

1. <span data-ttu-id="586de-126">[**プラットフォーム**] セクションまで下にスクロールします。</span><span class="sxs-lookup"><span data-stu-id="586de-126">Scroll down to the **Platforms** section.</span></span>

    1. <span data-ttu-id="586de-127">[**プラットフォームの追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="586de-127">Select **Add Platform**.</span></span>
    1. <span data-ttu-id="586de-128">[**プラットフォームの追加**] ダイアログで、[ **Web**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="586de-128">In the **Add Platform** dialog, select **Web**.</span></span>

        ![アプリのプラットフォームを作成するスクリーンショット](/tutorial/images/arp-create-app-04.png)

    1. <span data-ttu-id="586de-130">[ **Web**プラットフォーム] ボックスに、 `http://localhost:64107/` **リダイレクト url**の url を入力します。</span><span class="sxs-lookup"><span data-stu-id="586de-130">In the **Web** platform box, enter the URL `http://localhost:64107/` for the **Redirect URLs**.</span></span>

        ![アプリケーションに新たに追加された Web プラットフォームのスクリーンショット](/tutorial/images/arp-create-app-05.png)

1. <span data-ttu-id="586de-132">ページの一番下までスクロールして、[**保存**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="586de-132">Scroll to the bottom of the page and select **Save**.</span></span>

## <a name="configure-the-sample"></a><span data-ttu-id="586de-133">サンプルを構成する</span><span class="sxs-lookup"><span data-stu-id="586de-133">Configure the sample</span></span>

1. <span data-ttu-id="586de-134">ファイルの`PrivateSettings.config.example`名前を`PrivateSettings.config`に変更します。</span><span class="sxs-lookup"><span data-stu-id="586de-134">Rename the `PrivateSettings.config.example` file to `PrivateSettings.config`.</span></span>
1. <span data-ttu-id="586de-135">`PrivateSettings.config`ファイルを編集し、次のように変更します。</span><span class="sxs-lookup"><span data-stu-id="586de-135">Edit the `PrivateSettings.config` file and make the following changes.</span></span>
    1. <span data-ttu-id="586de-136">を`YOUR_APP_ID_HERE`アプリ登録ポータルで取得した**アプリケーション Id**に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="586de-136">Replace `YOUR_APP_ID_HERE` with the **Application Id** you got from the App Registration Portal.</span></span>
    1. <span data-ttu-id="586de-137">を`YOUR_APP_PASSWORD_HERE`アプリ登録ポータルから取得したパスワードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="586de-137">Replace `YOUR_APP_PASSWORD_HERE` with the password you got from the App Registration Portal.</span></span>
1. <span data-ttu-id="586de-138">Visual `graph-tutorial.sln` Studio で開きます。</span><span class="sxs-lookup"><span data-stu-id="586de-138">Open `graph-tutorial.sln` in Visual Studio.</span></span> <span data-ttu-id="586de-139">ソリューションエクスプローラーで、**グラフチュートリアル**ソリューションを右クリックして、[ **NuGet パッケージの復元**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="586de-139">In Solution Explorer, right-click the **graph-tutorial** solution and choose **Restore NuGet Packages**.</span></span>

## <a name="run-the-sample"></a><span data-ttu-id="586de-140">サンプルを実行する</span><span class="sxs-lookup"><span data-stu-id="586de-140">Run the sample</span></span>

<span data-ttu-id="586de-141">Visual Studio で、 **F5**キーを押すか、デバッグ > を選択して**デバッグを開始**します。</span><span class="sxs-lookup"><span data-stu-id="586de-141">In Visual Studio, press **F5** or choose **Debug > Start Debugging**.</span></span>