# CSS Animation について深く知る

## 要素の移動について

要素を縦横にアニメーションを伴って動かしたい場合 `transition` を適応させた要素に対して `right, left, top, bottom` や `transform: translate(X, Y)` のプロパティーを追加、またはその値を変更することで実現させることができる。

移動には `right, left, top, bottom` よりも `transform: translate(X, Y)` を使用した方が滑らかなアニメーションを実現することができます。

### なぜ transform: translate による移動の方が優れているか

それは `right, left, top, bottom` と `transform: translate` で要素の描画（レンダリング）方法が異なることに起因します。

簡単に説明すると `right, left, top, bottom` での移動は CPU で処理されます。  
`transform: translate` での移動は GPU で処理されます。  
単純な話グラフィック系の処理は CPU よりも GPU 側が得意なため描画処理がスムーズになります。

## CPU と GPU でのレンダリングの違い

描画済みの要素が再び何らかのアクションを起点に動く際、CPU と GPU 処理とでレンダリング方法が異なります。  

* CPU：  
  要素の位置が変更される都度 __ペイント（ピクセルの描画）処理__ が行われレンダリングされます。  
* GPU：  
__レイヤー__ という概念を用い、その __レイヤー__ を移動させ __合成（Composite）__ が行われレンダリングされます。

### 要素のレンダリングについて

要素が描画（レンダリング）されるまで大まかに分けて下記４つの工程を通ります。

1. Style（スタイルの適応）
2. Layout（スタイルの計算および要素の配置）
3. Paint（ピクセルの描画）
4. Composite（レイヤーの合成 = 定着）

ただし、描画済みの要素のスタイルに後で何らかの変更が生じる際は必ずしも４つの工程は通りません。  
例えば要素に対して「文字色の変更」だけであれば特に要素に変形が生じたりする訳ではないので、Layout（スタイルの再計算および再配置）は行われません。  
１（Style） と ４（Composite） は必ず行われますが、２（Layout）と３（Paint）についてはスタイルの変更方法次第では省くことが可能です。Layout と Paint はコストのかかる処理なため、 __これを省くことでスムーズなアニメーションが実現可能__ となります。

### レイヤーという概念について

