# コンポーネント指向を CSS 設計へ取り入れよう

## 概要

この記事では FLOCSS という CSS 設計をベースに、その一部だけを掻い摘んだミニマムな設計を紹介します。  
FLOCSS を適応する程の大規模なページ構成でも無いけど、その考えを活かしたい・取り入れたい場合に有効かと思います。  
既に BEM という設計手法を知っている方は FLOCSS について知らなくても理解できる内容になっていると思います。  
概要の章では FLOCSS について簡単に説明しているので、もう知ってるよ！という方は読み飛ばして頂いて結構です。

### FLOCSS について

基本的には BEM（MindBEMding）を拡張したような CSS 設計になります。  
これまでの BEM の命名に加えて prefix（接頭辞）が付与され、さらに役割が明確になります。
FLOCSS の語源など詳しい説明は [こちら](https://github.com/hiloki/flocss) を参照してください。  
下記が簡単な prefix の役割説明になります。

* `l-`： layout 要素  
  * ページを構成するヘッダーやメインのコンテンツエリア、サイドバーやフッターといった  
プロジェクト共通のコンテナーブロックのスタイルを定義します。
* `p-`： project 要素  
  * プロジェクト固有のパターンであり、いくつかのComponentと、それに該当しない要素によって構成されるものを定義します。
* `c-`： component 要素
  * 再利用できるパターンとして、小さな単位のモジュールを定義します。
* `u-`： utility 要素  
  * ComponentとProjectレイヤーのObjectのモディファイアで解決することが難しい・適切では無い、  
  わずかなスタイルの調整のための便利クラスなどを定義します。

## 前提（本題）

本稿の FLOCSS ベースのミニマム設計では下記 prefix（接頭辞）は使用しません。  

* `l-`： layout要素
* `p-`： project要素

### FLOCSS の Component（c-）と Utility（u-）のみを残したミニマム設計

FLOCSS という設計に置いて私は Component と Project を分けるという点に魅力を感じました。  
BEM では Block が Component 要素にあたるのかも知れませんが、全ての Block が再利用される訳ではありません。  
私が本稿の設計で重きを置いているのは、あくまで命名による Component 要素の区別と再利用にあります。  
Utility については深く触れませんが、補助として使用する程度であれば利便性もあるので取り入れても良いかなといった印象を持っています。

### この CSS 設計の適応を想定される場面

簡単な例ですが、下記項目のようなページを作る際に有効な設計です。  
下記は結構都合の良い例ですが、要は複数ページで共通の UI があれば導入可能です。  
ただ、あまりにも規模が大きい場合もっと厳格である本来の FLOCSS を適応するなりした方が設計として堅牢かなとは思います。

* __a, b, c__ の３つのページを作る必要がある
* __a, b, c__ ページはそれぞれ同じではないもののタイプ違いのようなもので、テーマや目的が同じなので共通の UI が多い

### Component の粒度について

これは本来の FLOCSS でも同じですが、CSS 設計を適応する範囲（規模）によります。  
あくまで想定ですが、下記のようになるのでは無いかと考えます。

* （例１）同じドメインを持つ全ページの場合

    ```html
    <!-- どのページでも汎用的に使用できるよう細かく Component が細分化されている -->
    <div class="c-miniBox">
        <p class="c-normalText">名前</p>
        <div class="c-button c-button--flat">
            <span class="c-buttonText">投票</span>
        </div>
    </div>
    ```

* （例２）何か同じ目的を持った複数のページ・同じテーマを持つタイプ違いのページの場合

    ```html
    <!-- 複数のページ表示（見た目）は似ており、パーツも割と共通なため Componen の粒度が大きい -->
    <div class="c-miniBox">
        <p class="c-miniBox__name">名前</p>
       <div class="c-miniBox__button">
            <span class="c-miniBox__buttonText">投票</span>
        </div>
    </div>
    ```

### 本稿で使用される用語説明

もしかしたら勘違いしてしまいそうな用語のみ補足で書いておきます。

* __Component__
  * BEM の B（Block）に接頭辞 c- がついたものを表しています。
* __component.css__
  * Component のスタイルの記載する css であり、全ページで共通して読み込む css です。  
Component に直接スタイルを記載する行為は component.css に直接記載する行為にあたります。
* __page.css__
  * そのページ固有のスタイルを記載する css であり、各々のページで読み込む css です。


## コーディングルールについて

FLOCSS の一部を取り入れたミニマム設計と Component 指向を元にコーディングを行っていく上で取り決めたルールを紹介します。

### 1. Component はなるべく汎用性を意識して命名する

```html
<!-- BAD -->
<div class="c-superHeroDesc">
    <dl class="c-superHeroDesc__list">
        <div class="c-superHeroDesc__group">
            ...
        </div>
    </dl>
</div>
```

```html
<!-- GOOD -->
<div class="c-packageDesc">
    <dl class="c-packageDesc__list">
        <div class="c-packageDesc__group">
            ...
        </div>
    </dl>
</div>
```

Component は複数のページや、繰り返し使われる想定であるのが前提です。  
BAD な例の命名では「スーパーヒーローの説明」という命名ですので、「スーパーヒーロー」以外の情報が入ってきた時に命名と中身の情報が全く合わないものになってしまいます。割とどんな情報が入ってきても良いように抽象度を高めた命名にしましょう。

### 2. Component には margin / width を直接付与しない

component.css（ 複数ページ共通で使用する css ）

```css
.c-hoge {
    display: flex;
    font-size: 14px;
    padding: 16px;
    color: #333;
    /* width: 800px; 付与しない */
    /* margin-bottom: 30px; 付与しない */
}
```

```html
<div class="c-hoge">
    <div class="c-hoge__imgWrap">
        <img class="c-hoge__img" src="" alt="">
    </div>
    <ul class="c-hoge__items">
        <li class="c-hoge__item">アイテム１</li>
        <li class="c-hoge__item">アイテム２</li>
        <li class="c-hoge__item">アイテム３</li>
    </ul>
</div>
```

Component に予め margin や width などのスタイルが固定値で付与されていると汎用的に使い回しづらくなります。  
margin が付与されていると Component のスタイルと html をコピペで何処からか持ってきた際、それは何らかの要素との間に意図しない余白を生じさせます。それは事実上 Component によって Component 外へ影響を与えてしまっています。  

width に関しては margin ほど Component 自身が外へ与える影響は少ないですが、各ページによっては html に入ってくる内容量などが違ったりすることは往々に有ります。Component に default で width（固定値）を設定しておいても適切に上書きすれば問題有りませんが、width: auto（width 記載無し）にしておけば div, ul, p などの要素は、その外側の要素の幅に依存するので Component 側で余計に width を設定したりせずに済む場合があります。

### 3. Component への margin / width 付与は Component では無い親要素を通して付与する

page.css（ １枚のページで使用する css ）

```css
.content .c-hoge {
    width: 800px;
    margin-bottom: 30px;
}
```

index.html

```html
<section class="content">
    <h1 class="content__title">Hoge Component</h1>
    <div class="c-hoge">
        ...
    </div>
</section>
```

直接 Component に margin/width を付与できないとすれば、Component 間の margin 設定ならびに width はどうすれば良いでしょう。ページレイアウトを組むには margin は必須になります。  

解決策として、Component 要素では無い親要素を通して Component に margin を付与するようにします。そのページ固有のスタイルを書く CSS と Component のスタイルを書く CSS は分けておく必要が有ります。component.css（Component のスタイルを記述） は複数のページで共通して使用されるため、margin を直接付与してしまうと複数ページ各々で margin 調整したいのに、最初から余計な margin がついてしまいます。  

page.css（ページ固有のスタイルを記述）で Component では無い親要素を通して Component に margin を設定することで、そのページに適した margin を Component に設定することができます。

### 4. あるページで Component 内部に少し変更を加えたい場合は、Component を直接変更しない

component.css

```css
/* 変更しない */
.c-hoge__imgWrap {
    width: 300px;
    margin-right: 30px;
}
.c-hoge__item {
    margin-bottom: 15px;
}
```

page.css

```css
/* Component のスタイルを上書き */
.content .c-hoge__imgWrap {
    width: 400px;
}
/* Component のスタイルを上書き */
.content .c-hoge__item {
    margin-bottom: 10px;
}
```

index.html

```html
<section class="content">
    <h1 class="content__title">Hoge Component</h1>
    <div class="c-hoge">
        <div class="c-hoge__imgWrap">
            <img class="c-hoge__img" src="" alt="">
        </div>
        <ul class="c-hoge__items">
            <li class="c-hoge__item">アイテム１</li>
            <li class="c-hoge__item">アイテム２</li>
            <li class="c-hoge__item">アイテム３</li>
        </ul>
    </div>
</section>
```

Component（component.css) は複数ページ共通で読み込むので、あるページで少し見た目を変更したいがために変更を加えてしまうと他のページで同じ Component が使用されていた場合に影響を及ぼしてしまいます。  
Component に margin/width を設定するのと同じように page.css（ページ固有のスタイルを記述）で Component では無い親要素を通してスタイルを上書きましょう。

