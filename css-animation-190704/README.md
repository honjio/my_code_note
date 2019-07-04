# CSS アニメーションについて深く知る

CSS で実装する要素の移動（アニメーション）に関しての簡単な説明から、パフォーマンスに関連する事象を深掘って説明していきます。この辺りの話はややこしいので、自身でも整理をつけるためにまとめました。  
長い記事になりますが、CSS のアニメーション（パフォーマンス関連）を深く理解するための手助けになれば幸いです。  
既にご存知の方はどこか間違っている点などあればご指摘宜しくお願いいたします。

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

※ レンダリング済要素に再度変更があった際に行われる Layout 処理は __リフロー__ Paint 処理は __リペイント__ とも呼ばれます。

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

#### will-change: transform の適応

div#test01 が #document とは別にレイヤー化され浮いているのが分かります。  
<img src="https://github.com/honjio/my-code-note/blob/master/css-animation-190704/reference-img/layer-willchange.png?raw=true" width="560">

#### transform: translateZ(0) の適応

`will-change` の場合とは違う位置（style 指定上の本来の位置）で浮動化していますが、  
div#test01 が #document とは別にレイヤー化され浮いているのが分かります。  
<img src="https://github.com/honjio/my-code-note/blob/master/css-animation-190704/reference-img/layer-3d.png?raw=true" width="560">

#### transform: translateX(0) の適応

`will-change` と `transform: translateZ(0)` の場合とは異なり、レイヤーが生成されていないのが分かります。  
ただしアニメーション時にレイヤー生成され浮動化します（DevTools で確認済）。  
<img src="https://github.com/honjio/my-code-note/blob/master/css-animation-190704/reference-img/layer-2d.png?raw=true" width="560">

## レイヤーがあらかじめ生成されている利点

`transform: translate(X,Y)` の他に `opacity` などのプロパティーが変化する際は GPU 処理されるためレイヤー生成されます。Paint 処理を伴う `right, left, top, bottom` よりは移動のアニメーションがスムーズに行われますが、レイヤーの生成は高コストなため場合によってはカクツキなどが発生する可能性があります。  

`will-change` や `transform: translateZ(0)` などで予めレイヤーを生成しておくことで、アニメーション前に発生するレイヤー生成のコストを削減する事ができます。これによりアニメーションのカクツキが改善される場合があります。

## CPU 処理での移動と GPU処理での移動を比較する

### レンダリングの可視化

Chrome の DevTools では要素が Paint される瞬間を可視化することができます。  
Paint が行われた箇所は緑色で表示（可視化）されます。  
また、レンダリング（Style, Layout, Paint, Composite）のログを確認することもできます。  
下記はそれぞれ要素に当てるプロパティーの違いによるレンダリングの差を可視化したものです。  

#### CPU処理 right, left, top, bottom での移動

移動に伴い緑色枠も密着するようについてきているのが分かります。  
移動の度毎回 Paint 処理が走っているのを確認できます。  
ログのイメージからも Layout と Paint 処理が毎回挟まっているのが確認できます。  
<img src="https://github.com/honjio/my-code-note/blob/master/css-animation-190704/reference-img/g-move-left.gif?raw=true" width="560">  
<img src="https://github.com/honjio/my-code-note/blob/master/css-animation-190704/reference-img/log-left-interval.png?raw=true" width="560">

#### GPU処理 transform: tlanslate(X,Y) または `transform: tlanslate(X, Y, Z) での移動

最初と最後に少し Paint 処理が挟まっていますが、  
移動区間では Paint 処理が走っていないことを確認できます。  
ログも `right, left, top, bottom` での移動とは異なり Layout 処理が回避され Paint 処理が減っているのを確認できます。  
<img src="https://github.com/honjio/my-code-note/blob/master/css-animation-190704/reference-img/g-move-2d-3d.gif?raw=true" width="560">  
<img src="https://github.com/honjio/my-code-note/blob/master/css-animation-190704/reference-img/log-2d-3d-interval.png?raw=true" width="560">

