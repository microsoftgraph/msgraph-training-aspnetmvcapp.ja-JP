<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b578c-101">このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報を取得する ASP.NET MVC web アプリを構築する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="b578c-101">This tutorial teaches you how to build an ASP.NET MVC web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="b578c-102">完成したチュートリアルをダウンロードするだけで済む場合は、2つの方法でダウンロードできます。</span><span class="sxs-lookup"><span data-stu-id="b578c-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="b578c-103">ASP.NET の[クイックスタート](https://developer.microsoft.com/graph/quick-start?platform=option-dotnet)をダウンロードして、作業中のコードを分単位で取得します。</span><span class="sxs-lookup"><span data-stu-id="b578c-103">Download the [ASP.NET quick start](https://developer.microsoft.com/graph/quick-start?platform=option-dotnet) to get working code in minutes.</span></span>
> - <span data-ttu-id="b578c-104">[GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp)をダウンロードするか、クローンを作成します。</span><span class="sxs-lookup"><span data-stu-id="b578c-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="b578c-105">前提条件</span><span class="sxs-lookup"><span data-stu-id="b578c-105">Prerequisites</span></span>

<span data-ttu-id="b578c-106">このチュートリアルを開始する前に、開発用のコンピューターに[Visual Studio](https://visualstudio.microsoft.com/vs/)がインストールされている必要があります。</span><span class="sxs-lookup"><span data-stu-id="b578c-106">Before you start this tutorial, you should have [Visual Studio](https://visualstudio.microsoft.com/vs/) installed on your development machine.</span></span> <span data-ttu-id="b578c-107">Visual Studio を持っていない場合は、「ダウンロードオプション」の前のリンクにアクセスしてください。</span><span class="sxs-lookup"><span data-stu-id="b578c-107">If you do not have Visual Studio, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="b578c-108">このチュートリアルは、Visual Studio 2019 version 16.4.0 を使用して作成されています。</span><span class="sxs-lookup"><span data-stu-id="b578c-108">This tutorial was written with Visual Studio 2019 version 16.4.0.</span></span> <span data-ttu-id="b578c-109">このガイドの手順は、他のバージョンでは動作しますが、テストされていません。</span><span class="sxs-lookup"><span data-stu-id="b578c-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="watch-the-tutorial"></a><span data-ttu-id="b578c-110">チュートリアルを見る</span><span class="sxs-lookup"><span data-stu-id="b578c-110">Watch the tutorial</span></span>

<span data-ttu-id="b578c-111">このモジュールは、Office 開発 YouTube チャネルで記録されており、利用できます。</span><span class="sxs-lookup"><span data-stu-id="b578c-111">This module has been recorded and is available in the Office Development YouTube channel.</span></span>

<!-- markdownlint-disable MD033 MD034 -->
<br/>

> [!VIDEO https://www.youtube-nocookie.com/embed/a2teHZ5WuNc]
<!-- markdownlint-enable MD033 MD034 -->

## <a name="feedback"></a><span data-ttu-id="b578c-112">フィードバック</span><span class="sxs-lookup"><span data-stu-id="b578c-112">Feedback</span></span>

<span data-ttu-id="b578c-113">このチュートリアルに関するフィードバックは、 [GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp)に記入してください。</span><span class="sxs-lookup"><span data-stu-id="b578c-113">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp).</span></span>
