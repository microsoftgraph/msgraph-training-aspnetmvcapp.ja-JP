<!-- IGNORE THE HTML BLOCK BELOW, THE INTERESTING PART IS AFTER IT -->

<h1 align="center">Popper</h1>

<p align="center">
    <strong>web アプリケーションで poppers 配置するために使用されるライブラリ。</strong>
</p>

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

## <a name="wut-poppers"></a>wut? poppers

popper は、画面上の要素で、アプリケーションの自然な流れから "ポップアウト" されます。  
poppers の一般的な例として、ポップヒント、popovers、ドロップダウンがあります。


## <a name="so-yet-another-tooltip-library"></a>それでは、もう1つのヒントライブラリについて

基本的には、**何も行いません**。  
Popper は**配置エンジン**です。その目的は、要素の位置を計算して、指定した reference 要素の近くに配置できるようにすることです。  

エンジンは完全にモジュール化されており、その機能のほとんどは**修飾子**として実装されています (middlewares またはプラグインに似ています)。  
コードベース全体が ES2015 に記述されており、その機能は、 [SauceLabs](https://saucelabs.com/)と[travisci](https://travis-ci.org/)により、実際のブラウザーで自動的にテストされます。

Popper の依存関係が0である。 jQuery なし、lodash なし、nothing。  
この技術は、[ブートストラップ v4 の Twitter](https://getbootstrap.com/)のような大規模企業、 [webclipper](https://github.com/OneNoteDev/WebClipper)および[atlaskit の Atlassian](https://aui-cdn.atlassian.com/atlaskit/registry/)などに使用されます。

### <a name="popperjs"></a>Popper

これはエンジンです。このライブラリは、計算を行い、必要に応じて、スタイルを poppers 適用します。

主な点は次のとおりです。

- 要素を元の dom コンテキスト (dom では混乱していません) に保持するように要素を配置します。
- 反応したその他のビューライブラリと統合するために、計算された結果をエクスポートできます。
- シャドウ DOM 要素をサポートします。
- 修飾子ベースの構造により、完全にカスタマイズできます。

Popper を使用して実行できるさまざまな例については、「[プロジェクトページ](https://fezvrasta.github.io/popper.js)」を参照してください。

[このドキュメントを](/docs/_includes/popper-documentation.md)参照してください。


### <a name="tooltipjs"></a>ツールヒント .js

多くのユーザーは、自分のプロジェクトに強力なツールヒントを統合する単純な方法を必要としているため、 **Tooltip**を作成しました。  
これは、エンジン Popper としてツールヒントを簡単に作成するための小さなライブラリです。  
この API は、ブートストラップの有名なツールチップシステムとほとんど同じですが、このようにしてプロジェクト内で容易に統合することができます。  
ツールヒントによって生成されたツールヒントに`aria`は、タグによってアクセスできます。

[このドキュメントを](/docs/_includes/tooltip-documentation.md)参照してください。


## <a name="installation"></a>インストール
Popper は、次のパッケージマネージャーと cdns で利用できます。

| Source |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| npm    | `npm install popper.js --save`                                                   |
| yarn   | `yarn add popper.js`                                                             |
| NuGet  | `PM> Install-Package popper.js`                                                  |
| Bower  | `bower install popper.js --save`                     |
| アンパッケージ  | [`https://unpkg.com/popper.js`](https://unpkg.com/popper.js)                     |
| cdnjs  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

また、次のようにします。

| Source |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| npm    | `npm install tooltip.js --save`                                                  |
| yarn   | `yarn add tooltip.js`                                                            |
| bower * | `bower install tooltip.js=https://unpkg.com/tooltip.js --save`                   |
| アンパッケージ  | [`https://unpkg.com/tooltip.js`](https://unpkg.com/tooltip.js)                   |
| cdnjs  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

\*: bower は正式にはサポートされていません。 trough をインストールするには、unpkg.com CDN のみを使用する必要があります。 このメソッドには、ライブラリの特定のバージョンを定義できないという制限があります。 bower と Popper は、プロジェクトに npm または Yarn を使用することを推奨します。  
詳細については、[関連する問題を参照](https://github.com/FezVrasta/popper.js/issues/390)してください。

### <a name="dist-targets"></a>Dist ターゲット

Popper には、現在3つのターゲットが付属しています。 umd、ESM、および esnext。

- umd-ユニバーサルモジュール定義: AMD、requirejs および globals
- ESM-ES モジュール: webpack/Rollup またはブラウザー用の仕様をサポートしています。
- esnext: で`dist/`利用できます、webpack と`babel-preset-env`共に使用できます。

必要に応じて、適切なものを使用してください。 `<script>`タグを使用してインポートする場合は、umd を使用します。

## <a name="usage"></a>使用方法

既存の popper DOM ノードを指定した場合は、popper にボタンの近くに配置するように指示します。

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

### <a name="callbacks"></a>コールバック

Popper では、2種類のコールバックが`onCreate`サポートされています。このコールバックは、Popper が初期化された後に呼び出されます。 その`onUpdate`後の更新で呼び出されます。

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

### <a name="writing-your-own-modifiers"></a>独自の修飾子を記述する

Popper は "プラグインのような" アーキテクチャに基づいていますが、その機能のほとんどは完全にカプセル化された "修飾子" です。  
修飾子は、Popper が Popper の位置を計算する必要があるたびに呼び出される関数です。 このため、ボトルネックを回避するには、モディファイアが非常に高性能である必要があります。  

修飾子を作成する方法については、「[修飾子」のドキュメントを参照して](docs/_includes/popper-documentation.md#modifiers--object)ください。


### <a name="react-vuejs-angular-angularjs-emberjs-etc-integration"></a>応答、vue、角度、AngularJS、Ember (その他) の統合

反応または他のライブラリにサードパーティ製ライブラリを統合することは、通常、DOM を変更してライブラリを変わってしまうため、問題になることがあります。  
Popper では、 `applyStyle`修飾子内のすべての DOM の変更を制限することができます。これを無効にして、選択したライブラリを使用して、Popper 座標を手動で適用することができます。  

Popper を既存のフレームワークに使用できるライブラリの包括的な一覧については、「[メンション](/MENTIONS.md)」ページを参照してください。

別の方法として、カスタム`applyStyles`のを使用して独自のものをオーバーライドして、Popper を自分で統合することもできます。

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

### <a name="migration-from-popperjs-v0"></a>Popper からの移行

API が変更されたため、Popper v1 へのアップグレードが容易になるように移行手順をいくつか準備しました。  

https://github.com/FezVrasta/popper.js/issues/62

ご質問がある場合は、問題の中でコメントを付けることができます。

### <a name="performances"></a>パフォーマンス

Popper は非常に高いパフォーマンスを備えています。 通常は、popper の位置 (3.5 g GHz Intel Core i5 を使用する imac 上) を計算するのに0.5 ミリ秒かかります。  
これは、 [jank](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/anatomy-of-jank)を発生させないことを意味します。これは、スムーズなユーザー操作につながることです。

## <a name="notes"></a>備考

### <a name="libraries-using-popperjs"></a>Popper を使用したライブラリ

Popper の目的は、サードパーティ製のライブラリで使用できる、安定した、強力なポジショニングエンジンを提供することです。  

更新されたプロジェクトのリストについては、「[メンション](/MENTIONS.md)」ページを参照してください。


### <a name="credits"></a>開発者情報
自分が行った作業について、いくつかの友人とプロジェクトに感謝します。

- GitHub ページでの作業に[@AndreaScn](https://github.com/AndreaScn)し、開発中に実行した手動テストを行います。
- 元のアイデアおよびライブラリの名前の[@vampolo](https://github.com/vampolo)ます。
- [](https://github.com/Draios)この数年に学んだすべてのすばらしい事柄を、このライブラリを書くことができるようにしました。
- 実際の環境に適した配置ライブラリを作成するためのヒントを持っている場合は、その他の[.js](http://github.hubspot.com/tether/)を使用できます。
- [投稿者は、](https://github.com/FezVrasta/popper.js/graphs/contributors)よく歓迎されるプル要求とバグレポートを作成できます。
- **** このプロジェクトに対してすばらしい星の場合は、このプロジェクトにお勧めします。🙂

### <a name="copyright-and-license"></a>著作権とライセンス
コードおよびドキュメントの著作権 2016 **Federico Zivolo**。 [MIT ライセンス](LICENSE.md)の下でリリースされたコード。 クリエイティブ Commons でリリースされた Docs。