> Webページは、ひとたびロードされ、パースされると、多くのWebデベロッパーにおなじみの構造、つまりDOMに変換されます。しかしながら、ページをレンダリングするに当たって、ブラウザはデベロッパーが直接のぞくことのできないいくつかの中間形式を持ちます。これらのうち最も重要なものはレイヤーです。  
[html5rocks.com 「レイヤーとは」より引用](https://www.html5rocks.com/ja/tutorials/speed/layers/)

下記CSSプロパティーなどの使用で要素がレイヤー化されるそうです。

* position
* transform
* opacity

Paint（ピクセルの描画）はレイヤー単位で行われる。  
これは再度描画が必要になった際、既に描画済みのレイヤーを再利用することで効率的にレンダリングを行うことができるためです。

### レイヤーの種類について

レイヤーには大まかに分けて RenderLayer と GraphicLayer が存在する
> Chromeにおいては、実際のところ複数の異なるタイプのレイヤーが存在します。RenderレイヤーはDOMのサブツリーに関して責任を持ちます。また、GraphicsレイヤーはRenderレイヤーのサブツリーに関して責任を持ちます。後者は我々にとってより興味深いものとなります。なぜならGraphicsレイヤーはテクスチャとしてGPUにアップロードされるからです。以後、単に「レイヤー」と行った場合はこのGraphicsレイヤーを指します。  
[html5rocks.com 「レイヤーとは」より引用](https://www.html5rocks.com/ja/tutorials/speed/layers/)

この内、GraphicLayer は GPU で処理されるレイヤーであり、Chrome の DevTools 実際に確認することができます。CSS の performance のために説明される「レイヤー」とはこの GraphicLayer を指すことが殆どだと思います。  

## レイヤー（GPU）の生成について

__※ 以降の説明において「レイヤー」とは GPU で処理されるレイヤーを指します。__

レイヤーの生成には要素に `will-change` という CSS プロパティーを使用します。`will-change` に必要な値については [こちら](https://developer.mozilla.org/ja/docs/Web/CSS/will-change) を参照してください。

`will-change` は比較的新しめのプロパティーであり、それまでは一般的にレイヤーを生成するには 3d 関連の CSS プロパティー（`transform: translateZ(0)`など）を適応する方法が取られてきました。また描画処理を GPU に任せる行為は「ハードウェア・アクセラレーション」と呼ばれます。要素のアニメーションがカクツク場合に 3d 関連の CSS プロパティーをとりあえず当てて GPU で処理させる行為は CSS ハック的な方法として認知されてきました。

`will-change` がプロパティーが登場したことにより、開発者は CSS ハック的な方法に頼らず明示的に レイヤーを生成させておくことが可能になりました。ただ現状 IE11, Edge などでは対応してませんので、いまだに `transform: translaetZ(0)` を適応する方法も取られています。

### ハードウェア・アクセラレーションは銀の弾丸ではない

GPU の方がグラフィックの処理が得意ならアニメーションする要素全てに `transform: translateZ(0)` を使用すればいいのでは？という考えが思い浮かぶかも知れないが、それは間違いである。GPU 処理ではレイヤーに要素のグラフィック情報を保持しておく必要があるため、メモリーが割り当てられます。大量のレイヤーを生成するということは、それだけメモリーを消費するということであり、返ってコンピュータのパフォーマンスを悪化させる原因となります。

### transform: translate(X, Y) などの 2d プロパティーではレイヤーは生成されないのか

はじめの方に `right, left, top, bottom` は CPU で処理され、 `transform: translate(X, Y)` は GPU で処理されると説明しました。`transform: translate(X, Y)` などの 2d プロパティーでもレイヤーは生成されます。ただし、それはアニメーションの直前〜終了の間までの限定的なレイヤーの生成となります。`will-change` や `transform: translateZ(0)` が適応された要素では、アニメーションされる前からレイヤー生成が成され、それは `will-change` や `transform: translateZ(0)` が指定されている間ずっと保持されます。

### 実際にレイヤーの生成を確認する

実際のレイヤー生成図（Chrome DevTools での表示）を確認します。

#### `will-change: transform` の適応

div#test01 が #document とは別にレイヤー化され浮いているのが分かります。  
<img src="https://github.com/honjio/my-code-note/blob/master/css-performance-190704/reference-img/layer-willchange.png?raw=true" width="560">

#### `transform: translateZ(0)` の適応

`will-change` の場合とは違う位置（style 上の本来の位置）で浮動化していますが、  
div#test01 が #document とは別にレイヤー化され浮いているのが分かります。  
<img src="https://github.com/honjio/my-code-note/blob/master/css-performance-190704/reference-img/layer-3d.png?raw=true" width="560">

#### `transform: translateX(0)` の適応

`will-change` と `transform: translateZ(0)` の場合とは異なり、レイヤーが生成されていないのが分かります。  
ただしアニメーション時にレイヤー生成され浮動化します（DevTools で確認済）。  
<img src="https://github.com/honjio/my-code-note/blob/master/css-performance-190704/reference-img/layer-2d.png?raw=true" width="560">

## 参考資料

* [Google_Accelerated Rendering in Chrome](https://www.html5rocks.com/ja/tutorials/speed/layers/)
* [Google_レンダリング パフォーマンス](https://developers.google.com/web/fundamentals/performance/rendering/?hl=ja)
* [ブラウザにやさしいHTML/CSS](https://www.slideshare.net/TakeharuIgari/htmlcss-34506501)
* [ブラウザレンダリングの仕組み](https://student-engineer.net/blowser-rendering/)
* [hatena_ブラウザのレンダリングの仕組み](http://cidermitaina.hatenablog.com/entry/2019/03/03/232516)
* [wpj_60fpsを実現するベストプラクティス](https://www.webprofessional.jp/achieve-60-fps-mobile-animations-with-css3/)
* [Dev.Opera_CSS will-changeプロパティについて知っておくべきこと](https://dev.opera.com/articles/ja/css-will-change-property/)
* [Qiita_スクロールが軽快に！will-change属性をつけるだけでFPSが...](https://qiita.com/ttiger55/items/b2423cb72668c3c98d89)
* [Fix scrolling performance with CSS will-change property](https://www.fourkitchens.com/blog/article/fix-scrolling-performance-css-will-change-property/)
* [github_Consider adding 'will-change: transform' even if translate3d if disabled](https://github.com/Microsoft/monaco-editor/issues/426#issuecomment-308395469)

<!-- 
<img src="https://github.com/honjio/my-code-note/blob/master/css-performance-190704/reference-img/g-move-willchange.gif"> -->