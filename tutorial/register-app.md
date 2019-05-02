<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f8e76-101">この演習では、Azure Active Directory 管理センターを使用して、新しい Azure AD web アプリケーション登録を作成します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-101">In this exercise, you will create a new Azure AD web application registration using the Azure Active Directory admin center.</span></span>

1. <span data-ttu-id="f8e76-102">ASP.NET アプリの URL を決定します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-102">Determine your ASP.NET app's URL.</span></span> <span data-ttu-id="f8e76-103">Visual Studio のソリューションエクスプローラーで、**グラフ-チュートリアル**プロジェクトを選択します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-103">In Visual Studio's Solution Explorer, select the **graph-tutorial** project.</span></span> <span data-ttu-id="f8e76-104">[**プロパティ**] ウィンドウで、**URL** の値を探します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-104">In the **Properties** window, find the value of **URL**.</span></span> <span data-ttu-id="f8e76-105">この値をコピーします。</span><span class="sxs-lookup"><span data-stu-id="f8e76-105">Copy this value.</span></span>

    ![Visual Studio の [プロパティ] ウィンドウのスクリーンショット](./images/vs-project-url.png)

1. <span data-ttu-id="f8e76-107">ブラウザーを開き、[Azure Active Directory 管理センター](https://aad.portal.azure.com)へ移動します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-107">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="f8e76-108">**個人用アカウント** (別名: Microsoft アカウント)、または**職場/学校アカウント**を使用してログインします。</span><span class="sxs-lookup"><span data-stu-id="f8e76-108">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="f8e76-109">左側のナビゲーションで [ **Azure Active Directory** ] を選択し、[**管理**] の下にある [**アプリの登録**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-109">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="f8e76-110">アプリの登録のスクリーンショット</span><span class="sxs-lookup"><span data-stu-id="f8e76-110">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

1. <span data-ttu-id="f8e76-111">**[新規登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-111">Select **New registration**.</span></span> <span data-ttu-id="f8e76-112">**[アプリケーションを登録]** ページで、次のように値を設定します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-112">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="f8e76-113">`ASP.NET Graph Tutorial` に **[名前]** を設定します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-113">Set **Name** to `ASP.NET Graph Tutorial`.</span></span>
    - <span data-ttu-id="f8e76-114">[**サポートされているアカウントの種類**] を [**任意の組織のディレクトリ内のアカウントと個人用の Microsoft アカウント**] に設定します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-114">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="f8e76-115">[**リダイレクト URI**] で、最初のドロップダウン リストを `Web` に設定し、値を手順 1 でコピーした ASP.NET アプリ URL に設定します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-115">Under **Redirect URI**, set the first drop-down to `Web` and set the value to the ASP.NET app URL you copied in step 1.</span></span>

    ![[アプリケーションの登録] ページのスクリーンショット](./images/aad-register-an-app.png)

1. <span data-ttu-id="f8e76-117">**[登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-117">Choose **Register**.</span></span> <span data-ttu-id="f8e76-118">[ **ASP.NET Graph のチュートリアル**] ページで、**アプリケーション (クライアント) ID**の値をコピーして保存します。次の手順で必要になります。</span><span class="sxs-lookup"><span data-stu-id="f8e76-118">On the **ASP.NET Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![新しいアプリの登録のアプリケーション ID のスクリーンショット](./images/aad-application-id.png)

1. <span data-ttu-id="f8e76-120">**[管理]** の下の **[認証]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-120">Select **Authentication** under **Manage**.</span></span> <span data-ttu-id="f8e76-121">**[暗黙的な許可]** セクションを検索し、**[ID トークン]** を有効にします。</span><span class="sxs-lookup"><span data-stu-id="f8e76-121">Locate the **Implicit grant** section and enable **ID tokens**.</span></span> <span data-ttu-id="f8e76-122">**[保存]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-122">Choose **Save**.</span></span>

    ![暗黙的な grant セクションのスクリーンショット](./images/aad-implicit-grant.png)

1. <span data-ttu-id="f8e76-124">**[管理]** で **[証明書とシークレット]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-124">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="f8e76-125">**[新しいクライアント シークレット]** ボタンを選択します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-125">Select the **New client secret** button.</span></span> <span data-ttu-id="f8e76-126">**[説明]** に値を入力して、**[有効期限]** のオプションのいずれかを選び、**[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-126">Enter a value in **Description** and select one of the options for **Expires** and choose **Add**.</span></span>

    ![[クライアントシークレットの追加] ダイアログのスクリーンショット](./images/aad-new-client-secret.png)

1. <span data-ttu-id="f8e76-128">クライアント シークレットの値をコピーしてから、このページから移動します。</span><span class="sxs-lookup"><span data-stu-id="f8e76-128">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="f8e76-129">次の手順で行います。</span><span class="sxs-lookup"><span data-stu-id="f8e76-129">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="f8e76-130">このクライアント シークレットは今後表示されないため、今必ずコピーするようにしてください。</span><span class="sxs-lookup"><span data-stu-id="f8e76-130">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![新しく追加されたクライアントシークレットのスクリーンショット](./images/aad-copy-client-secret.png)