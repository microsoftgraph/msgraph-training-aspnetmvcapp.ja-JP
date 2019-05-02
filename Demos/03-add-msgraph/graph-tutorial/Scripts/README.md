<!-- IGNORE THE HTML BLOCK BELOW, THE INTERESTING PART IS AFTER IT -->

<h1 align="center"><span data-ttu-id="bbbee-101">Popper</span><span class="sxs-lookup"><span data-stu-id="bbbee-101">Popper.js</span></span></h1>

<p align="center"><span data-ttu-id="bbbee-102">
    <strong>Web アプリケーションで poppers 配置するために使用されるライブラリ。</strong>
</span><span class="sxs-lookup"><span data-stu-id="bbbee-102">
    <strong>A library used to position poppers in web applications.</strong>
</span></span></p>

<p align="center">
    <a href="https://travis-ci.org/FezVrasta/popper.js/branches" target="_blank"><img src="https://travis-ci.org/FezVrasta/popper.js.svg?branch=master" alt="Build Status"/></a>
    <img src="http://img.badgesize.io/https://unpkg.com/popper.js/dist/popper.min.js?compression=gzip" alt="Stable Release Size"/>
    <a href="https://www.bithound.io/github/FezVrasta/popper.js"><img src="https://www.bithound.io/github/FezVrasta/popper.js/badges/score.svg" alt="bitHound Overall Score"></a>
    <a href="https://codeclimate.com/github/FezVrasta/popper.js/coverage"><img src="https://codeclimate.com/github/FezVrasta/popper.js/badges/coverage.svg" alt="Istanbul Code Coverage"/></a>
    <a href="https://gitter.im/FezVrasta/popper.js" target="_blank"><img src="https://img.shields.io/gitter/room/nwjs/nw.js.svg" alt="Get support or discuss"/></a>
    <br />
    <a href="https://saucelabs.com/u/popperjs" target="_blank"><img src="https://badges.herokuapp.com/browsers?labels=none&googlechrome=latest&firefox=latest&microsoftedge=latest&iexplore=11,10&safari=latest&iphone=latest" alt="SauceLabs Reports"/></a>
</p>

<img src="https://raw.githubusercontent.com/FezVrasta/popper.js/master/popperjs.png" align="right" width=250 />

<!-- 🚨 HEY! HERE BEGINS THE INTERESTING STUFF 🚨 -->

## <a name="wut-poppers"></a><span data-ttu-id="bbbee-103">Wut?</span><span class="sxs-lookup"><span data-stu-id="bbbee-103">Wut?</span></span> <span data-ttu-id="bbbee-104">Poppers</span><span class="sxs-lookup"><span data-stu-id="bbbee-104">Poppers?</span></span>

<span data-ttu-id="bbbee-105">Popper は、画面上の要素で、アプリケーションの自然な流れから "ポップアウト" されます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-105">A popper is an element on the screen which "pops out" from the natural flow of your application.</span></span>  
<span data-ttu-id="bbbee-106">Poppers の一般的な例として、ポップヒント、popovers、ドロップダウンがあります。</span><span class="sxs-lookup"><span data-stu-id="bbbee-106">Common examples of poppers are tooltips, popovers and drop-downs.</span></span>


## <a name="so-yet-another-tooltip-library"></a><span data-ttu-id="bbbee-107">それでは、もう1つのヒントライブラリについて</span><span class="sxs-lookup"><span data-stu-id="bbbee-107">So, yet another tooltip library?</span></span>

<span data-ttu-id="bbbee-108">基本的には、**何も行いません**。</span><span class="sxs-lookup"><span data-stu-id="bbbee-108">Well, basically, **no**.</span></span>  
<span data-ttu-id="bbbee-109">Popper は**配置エンジン**です。その目的は、要素の位置を計算して、指定した reference 要素の近くに配置できるようにすることです。</span><span class="sxs-lookup"><span data-stu-id="bbbee-109">Popper.js is a **positioning engine**, its purpose is to calculate the position of an element to make it possible to position it near a given reference element.</span></span>  