#### GPU処理 will-change: transform + transform(X, Y) での移動  

一度も Paint 処理が成されず移動できていることが確認できます。  
ログも Layout, Paint 処理が一切無いことを確認できます。  
<img src="https://github.com/honjio/my-code-note/blob/master/css-animation-190704/reference-img/g-move-willchange.gif?raw=true" width="560">  
<img src="https://github.com/honjio/my-code-note/blob/master/css-animation-190704/reference-img/log-willchange-2d-interval.png?raw=true" width="560">

#### GPU処理 will-change: trnasform + left による移動（GIF 画像なし）

GIF 画像は無しですが、「will-change: transform + transform(X, Y)」の場合と同じです。  
ログを見ると「will-change: transform + transform(X, Y)」の場合とは違って Layout 処理のみ発生していますが、Paint 処理はされずにアニメーションできています。  
これは `left` プロパティーで移動しているが、`will-change` によりレイヤー化されているためです。  
しかしながら、 Layout 処理を発生させない `transform: translate(X, Y)` での移動がパフォーマンス上有利になります。  
<img src="https://github.com/honjio/my-code-note/blob/master/css-animation-190704/reference-img/log-willchange-left-interval.png?raw=true" width="560">

## will-change はどのような場面で使用するべきか

あくまで私個人の考察になりますが、下記の場合に使用の検討が考えられるかと思います。

* `transform: translate(X, Y)` による移動を行なっているが、それでもカクツク場合。
* アニメーションの際に、他にも高コスト（※１）なプロパティーの変更を行う要素の場合。
* アニメーションを滑らかにしたい + アニメーションの頻度が高いと思われる要素。

※１. 高コストなプロパティー
> color: rgba(), border-radius, box-shadow, text-shadow, linear-gradient, position: fixed... など

様々なページ（以下）から一部抜粋

