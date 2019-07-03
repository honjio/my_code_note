# CSS performance memo

CSS で performance に関連する下記の項目についてまとめます。

* アニメーション
* セレクタ

## CSS Animation

### 要素の移動について

要素を縦横にアニメーションを伴って動かしたい場合 `transition` を適応させた要素に対して `right, left, top, bottom` や `transform: translate(数値)` のプロパティーを追加、またはその値を変更することで実現させることができる。

移動には `right, left, top, bottom` よりも `transform: translate(数値)` を使用した方が滑らかなアニメーションを実現することができます。

### なぜ transform: translate による移動の方が優れているか

それは `right, left, top, bottom` と `transform: translate` による要素の描画（レンダリング）方法が異なることに起因します。

簡単に説明すると `right, left, top, bottom` での移動は CPU で処理されます。  
`transform: translate` での移動は GPU で処理されます。  
単純な話グラフィック系の処理は CPU よりも GPU 側が得意なため描画処理がスムーズになります。

### CPU と GPU でのレンダリングの違い

描画済みの要素が再び何らかのアクションを起点に動く際、CPU と GPU 処理とでレンダリング方法が異なります。  

* CPU：  
  要素の位置が変更される都度 __ペイント（ピクセルの描画）処理__ が行われレンダリングされます。  
* GPU：  
__レイヤー__ という概念を用い、その __レイヤー__ を移動させ __合成（Composite）__ が行われレンダリングされます。

#### 要素のレンダリングについて

要素が描画（レンダリング）されるまで大まかに分けて下記４つの工程を通ります。

1. Style（スタイルの適応）
2. Layout（スタイルの計算および要素の配置）
3. Paint（ピクセルの描画）
4. Composite（レイヤーの合成 = 定着）

ただし、描画済みの要素のスタイルに後で何らかの変更が生じる際は必ずしも４つの工程は通りません。  
例えば要素に対して「文字色の変更」だけであれば特に要素に変形が生じたりする訳ではないので、Layout（スタイルの再計算および再配置）は行われません。  
１（Style） と ４（Composite） は必ず行われますが、２（Layout）と３（Paint）についてはスタイルの変更方法次第では省くことが可能です。Layout と Paint はコストのかかる処理なため、 __これを省くことでスムーズなアニメーションが実現可能__ となります。

#### レイヤーという概念について

> Webページは、ひとたびロードされ、パースされると、多くのWebデベロッパーにおなじみの構造、つまりDOMに変換されます。しかしながら、ページをレンダリングするに当たって、ブラウザはデベロッパーが直接のぞくことのできないいくつかの中間形式を持ちます。これらのうち最も重要なものはレイヤーです。  
[html5rocks.com 「レイヤーとは」より引用](https://www.html5rocks.com/ja/tutorials/speed/layers/)

下記CSSプロパティーなどの使用で要素がレイヤー化されるそうです。

* position
* transform
* opacity

Paint（ピクセルの描画）はレイヤー単位で行われる。  
これは再度描画が必要になった際、既に描画済みのレイヤーを再利用することで効率的にレンダリングを行うことができるためです。

#### レイヤーの種類について

レイヤーには大まかに分けて RenderLayer と GraphicLayer が存在する
> Chromeにおいては、実際のところ複数の異なるタイプのレイヤーが存在します。RenderレイヤーはDOMのサブツリーに関して責任を持ちます。また、GraphicsレイヤーはRenderレイヤーのサブツリーに関して責任を持ちます。後者は我々にとってより興味深いものとなります。なぜならGraphicsレイヤーはテクスチャとしてGPUにアップロードされるからです。以後、単に「レイヤー」と行った場合はこのGraphicsレイヤーを指します。  
[html5rocks.com 「レイヤーとは」より引用](https://www.html5rocks.com/ja/tutorials/speed/layers/)

この内、GraphicLayer は GPU で処理されるレイヤーであり、Chrome の DevTools 実際に確認することができます。CSS の performance のために説明される「レイヤー」とはこの GraphicLayer を指すことが殆どだと思います。  

### レイヤー（GPU）の生成について

一般的に GPU レイヤーを生成するには 3d 関連の CSS プロパティーを適応すると生成されると言われています。また描画処理を GPU に任せる行為は「ハードウェア・アクセラレーション」と呼ばれます。要素のアニメーションがカクツク場合に `transform: translateZ(0)` などの 3d 関連の CSS プロパティーをとりあえず当てて GPU で処理させる行為は CSS ハック的な方法として認知されています。

...

### 参考資料

* [Google_Accelerated Rendering in Chrome](https://www.html5rocks.com/ja/tutorials/speed/layers/)
* [Google_レンダリング パフォーマンス](https://developers.google.com/web/fundamentals/performance/rendering/?hl=ja)
* [ブラウザにやさしいHTML/CSS](https://www.slideshare.net/TakeharuIgari/htmlcss-34506501)
* [ブラウザレンダリングの仕組み](https://student-engineer.net/blowser-rendering/)
* [hatena_ブラウザのレンダリングの仕組み](http://cidermitaina.hatenablog.com/entry/2019/03/03/232516)
* ...
<!-- 
<img src="https://github.com/honjio/my-code-note/blob/master/css-performance-190704/reference-img/g-move-willchange.gif"> -->