<span data-ttu-id="bbbee-110">エンジンは完全にモジュール化されており、その機能のほとんどは**修飾子**として実装されています (middlewares またはプラグインに似ています)。</span><span class="sxs-lookup"><span data-stu-id="bbbee-110">The engine is completely modular and most of its features are implemented as **modifiers** (similar to middlewares or plugins).</span></span>  
<span data-ttu-id="bbbee-111">コードベース全体が ES2015 に記述されており、その機能は、 [SauceLabs](https://saucelabs.com/)と[travisci](https://travis-ci.org/)により、実際のブラウザーで自動的にテストされます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-111">The whole code base is written in ES2015 and its features are automatically tested on real browsers thanks to [SauceLabs](https://saucelabs.com/) and [TravisCI](https://travis-ci.org/).</span></span>

<span data-ttu-id="bbbee-112">Popper の依存関係が0である。</span><span class="sxs-lookup"><span data-stu-id="bbbee-112">Popper.js has zero dependencies.</span></span> <span data-ttu-id="bbbee-113">JQuery なし、LoDash なし、nothing。</span><span class="sxs-lookup"><span data-stu-id="bbbee-113">No jQuery, no LoDash, nothing.</span></span>  
<span data-ttu-id="bbbee-114">この技術は、[ブートストラップ v4 の Twitter](https://getbootstrap.com/)のような大規模企業、 [Webclipper](https://github.com/OneNoteDev/WebClipper)および[atlaskit の Atlassian](https://aui-cdn.atlassian.com/atlaskit/registry/)などに使用されます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-114">It's used by big companies like [Twitter in Bootstrap v4](https://getbootstrap.com/), [Microsoft in WebClipper](https://github.com/OneNoteDev/WebClipper) and [Atlassian in AtlasKit](https://aui-cdn.atlassian.com/atlaskit/registry/).</span></span>

### <a name="popperjs"></a><span data-ttu-id="bbbee-115">Popper</span><span class="sxs-lookup"><span data-stu-id="bbbee-115">Popper.js</span></span>

<span data-ttu-id="bbbee-116">これはエンジンです。このライブラリは、計算を行い、必要に応じて、スタイルを poppers 適用します。</span><span class="sxs-lookup"><span data-stu-id="bbbee-116">This is the engine, the library that computes and, optionally, applies the styles to the poppers.</span></span>

<span data-ttu-id="bbbee-117">主な点は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="bbbee-117">Some of the key points are:</span></span>

- <span data-ttu-id="bbbee-118">要素を元の DOM コンテキスト (DOM では混乱していません) に保持するように要素を配置します。</span><span class="sxs-lookup"><span data-stu-id="bbbee-118">Position elements keeping them in their original DOM context (doesn't mess with your DOM!);</span></span>
- <span data-ttu-id="bbbee-119">反応したその他のビューライブラリと統合するために、計算された結果をエクスポートできます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-119">Allows to export the computed informations to integrate with React and other view libraries;</span></span>
- <span data-ttu-id="bbbee-120">シャドウ DOM 要素をサポートします。</span><span class="sxs-lookup"><span data-stu-id="bbbee-120">Supports Shadow DOM elements;</span></span>
- <span data-ttu-id="bbbee-121">修飾子ベースの構造により、完全にカスタマイズできます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-121">Completely customizable thanks to the modifiers based structure;</span></span>

<span data-ttu-id="bbbee-122">Popper を使用して実行できるさまざまな例については、「[プロジェクトページ](https://fezvrasta.github.io/popper.js)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="bbbee-122">Visit our [project page](https://fezvrasta.github.io/popper.js) to see a lot of examples of what you can do with Popper.js!</span></span>

<span data-ttu-id="bbbee-123">[このドキュメントを](/docs/_includes/popper-documentation.md)参照してください。</span><span class="sxs-lookup"><span data-stu-id="bbbee-123">Find [the documentation here](/docs/_includes/popper-documentation.md).</span></span>


### <a name="tooltipjs"></a><span data-ttu-id="bbbee-124">ツールヒント .js</span><span class="sxs-lookup"><span data-stu-id="bbbee-124">Tooltip.js</span></span>

<span data-ttu-id="bbbee-125">多くのユーザーは、自分のプロジェクトに強力なツールヒントを統合する単純な方法を必要としているため、 **Tooltip**を作成しました。</span><span class="sxs-lookup"><span data-stu-id="bbbee-125">Since lots of users just need a simple way to integrate powerful tooltips in their projects, we created **Tooltip.js**.</span></span>  
<span data-ttu-id="bbbee-126">これは、エンジン Popper としてツールヒントを簡単に作成するための小さなライブラリです。</span><span class="sxs-lookup"><span data-stu-id="bbbee-126">It's a small library that makes it easy to automatically create tooltips using as engine Popper.js.</span></span>  
<span data-ttu-id="bbbee-127">この API は、ブートストラップの有名なツールチップシステムとほとんど同じですが、このようにしてプロジェクト内で容易に統合することができます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-127">Its API is almost identical to the famous tooltip system of Bootstrap, in this way it will be easy to integrate it in your projects.</span></span>  
<span data-ttu-id="bbbee-128">ツールヒントによって生成されたツールヒントに`aria`は、タグによってアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-128">The tooltips generated by Tooltip.js are accessible thanks to the `aria` tags.</span></span>

<span data-ttu-id="bbbee-129">[このドキュメントを](/docs/_includes/tooltip-documentation.md)参照してください。</span><span class="sxs-lookup"><span data-stu-id="bbbee-129">Find [the documentation here](/docs/_includes/tooltip-documentation.md).</span></span>


## <a name="installation"></a><span data-ttu-id="bbbee-130">インストール</span><span class="sxs-lookup"><span data-stu-id="bbbee-130">Installation</span></span>
<span data-ttu-id="bbbee-131">Popper は、次のパッケージマネージャーと CDNs で利用できます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-131">Popper.js is available on the following package managers and CDNs:</span></span>

| <span data-ttu-id="bbbee-132">Source</span><span class="sxs-lookup"><span data-stu-id="bbbee-132">Source</span></span> |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| <span data-ttu-id="bbbee-133">npm</span><span class="sxs-lookup"><span data-stu-id="bbbee-133">npm</span></span>    | `npm install popper.js --save`                                                   |
| <span data-ttu-id="bbbee-134">yarn</span><span class="sxs-lookup"><span data-stu-id="bbbee-134">yarn</span></span>   | `yarn add popper.js`                                                             |
| <span data-ttu-id="bbbee-135">NuGet</span><span class="sxs-lookup"><span data-stu-id="bbbee-135">NuGet</span></span>  | `PM> Install-Package popper.js`                                                  |
| <span data-ttu-id="bbbee-136">Bower</span><span class="sxs-lookup"><span data-stu-id="bbbee-136">Bower</span></span>  | `bower install popper.js --save`                     |
| <span data-ttu-id="bbbee-137">アンパッケージ</span><span class="sxs-lookup"><span data-stu-id="bbbee-137">unpkg</span></span>  | [`https://unpkg.com/popper.js`](https://unpkg.com/popper.js)                     |
| <span data-ttu-id="bbbee-138">cdnjs</span><span class="sxs-lookup"><span data-stu-id="bbbee-138">cdnjs</span></span>  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

<span data-ttu-id="bbbee-139">また、次のようにします。</span><span class="sxs-lookup"><span data-stu-id="bbbee-139">Tooltip.js as well:</span></span>

| <span data-ttu-id="bbbee-140">Source</span><span class="sxs-lookup"><span data-stu-id="bbbee-140">Source</span></span> |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| <span data-ttu-id="bbbee-141">npm</span><span class="sxs-lookup"><span data-stu-id="bbbee-141">npm</span></span>    | `npm install tooltip.js --save`                                                  |
| <span data-ttu-id="bbbee-142">yarn</span><span class="sxs-lookup"><span data-stu-id="bbbee-142">yarn</span></span>   | `yarn add tooltip.js`                                                            |
| <span data-ttu-id="bbbee-143">Bower \*</span><span class="sxs-lookup"><span data-stu-id="bbbee-143">Bower\*</span></span> | `bower install tooltip.js=https://unpkg.com/tooltip.js --save`                   |
| <span data-ttu-id="bbbee-144">アンパッケージ</span><span class="sxs-lookup"><span data-stu-id="bbbee-144">unpkg</span></span>  | [`https://unpkg.com/tooltip.js`](https://unpkg.com/tooltip.js)                   |
| <span data-ttu-id="bbbee-145">cdnjs</span><span class="sxs-lookup"><span data-stu-id="bbbee-145">cdnjs</span></span>  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

<span data-ttu-id="bbbee-146">\*: Bower は正式にはサポートされていません。 trough をインストールするには、unpkg.com CDN のみを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="bbbee-146">\*: Bower isn't officially supported, it can be used to install Tooltip.js only trough the unpkg.com CDN.</span></span> <span data-ttu-id="bbbee-147">このメソッドには、ライブラリの特定のバージョンを定義できないという制限があります。</span><span class="sxs-lookup"><span data-stu-id="bbbee-147">This method has the limitation of not being able to define a specific version of the library.</span></span> <span data-ttu-id="bbbee-148">Bower と Popper は、プロジェクトに npm または Yarn を使用することを推奨します。</span><span class="sxs-lookup"><span data-stu-id="bbbee-148">Bower and Popper.js suggests to use npm or Yarn for your projects.</span></span>  
<span data-ttu-id="bbbee-149">詳細については、[関連する問題を参照](https://github.com/FezVrasta/popper.js/issues/390)してください。</span><span class="sxs-lookup"><span data-stu-id="bbbee-149">For more info, [read the related issue](https://github.com/FezVrasta/popper.js/issues/390).</span></span>

### <a name="dist-targets"></a><span data-ttu-id="bbbee-150">Dist ターゲット</span><span class="sxs-lookup"><span data-stu-id="bbbee-150">Dist targets</span></span>

<span data-ttu-id="bbbee-151">Popper には、現在3つのターゲットが付属しています。 UMD、ESM、および ESNext。</span><span class="sxs-lookup"><span data-stu-id="bbbee-151">Popper.js is currently shipped with 3 targets in mind: UMD, ESM and ESNext.</span></span>

- <span data-ttu-id="bbbee-152">UMD-ユニバーサルモジュール定義: AMD、RequireJS および globals</span><span class="sxs-lookup"><span data-stu-id="bbbee-152">UMD - Universal Module Definition: AMD, RequireJS and globals;</span></span>
- <span data-ttu-id="bbbee-153">ESM-ES モジュール: webpack/Rollup またはブラウザー用の仕様をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="bbbee-153">ESM - ES Modules: For webpack/Rollup or browser supporting the spec;</span></span>
- <span data-ttu-id="bbbee-154">ESNext: で`dist/`利用できます、webpack と`babel-preset-env`共に使用できます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-154">ESNext: Available in `dist/`, can be used with webpack and `babel-preset-env`;</span></span>

<span data-ttu-id="bbbee-155">必要に応じて、適切なものを使用してください。</span><span class="sxs-lookup"><span data-stu-id="bbbee-155">Make sure to use the right one for your needs.</span></span> <span data-ttu-id="bbbee-156">`<script>`タグを使用してインポートする場合は、umd を使用します。</span><span class="sxs-lookup"><span data-stu-id="bbbee-156">If you want to import it with a `<script>` tag, use UMD.</span></span>

## <a name="usage"></a><span data-ttu-id="bbbee-157">使用方法</span><span class="sxs-lookup"><span data-stu-id="bbbee-157">Usage</span></span>

<span data-ttu-id="bbbee-158">既存の popper DOM ノードを指定した場合は、Popper にボタンの近くに配置するように指示します。</span><span class="sxs-lookup"><span data-stu-id="bbbee-158">Given an existing popper DOM node, ask Popper.js to position it near its button</span></span>

```js
var reference = document.querySelector('.my-button');
var popper = document.querySelector('.my-popper');
var anotherPopper = new Popper(
    reference,
    popper,
    {
        // popper options here
    }
);
```

### <a name="callbacks"></a><span data-ttu-id="bbbee-159">コールバック</span><span class="sxs-lookup"><span data-stu-id="bbbee-159">Callbacks</span></span>

<span data-ttu-id="bbbee-160">Popper では、2種類のコールバックが`onCreate`サポートされています。このコールバックは、Popper が初期化された後に呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-160">Popper.js supports two kinds of callbacks, the `onCreate` callback is called after the popper has been initalized.</span></span> <span data-ttu-id="bbbee-161">その`onUpdate`後の更新で呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-161">The `onUpdate` one is called on any subsequent update.</span></span>

```js
const reference = document.querySelector('.my-button');
const popper = document.querySelector('.my-popper');
new Popper(reference, popper, {
    onCreate: (data) => {
        // data is an object containing all the informations computed
        // by Popper.js and used to style the popper and its arrow
        // The complete description is available in Popper.js documentation
    },
    onUpdate: (data) => {
        // same as `onCreate` but called on subsequent updates
    }
});
```

### <a name="writing-your-own-modifiers"></a><span data-ttu-id="bbbee-162">独自の修飾子を記述する</span><span class="sxs-lookup"><span data-stu-id="bbbee-162">Writing your own modifiers</span></span>

<span data-ttu-id="bbbee-163">Popper は "プラグインのような" アーキテクチャに基づいていますが、その機能のほとんどは完全にカプセル化された "修飾子" です。</span><span class="sxs-lookup"><span data-stu-id="bbbee-163">Popper.js is based on a "plugin-like" architecture, most of its features are fully encapsulated "modifiers".</span></span>  
<span data-ttu-id="bbbee-164">修飾子は、Popper が Popper の位置を計算する必要があるたびに呼び出される関数です。</span><span class="sxs-lookup"><span data-stu-id="bbbee-164">A modifier is a function that is called each time Popper.js needs to compute the position of the popper.</span></span> <span data-ttu-id="bbbee-165">このため、ボトルネックを回避するには、モディファイアが非常に高性能である必要があります。</span><span class="sxs-lookup"><span data-stu-id="bbbee-165">For this reason, modifiers should be very performant to avoid bottlenecks.</span></span>  

<span data-ttu-id="bbbee-166">修飾子を作成する方法については、「[修飾子」のドキュメントを参照して](docs/_includes/popper-documentation.md#modifiers--object)ください。</span><span class="sxs-lookup"><span data-stu-id="bbbee-166">To learn how to create a modifier, [read the modifiers documentation](docs/_includes/popper-documentation.md#modifiers--object)</span></span>


### <a name="react-vuejs-angular-angularjs-emberjs-etc-integration"></a><span data-ttu-id="bbbee-167">応答、Vue、角度、AngularJS、Ember (その他) の統合</span><span class="sxs-lookup"><span data-stu-id="bbbee-167">React, Vue.js, Angular, AngularJS, Ember.js (etc...) integration</span></span>

<span data-ttu-id="bbbee-168">反応または他のライブラリにサードパーティ製ライブラリを統合することは、通常、DOM を変更してライブラリを変わってしまうため、問題になることがあります。</span><span class="sxs-lookup"><span data-stu-id="bbbee-168">Integrating 3rd party libraries in React or other libraries can be a pain because they usually alter the DOM and drive the libraries crazy.</span></span>  
<span data-ttu-id="bbbee-169">Popper では、 `applyStyle`修飾子内のすべての DOM の変更を制限することができます。これを無効にして、選択したライブラリを使用して、Popper 座標を手動で適用することができます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-169">Popper.js limits all its DOM modifications inside the `applyStyle` modifier, you can simply disable it and manually apply the popper coordinates using your library of choice.</span></span>  

<span data-ttu-id="bbbee-170">Popper を既存のフレームワークに使用できるライブラリの包括的な一覧については、「[メンション](/MENTIONS.md)」ページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="bbbee-170">For a comprehensive list of libraries that let you use Popper.js into existing frameworks, visit the [MENTIONS](/MENTIONS.md) page.</span></span>

<span data-ttu-id="bbbee-171">別の方法として、カスタム`applyStyles`のを使用して独自のものをオーバーライドして、Popper を自分で統合することもできます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-171">Alternatively, you may even override your own `applyStyles` with your custom one and integrate Popper.js by yourself!</span></span>

```js
function applyReactStyle(data) {
    // export data in your framework and use its content to apply the style to your popper
};

const reference = document.querySelector('.my-button');
const popper = document.querySelector('.my-popper');
new Popper(reference, popper, {
    modifiers: {
        applyStyle: { enabled: false },
        applyReactStyle: {
            enabled: true,
            fn: applyReactStyle,
            order: 800,
        },
    },
});

```

### <a name="migration-from-popperjs-v0"></a><span data-ttu-id="bbbee-172">Popper からの移行</span><span class="sxs-lookup"><span data-stu-id="bbbee-172">Migration from Popper.js v0</span></span>

<span data-ttu-id="bbbee-173">API が変更されたため、Popper v1 へのアップグレードが容易になるように移行手順をいくつか準備しました。</span><span class="sxs-lookup"><span data-stu-id="bbbee-173">Since the API changed, we prepared some migration instructions to make it easy to upgrade to Popper.js v1.</span></span>  

https://github.com/FezVrasta/popper.js/issues/62

<span data-ttu-id="bbbee-174">ご質問がある場合は、問題の中でコメントを付けることができます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-174">Feel free to comment inside the issue if you have any questions.</span></span>

### <a name="performances"></a><span data-ttu-id="bbbee-175">パフォーマンス</span><span class="sxs-lookup"><span data-stu-id="bbbee-175">Performances</span></span>

<span data-ttu-id="bbbee-176">Popper は非常に高いパフォーマンスを備えています。</span><span class="sxs-lookup"><span data-stu-id="bbbee-176">Popper.js is very performant.</span></span> <span data-ttu-id="bbbee-177">通常は、popper の位置 (3.5 G GHz Intel Core i5 を使用する iMac 上) を計算するのに0.5 ミリ秒かかります。</span><span class="sxs-lookup"><span data-stu-id="bbbee-177">It usually takes 0.5ms to compute a popper's position (on an iMac with 3.5G GHz Intel Core i5).</span></span>  
<span data-ttu-id="bbbee-178">これは、 [jank](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/anatomy-of-jank)を発生させないことを意味します。これは、スムーズなユーザー操作につながることです。</span><span class="sxs-lookup"><span data-stu-id="bbbee-178">This means that it will not cause any [jank](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/anatomy-of-jank), leading to a smooth user experience.</span></span>

## <a name="notes"></a><span data-ttu-id="bbbee-179">ノート</span><span class="sxs-lookup"><span data-stu-id="bbbee-179">Notes</span></span>

### <a name="libraries-using-popperjs"></a><span data-ttu-id="bbbee-180">Popper を使用したライブラリ</span><span class="sxs-lookup"><span data-stu-id="bbbee-180">Libraries using Popper.js</span></span>

<span data-ttu-id="bbbee-181">Popper の目的は、サードパーティ製のライブラリで使用できる、安定した、強力なポジショニングエンジンを提供することです。</span><span class="sxs-lookup"><span data-stu-id="bbbee-181">The aim of Popper.js is to provide a stable and powerful positioning engine ready to be used in 3rd party libraries.</span></span>  

<span data-ttu-id="bbbee-182">更新されたプロジェクトのリストについては、「[メンション](/MENTIONS.md)」ページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="bbbee-182">Visit the [MENTIONS](/MENTIONS.md) page for an updated list of projects.</span></span>


### <a name="credits"></a><span data-ttu-id="bbbee-183">開発者情報</span><span class="sxs-lookup"><span data-stu-id="bbbee-183">Credits</span></span>
<span data-ttu-id="bbbee-184">自分が行った作業について、いくつかの友人とプロジェクトに感謝します。</span><span class="sxs-lookup"><span data-stu-id="bbbee-184">I want to thank some friends and projects for the work they did:</span></span>

- <span data-ttu-id="bbbee-185">GitHub ページでの作業に[@AndreaScn](https://github.com/AndreaScn)し、開発中に実行した手動テストを行います。</span><span class="sxs-lookup"><span data-stu-id="bbbee-185">[@AndreaScn](https://github.com/AndreaScn) for his work on the GitHub Page and the manual testing he did during the development;</span></span>
- <span data-ttu-id="bbbee-186">元のアイデアおよびライブラリの名前の[@vampolo](https://github.com/vampolo)ます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-186">[@vampolo](https://github.com/vampolo) for the original idea and for the name of the library;</span></span>
- <span data-ttu-id="bbbee-187">[](https://github.com/Draios)この数年に学んだすべてのすばらしい事柄を、このライブラリを書くことができるようにしました。</span><span class="sxs-lookup"><span data-stu-id="bbbee-187">[Sysdig](https://github.com/Draios) for all the awesome things I learned during these years that made it possible for me to write this library;</span></span>
- <span data-ttu-id="bbbee-188">実際の環境に適した配置ライブラリを作成するためのヒントを持っている場合は、その他の[.js](http://github.hubspot.com/tether/)を使用できます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-188">[Tether.js](http://github.hubspot.com/tether/) for having inspired me in writing a positioning library ready for the real world;</span></span>
- <span data-ttu-id="bbbee-189">[投稿者は、](https://github.com/FezVrasta/popper.js/graphs/contributors)よく歓迎されるプル要求とバグレポートを作成できます。</span><span class="sxs-lookup"><span data-stu-id="bbbee-189">[The Contributors](https://github.com/FezVrasta/popper.js/graphs/contributors) for their much appreciated Pull Requests and bug reports;</span></span>
- <span data-ttu-id="bbbee-190">\*\*\*\* このプロジェクトに対してすばらしい星の場合は、このプロジェクトにお勧めします。🙂</span><span class="sxs-lookup"><span data-stu-id="bbbee-190">**you** for the star you'll give to this project and for being so awesome to give this project a try 🙂</span></span>

### <a name="copyright-and-license"></a><span data-ttu-id="bbbee-191">著作権とライセンス</span><span class="sxs-lookup"><span data-stu-id="bbbee-191">Copyright and license</span></span>
<span data-ttu-id="bbbee-192">コードおよびドキュメントの著作権 2016 **Federico Zivolo**。</span><span class="sxs-lookup"><span data-stu-id="bbbee-192">Code and documentation copyright 2016 **Federico Zivolo**.</span></span> <span data-ttu-id="bbbee-193">[MIT ライセンス](LICENSE.md)の下でリリースされたコード。</span><span class="sxs-lookup"><span data-stu-id="bbbee-193">Code released under the [MIT license](LICENSE.md).</span></span> <span data-ttu-id="bbbee-194">クリエイティブ Commons でリリースされた Docs。</span><span class="sxs-lookup"><span data-stu-id="bbbee-194">Docs released under Creative Commons.</span></span>
