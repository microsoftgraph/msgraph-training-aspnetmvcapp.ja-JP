<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a71a0-101">このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報を取得する ASP.NET MVC web アプリを構築する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="a71a0-101">This tutorial teaches you how to build an ASP.NET MVC web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="a71a0-102">完成したチュートリアルをダウンロードするだけで済む場合は、2つの方法でダウンロードできます。</span><span class="sxs-lookup"><span data-stu-id="a71a0-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="a71a0-103">ASP.NET の[クイックスタート](https://developer.microsoft.com/graph/quick-start?platform=option-dotnet)をダウンロードして、作業中のコードを分単位で取得します。</span><span class="sxs-lookup"><span data-stu-id="a71a0-103">Download the [ASP.NET quick start](https://developer.microsoft.com/graph/quick-start?platform=option-dotnet) to get working code in minutes.</span></span>
> - <span data-ttu-id="a71a0-104">[GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp)をダウンロードするか、クローンを作成します。</span><span class="sxs-lookup"><span data-stu-id="a71a0-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="a71a0-105">前提条件</span><span class="sxs-lookup"><span data-stu-id="a71a0-105">Prerequisites</span></span>

<span data-ttu-id="a71a0-106">このチュートリアルを開始する前に、開発用のコンピューターに[Visual Studio](https://visualstudio.microsoft.com/vs/)がインストールされている必要があります。</span><span class="sxs-lookup"><span data-stu-id="a71a0-106">Before you start this tutorial, you should have [Visual Studio](https://visualstudio.microsoft.com/vs/) installed on your development machine.</span></span> <span data-ttu-id="a71a0-107">Visual Studio を持っていない場合は、「ダウンロードオプション」の前のリンクにアクセスしてください。</span><span class="sxs-lookup"><span data-stu-id="a71a0-107">If you do not have Visual Studio, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="a71a0-108">このチュートリアルは、Visual Studio 2017 バージョン15.81 で記述されています。</span><span class="sxs-lookup"><span data-stu-id="a71a0-108">This tutorial was written with Visual Studio 2017 version 15.81.</span></span> <span data-ttu-id="a71a0-109">このガイドの手順は、他のバージョンでは動作しますが、テストされていません。</span><span class="sxs-lookup"><span data-stu-id="a71a0-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="a71a0-110">フィードバック</span><span class="sxs-lookup"><span data-stu-id="a71a0-110">Feedback</span></span>

<span data-ttu-id="a71a0-111">このチュートリアルに関するフィードバックは、 [GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp)に記入してください。</span><span class="sxs-lookup"><span data-stu-id="a71a0-111">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp).</span></span>