※ もし Component の見た目が大きく変わるような変更の場合、新しく Component 定義しましょう。  
※ 一部他のページでも同じような変更がある場合、予測できる場合は Component に Modifire を定義しましょう。

### 5. Section 要素は Component には含めない

index.html（ BAD な Component 例 ）

```html
<!-- 背景色（透明）が見た目上青色に染まってしまう -->

<section class="c-fuga"> <!-- 背景色（青） -->
    <h1 class="c-fuga__title">見出し</h1>
    <div class="c-fuga__content">
        ...
    </div>
    <section class="c-piyo"> <!-- 背景色（透明）-->
         <h1 class="c-piyo__title">小見出し</h1>
         ...
    </section>
</section>
```

index.html（ GOOD な Component 例 ）

```html
<!-- 背景色（青）と（透明）が見た目上別れつつ理想のアウトラインを保っている -->

<section class="content">
    <div class="c-fuga"> <!-- 背景色（青） -->
        <h1 class="c-fuga__title">見出し</h1>
        <div class="c-fuga__content">
            ...
        </div>
    </div>
    <section class="content content--sub">
        <div class="c-piyo"> <!-- 背景色（透明）-->
            <h1 class="c-piyo__title">小見出し</h1>
            ...
         </div>
    </section>
</section>
```

