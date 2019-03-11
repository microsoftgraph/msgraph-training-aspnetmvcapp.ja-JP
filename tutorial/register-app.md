<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="987ef-101">この演習では、アプリケーションレジストリポータル (ARP) を使用して、新しい Azure AD web アプリケーション登録を作成します。</span><span class="sxs-lookup"><span data-stu-id="987ef-101">In this exercise, you will create a new Azure AD web application registration using the Application Registry Portal (ARP).</span></span>

1. <span data-ttu-id="987ef-102">ブラウザーを開き、[アプリケーション登録ポータル](https://apps.dev.microsoft.com)に移動します。</span><span class="sxs-lookup"><span data-stu-id="987ef-102">Open a browser and navigate to the [Application Registration Portal](https://apps.dev.microsoft.com).</span></span> <span data-ttu-id="987ef-103">**個人アカウント**(別名: Microsoft アカウント) または**職場または学校のアカウント**を使用してログインします。</span><span class="sxs-lookup"><span data-stu-id="987ef-103">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="987ef-104">ページの上部にある [**アプリの追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="987ef-104">Select **Add an app** at the top of the page.</span></span>

    > [!NOTE]
    > <span data-ttu-id="987ef-105">ページに [**アプリの追加**] ボタンが複数表示されている場合は、[収束した**アプリ**] リストに対応するボタンを選択します。</span><span class="sxs-lookup"><span data-stu-id="987ef-105">If you see more than one **Add an app** button on the page, select the one that corresponds to the **Converged apps** list.</span></span>

1. <span data-ttu-id="987ef-106">[**アプリケーションの登録**] ページで、[**アプリケーション名**] を「 **ASP.NET Graph チュートリアル**」に設定し、[**作成**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="987ef-106">On the **Register your application** page, set the **Application Name** to **ASP.NET Graph Tutorial** and select **Create**.</span></span>

    ![アプリ登録ポータル web サイトで新しいアプリを作成するスクリーンショット](./images/arp-create-app-01.png)

1. <span data-ttu-id="987ef-108">[ **ASP.NET Graph のチュートリアル登録**] ページの [**プロパティ**] セクションで、後で必要になるように**アプリケーション Id**をコピーします。</span><span class="sxs-lookup"><span data-stu-id="987ef-108">On the **ASP.NET Graph Tutorial Registration** page, under the **Properties** section, copy the **Application Id** as you will need it later.</span></span>

    ![新しく作成されたアプリケーションの ID のスクリーンショット](./images/arp-create-app-02.png)

1. <span data-ttu-id="987ef-110">[**アプリケーションシークレット**] セクションまで下にスクロールします。</span><span class="sxs-lookup"><span data-stu-id="987ef-110">Scroll down to the **Application Secrets** section.</span></span>

    1. <span data-ttu-id="987ef-111">[**新しいパスワードの生成**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="987ef-111">Select **Generate New Password**.</span></span>
    1. <span data-ttu-id="987ef-112">[**新しいパスワードが生成さ**れました] ダイアログで、後で必要になるように、ボックスの内容をコピーします。</span><span class="sxs-lookup"><span data-stu-id="987ef-112">In the **New password generated** dialog, copy the contents of the box as you will need it later.</span></span>

        > <span data-ttu-id="987ef-113">**重要:** このパスワードは今後表示されないため、ここでコピーしてください。</span><span class="sxs-lookup"><span data-stu-id="987ef-113">**Important:** This password is never shown again, so make sure you copy it now.</span></span>

    ![新しく作成されたアプリケーションのパスワードのスクリーンショット](./images/arp-create-app-03.png)

1. <span data-ttu-id="987ef-115">ASP.NET アプリの URL を決定します。</span><span class="sxs-lookup"><span data-stu-id="987ef-115">Determine your ASP.NET app's URL.</span></span> <span data-ttu-id="987ef-116">Visual Studio のソリューションエクスプローラーで、**グラフ-チュートリアル**プロジェクトを選択します。</span><span class="sxs-lookup"><span data-stu-id="987ef-116">In Visual Studio's Solution Explorer, select the **graph-tutorial** project.</span></span> <span data-ttu-id="987ef-117">[**プロパティ**] ウィンドウで、[ **URL**] の値を見つけます。</span><span class="sxs-lookup"><span data-stu-id="987ef-117">In the **Properties** window, find the value of **URL**.</span></span> <span data-ttu-id="987ef-118">この値をコピーします。</span><span class="sxs-lookup"><span data-stu-id="987ef-118">Copy this value.</span></span>

    ![Visual Studio の [プロパティ] ウィンドウのスクリーンショット](./images/vs-project-url.png)

1. <span data-ttu-id="987ef-120">[**プラットフォーム**] セクションまで下にスクロールします。</span><span class="sxs-lookup"><span data-stu-id="987ef-120">Scroll down to the **Platforms** section.</span></span>

    1. <span data-ttu-id="987ef-121">[**プラットフォームの追加**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="987ef-121">Select **Add Platform**.</span></span>
    1. <span data-ttu-id="987ef-122">[**プラットフォームの追加**] ダイアログで、[ **Web**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="987ef-122">In the **Add Platform** dialog, select **Web**.</span></span>

        ![アプリのプラットフォームを作成するスクリーンショット](./images/arp-create-app-04.png)

    1. <span data-ttu-id="987ef-124">[ **Web**プラットフォーム] ボックスに、Visual Studio プロジェクトの**リダイレクト url**のプロパティからコピーした URL を入力します。</span><span class="sxs-lookup"><span data-stu-id="987ef-124">In the **Web** platform box, enter the URL you copied from the Visual Studio project's properties for the **Redirect URLs**.</span></span>

        ![アプリケーションに新たに追加された Web プラットフォームのスクリーンショット](./images/arp-create-app-05.png)

1. <span data-ttu-id="987ef-126">ページの一番下までスクロールして、[**保存**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="987ef-126">Scroll to the bottom of the page and select **Save**.</span></span>