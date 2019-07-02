# Web Performance Memo

フロント側でできる web site の performance に関する施策メモ。  
web site の表示速度を改善する方法として、下記のようなファイル自体を圧縮したり変換させたりといった方法を除いて、コードによる performance へのアプローチ方法をまとめておく。

* 画像、HTML/CSS/JavaScript の圧縮
* HTTPリクエストの削減
  * スタイルのインライン記述
  * 画像の埋め込み（bese64 化）
  * 画像のスプライト化

## CSS

### Reflow / Repaint を避ける

ブラウザーが要素をレンダリング（描画）する際には通常下記４つの工程が発生します。

1. Style（スタイルの適応）
2. Layout（スタイルの計算と要素の再配置）
3. Paint（ピクセルの書き込み処理 = 要素の視覚的な部分すべての描画）
4. Composite（）

レンダリングされた要素に対してアニメーションが行われると、同じように

#### 参考資料（Reflow / Repaint を避ける）

* [Qiita_Reflowを制するものはDOMを制す](https://qiita.com/jkr_2255/items/5cdead4ee7fa289bfeed)
* [Google_ブラウザのリフローを最小限にする](https://developers.google.com/speed/docs/insights/browser-reflow?hl=ja)
* [Google_レンダリング パフォーマンス](https://developers.google.com/web/fundamentals/performance/rendering/?hl=ja)
* [パフォーマンス改善を行うためにGPU処理を取り入れてみた](http://un-tech.jp/performance-gpu/)
* [レンダリングを意識したパフォーマンスチューニング](https://www.slideshare.net/hayatomizuno/ss-23379553)

### will-change によるアニメーションの最適化（GPUレイヤー）

description

#### 参考資料（will-change によるアニメーションの最適化）

* [wpj_60fpsを実現するベストプラクティス](https://www.webprofessional.jp/achieve-60-fps-mobile-animations-with-css3/)
* [Dev.Opera_CSS will-changeプロパティについて知っておくべきこと](https://dev.opera.com/articles/ja/css-will-change-property/)
* [Qiita_スクロールが軽快に！will-change属性をつけるだけでFPSが...](https://qiita.com/ttiger55/items/b2423cb72668c3c98d89)
* [Fix scrolling performance with CSS will-change property](https://www.fourkitchens.com/blog/article/fix-scrolling-performance-css-will-change-property/)
* [github_Consider adding 'will-change: transform' even if translate3d if disabled](https://github.com/Microsoft/monaco-editor/issues/426#issuecomment-308395469)

### セレクタの書き方が及ぼす performance への影響

description

#### 参考資料（セレクタの書き方が及ぼす performance への影響）

* [Qiita_サイトの表示速度を意識したセレクタの書き方](https://qiita.com/mamiyan/items/778183160e9e58546824)
* [Coliss_プロパティやセレクタがパフォーマンスに与える影響](https://coliss.com/articles/build-websites/operation/css/things-nobody-ever-taught-me-about-css.html)

### アニメーションはなるべく JS ではなく CSS で行う

description  

#### 参考資料（アニメーションはなるべく JS ではなく CSS で行う）

* [スムーズなアニメーションを実装するコツと仕組みを説明...](http://ginpen.com/2013/12/06/hardware-acceleration/)
* [命令的アニメーション vs 宣言的アニメーション](https://www.html5rocks.com/ja/tutorials/speed/high-performance-animations/#toc-imperative-declarative)

## JavaScript

### setTimeout

description

### 参考資料（setTimeout）

* [Qiita_setTimeout(...,0)などの使いドコロ](https://qiita.com/jkr_2255/items/17693ab77beea71a871c)
* [Qiita_JavaScriptの非同期処理を並列処理と勘違いしていませんか？](https://qiita.com/klme_u6/items/ea155f82cbe44d6f5d88)

## HTML

### Resource Hints API

description

#### 参考資料（Resource Hints API）

* [Members_Webページに適した手法で表示高速化ができるResource Hints...](https://blog.members.co.jp/article/33474)
* [Qiita_Prerender APIでWebサイトを高速化](https://qiita.com/sueshin/items/202d82e08b9d051004fd)
* [Qiita_ロードを高速化するprefetch](https://qiita.com/ShinyaOkazawa/items/f95788c67114d0360e40)
* [ソーシャルメディアの読み込みはDNSプリフェッチのまとめ設定がお得](http://tokkono.cute.coocan.jp/blog/slow/index.php/programming/boostup-socials-with-dns-prefetch/)
* [Resource Hintsの対応をしてWebPageTestの点数を改善した](https://paranishian.hateblo.jp/entry/2018/11/02/183724)
* [DNS-Prefetch,Preconnect,Prefetch,Prerender,Preloadを試す](https://qwerty.work/blog/2019/02/dns-prefetch-preconnect-prefetch-prerender-preload.php)
* [MDN_X-DNS-Prefetch-Control](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/X-DNS-Prefetch-Control)