アウトライン（文章の階層構造・見出し階層）は作成するページごとに柔軟に決定することができる状態が望ましい。  
また、コンポーネントごとに背景色を持っている場合、Component が section 要素になっていると場合によってはスタイルの再現が難しくなってしまう。  
具体的には、ページのアウトラインを構成する上で、「見出しA」に対して「小見出しA」を作りたい場合、「見出しA」の section の中に「小見出しA」の section をネストさせる必要が出てくる。この時に Component（section） に背景色を指定していると、Component ごとに背景色を分けるといった表示の実現は難しくなる。

### 6. font-size や color などの親要素から引き継がれるスタイルは、なるべく親要素に記載する

```css
.c-fuga {
    ...
    color: #fff;
    font-size: 14px;
    text-align: center;
    background-color: #999;
}
```

```css
/* BAD */
.c-fuga:hover {
    background-color: #333;
}
.c-fuga:hover .c-fuga__text {
    color: #ff0;
}
```

```css
/* GOOD */
.c-fuga:hover {
    color: #ff0;
    background-color: #333;
}
```

これは正直 Component の見た目にもよります。子要素でなく親要素にスタイルを設定しても問題無いような場合、なるべく親要素にスタイルを記載するのが良いと自分は考えます。  

例えば要素を hover した際にテキストの色を変更する事は良くあります。多くの場合はそのテキストを hover した時よりも、そのテキストを含む要素を hover した際に色を変更する場合が多いかと思います。このような場合、子要素（テキスト）に直接 color を設定していると BAD な例のように hover 時のスタイル指定が別れてしまう場合があります。親要素に color を設定しておけば、それを引き継ぐ子要素も hover した際に同じ色に変化してくれます（GOOD 例）

また、Component に margin を付与する際にまとめて font-size, color, text-align などの変更も親要素だけで済む場合があるので、この点がメリットになります。

### 7. Utility は適度に利用

```css
.u-red {
    color: #f00;
}
.u-link {
    color: #00f;
    text-decoration: underline;
}
```

```html
<div class="caution">
    <p class="caution__text">
        明日の天気が<span class="u-red">晴れ</span>であればイベントを開催します。
        <br><span class="u-red">雨</span>であればイベントは中止になります。<a href=""
        class="u-link">詳しくはこちらをご覧ください。</a>
    </p>
</div>
```

例えば ○○ のテキストの色を赤色にして目立たせてくれ！だったり、ここはリンクにしてほしい、入れてほしいといった事は良くある事かと思います。こう言った軽微な変更を予め Component に含めておくのは面倒だし、含めておいても使用されない場合があります。このような時にいくつか Utility 化しておくと柔軟に変更でき利便性があります。

## 最後に

結局設計なんてページ構成や規模によって最適なものは変わります。単発のLPページであれば通常のBEMでも十分です。
あくまで今回紹介した設計は独自の考えによるものですので、参考程度にして頂ければと思います。