* [楽しく役に立つCSSのプロファイリング](https://postd.cc/profiling-css-for-fun-and-profit-optimization-notes/)  
* [プロパティやセレクタがパフォーマンスに与える影響](https://coliss.com/articles/build-websites/operation/css/things-nobody-ever-taught-me-about-css.html)  
* [レンダリングを意識したパフォーマンスチューニング](https://www.slideshare.net/hayatomizuno/ss-23379553)

### will-change 使用の留意点

 [こちら（MDN）](https://developer.mozilla.org/ja/docs/Web/CSS/will-change) にも記載されていますが、頻繁に使うべきプロパティーではありません。
 style に静的に記述する場合は使用頻度を控えめに押さえましょう  

 また、一度だけ行われるリッチなアニメーション（例えばページを表示する前の intro アニメーション）などがある場合は、アニメーションの前に JavaScript で will-change プロパティーを付与し、アニメーション終了後に取り除くといった処理が効果的だと思われます。

## パフォーマンスの観点でアニメーション実装の際に気をつけられる事

### リフローとリペイントを避ける

最初の方に少し記載しましたが、Layout（リフロー）, Paint（リペイント） 処理の発生を抑える事が大事です。  

> Reflowを減らすには
> * 細かい単位でスタイルを変更せず、できればクラス一発で切り替える
> * DOMに要素を追加する場合も、documentFragmentなどを使って、一気に追加する
> * DOMへの追加前にスタイルを整えて、それからDOMに追加する
>
> [Qiita「Reflowを制するものはDOMを制す」より引用](https://qiita.com/jkr_2255/items/5cdead4ee7fa289bfeed) 

> ウェブページのリフローを最小限に抑えるための簡単なガイドラインをいくつか紹介します
> * 必要以上に DOM を深くしないようにします。DOM ツリー内の 1 階層での変更が、上はルート、下は変更されたノードの子に至るまで、ツリー内の全階層での変更の引き金になることがあります。それにより、リフローに要する時間がさらに長くなります。
> * アニメーションなどの複雑なレンダリングの変更は、フローの外で行うようにします。これは「position: absolute」や「position: fixed」を使用することで実現できます。
>
> [Google「ブラウザのリフローを最小限にする」より一部抜粋引用](https://developers.google.com/speed/docs/insights/browser-reflow?hl=ja#header)

引用文からの捕捉になりますが、アニメーションでリフローが発生しそうな要素を `position: absolute` で浮動化させておくと、リフローの影響をその要素のみに限定させ、周囲（外）の要素には影響を与えません。通常では要素の `width` などが変更されれば、その親要素の幅や子要素の幅にも影響を与えるので、自身以外もリフロー（再計算）の対象になってしまいます。

他下記ページでもリフローやリペイントを押さえた実装方法を紹介されています。  
=> [レンダリングを意識したパフォーマンスチューニング](https://www.slideshare.net/hayatomizuno/ss-23379553)

### アニメーションで幾つかプロパティーを変更する際、GPU 処理されるプロパティーも活用する

`transform` や `opacity` はアニメーション時に GPU レイヤーにて処理されます。  
例えばアニメーションの際 `right, left, top, bottom` の値変更の他に `opacity` の値変更も加える事で、`right, left, top, bottom` の処理も GPU 処理に含まれリペイントの発生を無くすことができます。  

実現したい表示によっても難しいかもしれませんが、高コストなプロパティーの変更がある際 GPU 処理されるプロパティーも一緒に変更する事で、処理を GPU に任せる事ができます。

## 参考資料まとめ

* [Google_Accelerated Rendering in Chrome](https://www.html5rocks.com/ja/tutorials/speed/layers/)
* [Google_レンダリング パフォーマンス](https://developers.google.com/web/fundamentals/performance/rendering/?hl=ja)
* [Google_ブラウザのリフローを最小限にする](https://developers.google.com/speed/docs/insights/browser-reflow?hl=ja)
* [ブラウザにやさしいHTML/CSS](https://www.slideshare.net/TakeharuIgari/htmlcss-34506501)
* [ブラウザレンダリングの仕組み](https://student-engineer.net/blowser-rendering/)
* [hatena_ブラウザのレンダリングの仕組み](http://cidermitaina.hatenablog.com/entry/2019/03/03/232516)
* [wpj_60fpsを実現するベストプラクティス](https://www.webprofessional.jp/achieve-60-fps-mobile-animations-with-css3/)
* [Dev.Opera_CSS will-changeプロパティについて知っておくべきこと](https://dev.opera.com/articles/ja/css-will-change-property/)
* [Qiita_DevToolsのTimelineパネルを見ながら、レンダリングの仕組みを理解する](https://qiita.com/cy-mitsuki/items/51a0a4c17b89154a7af2)
* [Qiita_will-change指定時の挙動, パフォーマンスへの影響と考察](https://qiita.com/damele0n/items/71352757d0e6fdf5b184)
* [Qiita_スクロールが軽快に！will-change属性をつけるだけでFPSが...](https://qiita.com/ttiger55/items/b2423cb72668c3c98d89)
* [Qiita_Reflowを制するものはDOMを制す](https://qiita.com/jkr_2255/items/5cdead4ee7fa289bfeed)
* [Fix scrolling performance with CSS will-change property](https://www.fourkitchens.com/blog/article/fix-scrolling-performance-css-will-change-property/)
* [github_Consider adding 'will-change: transform' even if translate3d if disabled](https://github.com/Microsoft/monaco-editor/issues/426#issuecomment-308395469)
* [パフォーマンス改善を行うためにGPU処理を取り入れてみた](http://un-tech.jp/performance-gpu/)
* [レンダリングを意識したパフォーマンスチューニング](https://www.slideshare.net/hayatomizuno/ss-23379553)
