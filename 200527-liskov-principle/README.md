# リスコフの置換原則（LSP）をしっかり理解する

SOLID 原則の１つ「リスコフの置換原則」についての記事になります。  
この原則に関する幾つかの記事を眺めてみても、どうも他４つの原則と比べて腹落ちしない部分があったので理解の為しっかり調べてみました。

> S が T の派生型であれば、プログラム内で T 型のオブジェクトが使われている箇所は全て S 型のオブジェクトで置換可能  
> ([[1] wikipedia](https://ja.wikipedia.org/wiki/%E3%83%AA%E3%82%B9%E3%82%B3%E3%83%95%E3%81%AE%E7%BD%AE%E6%8F%9B%E5%8E%9F%E5%89%87) より引用）

リスコフの置換原則は上記のようにシンプルに説明されている事が多いですが、基底型と置換可能な派生型となるには幾つか遵守すべきルールがあります。この原則は Barbara Liskov 氏が 『A Behavioral Notion of Subtyping』([[9]](https://www.cs.cmu.edu/~wing/publications/LiskovWing94.pdf)) という論文で提唱した内容であり、後に「リスコフの置換原則」として認知されるようになりました。
元々の論文名から分かるように、その本質は「サブタイプ（派生型）の振る舞いの概念」について説明しています。つまり「サブタイプの振る舞いはこうあるべきだ」という１つの指針がリスコフの置換原則と言えます。

## 関連ワードについて説明

リスコフの置換原則で重要となるワードについてそれぞれ説明します。  
また、以降引用以外の文中のワードは次のように定義します。

* 基底型 / 基底クラス => スーパータイプ / スーパークラス
* 派生型 / 派生クラス => サブタイプ / サブクラス

### サブタイプ

> コンピュータサイエンスにおいて、データ型 S が他のデータ型 T と is-a 関係にあるとき、S を T の派生型（はせいがた、subtype）であるという。また T は S の基本型（きほんがた、supertype）であるという。  
（[[11] wikipedia](https://ja.wikipedia.org/wiki/%E6%B4%BE%E7%94%9F%E5%9E%8B) より引用）

型 T に対して is-a 関係である型 S は T のサブタイプと言えます。反対に T は S のスーパータイプとなります。
※ 引用文中の「基本型」は「基底型」と同一の意味合いです。

### is-a 関係

is-a 関係とは「B is a A」（B は A の一種である）と言う事です。  
オブジェクト指向ではクラス継承の概念として主に使用されます。  
通常継承を使用する事でスーパークラス(A)の持っているメソッドやプロパティーを全て引き継いだサブクラス(B)が生成できます。この時 B は A の全てを引き継いでいるので、「B は A の一種である」（is-a関係）が成り立つと言えます。  

ただ、全てのスーパークラスとサブクラスの間で is-a の関係が成り立つとも限りません。  
「B は A の一種である」為には B の振る舞いが A に期待されるものと同一である必要があります。  
これは A を継承した結果 B で A と同じようには振る舞えないのでは「B は A の一種である」とは言えないからです。

### 補足

「タイプ」や「クラス」と言う単語の区別がややこしいので補足します。  
スーパータイプ・サブタイプは実装の概念を含みません。あくまでインターフェイス（型）のみです。  
スーパークラス・サブクラスはインターフェイスと実装を含みます。([[17]](https://kkanda.hatenadiary.org/entry/20040830/p1) 参考)  

* ※ スーパータイプ・サブタイプ = インターフェイス
* ※ スーパークラス・サブクラス ＝ インターフェイス + 実装

サブタイプの説明では実装の概念を含まないため、クラスというワードが使用されていません。  
下記説明をクラス間の説明へ置き換えると「サブクラスの型(S)がスーパークラスの型(T)と is-a 関係にある時」となります。

> データ型 S が他のデータ型 T と is-a 関係にあるとき

## リスコフの置換原則が提唱するサブタイプの振る舞い

リスコフの置換原則が提唱するサブタイプにおける振る舞いの規範は、下記大きく分けて２項目の各ルールを守る事で達成する事ができます。
（参考 [[10]](http://reports-archive.adm.cs.cmu.edu/anon/1999/CMU-CS-99-156.pdf), [[11]](https://stackoverflow.com/questions/56860/what-is-an-example-of-the-liskov-substitution-principle/38933690#38933690), [[12]](https://softwareengineering.stackexchange.com/questions/170189/how-to-verify-the-liskov-substitution-principle-in-an-inheritance-hierarchy#comment326550_170191), [[13]](http://www.blackwasp.co.uk/LSP.aspx))

### 1. スーパータイプメソッドの振る舞い保持

#### シグネチャ（インターフェイス）ルール

* 引数はスーパータイプと同一の数です。引数の型はスーパータイプと同一か、それより制限の少ない型（反変性）を受け取る必要があります。
* 返り値の型はスーパータイプと同一か、それよりも制限が強い型（共変性）を返す必要があります。
* スーパータイプと同一型の例外、もしくはその例外のサブタイプのみを返す必要があります。

反変性、共変性については [[14]](https://qiita.com/7shi/items/39a222b5928bf0b6f745), [[15]](https://www.php.net/manual/ja/language.oop5.variance.php) ,[[16]](https://www.phper.ninja/entry/2020/03/01/054833) が参考になります。

#### 事前条件と事後条件のルール

* 事前条件はスーパータイプと同一か、それよりも弱める事ができます。反対に条件を強める事はできません。
* 事後条件はスーパータイプと同一か、それよりも強める事ができます。反対に条件を弱める事はできません。

事前条件と事後条件は Bertrand Meyer 氏の「契約による設計（DbC）」でも使用されており、Barbara Liskov 氏も [[9]](https://www.cs.cmu.edu/~wing/publications/LiskovWing94.pdf) の論文の中で同一のルールであると述べています。事前条件と事後条件の詳細に関しては [[2]](https://qiita.com/hiko1129/items/9b3066feffabccf83c16), [[8]](https://qiita.com/Kokudori/items/2e4bd32abf7abea3186f) でわかりやすく説明されています。

### 2. スーパータイプのプロパティーの保持

#### 不変条件のルール

* スーパータイプで常に満たされていたプロパティーの不変条件は保持する必要があります。たとえば、スーパータイプでプロパティー A の値は B の値を超えないという条件があった場合に、サブタイプもこれを保持する必要があります。

#### 履歴（制約）のルール

* スーパータイプから新規追加またはオーバーライドされたメソッドは、スーパータイプで許可されていない方法でプロパティー値を変更してはならない。たとえば、スーパータイプで常に固定値であると確約されているプロパティー A に対して、サブタイプでこれを変更する事はできません。

## サブタイプになり得るのは継承によるサブクラスだけでは無い

> 構造型のオブジェクト指向言語では、型 A のオブジェクトが型 B のオブジェクトの処理できるメッセージすべてを処理できるなら（言い換えると、同じメソッドを実装していれば）、継承関係に関係なく、A は B の派生型となる。  
([[19] wikipedia](https://ja.wikipedia.org/wiki/%E6%B4%BE%E7%94%9F%E5%9E%8B) より引用)
>
> OCamlは、クラスや継承・インタフェースなどを宣言しなくてもオブジェクトやメソッドの型を自動で推論する。部分型関係（いわゆるis-a関係の一種）も、実際のオブジェクトの型にしたがいあらかじめ宣言をしなくても成立する。これを構造的部分型（structural subtyping）という  
([[20] xtechの記事](https://xtech.nikkei.com/it/article/COLUMN/20061107/252787/) より引用)

上記は構造型(structural subtyping)オブジェクト指向言語の場合には継承を用いなくても、スーパータイプと同一の[シグネチャ](http://e-words.jp/w/%E3%82%B7%E3%82%B0%E3%83%8D%E3%83%81%E3%83%A3.html#Section_%25E3%2583%2597%25E3%2583%25AD%25E3%2582%25B0%25E3%2583%25A9%25E3%2583%259F%25E3%2583%25B3%25E3%2582%25B0%25E3%2581%25AB%25E3%2581%258A%25E3%2581%2591%25E3%2582%258B%25E3%2582%25B7%25E3%2582%25B0%25E3%2583%258D%25E3%2583%2581%25E3%2583%25A3)を持ち、置換可能なオブジェクトはサブタイプである事を説明しています。つまり、is-a 関係はクラス継承以外でも成り立ちます。is-a 関係がクラス継承の概念として説明されるのは [[21]](https://think-on-object.blogspot.com/2011/11/is-ahas-is-ahas-top-is-a-is-b.html#isa) でも述べられている通り、大抵都合が良いからです。

また、サブタイプの定義ではあくまで型同士のみが is-a の関係であることを示唆しており、実装は関係ないためスーパータイプに期待される振る舞いとシグネチャを引き継いでいれば、それはサブタイプと言えます。従って、実装継承（extends）ではなくインターフェイス実装（implements）の間でもサブタイプは成り立つと言えます。「振る舞い = 契約（事前条件や事後条件、不変条件）」であり、インターフェイスで予めそれらを取り決めて（設計して）おき、後はそれを遵守して実装するという考え方になります。

## リスコフの置換原則は何を示してくれるのか

正しい is-a 関係を示す為の原則であり、それは継承すべきかどうかの判断指針になります。この原則に照らし合わせて is-a 関係が成り立たない場合、他のアプローチを模索してより良い設計に導く事ができます。たとえば、下記のような選択肢も取ることができます。([[22]](http://stefan-mehnert.de/2016/04/23/liskov-substitution-principle/), [[23]](https://ikenox.info/blog/inheritance-and-delegation-and-interface/) 参考)

* 両者の関係性を見直し、共通している部分をインターフェイス又は抽象クラス化して、より適切なスーパータイプとサブタイプの関係性を築く。
* is-a 関係を解消し、has-a 関係であるコンポジション（委譲）に切り替える。

## リスコフの置換原則に違反してはいけない理由

この原則に違反することは、SOLID 原則の１つ「オープン・クロースドの原則（OCP）」の違反にも繋がってしまうためです。下記に一例としてリスコフ置換の原則（LSP）違反のコードを記載してみました。スーパータイプ（Player）に対して、サブタイプ（HlsPlayer, DashPlayer）は LSP に違反しています。

```js
// スーパータイプ
class Player {
    constructor() { /*...*/ }
    /**
     * プレイヤーの停止処理
     */
    function pause() {
        /*...*/
    }
};

// Player のサブタイプ
class HlsPlayer extends player {
    constructor() { /*...*/ }
    /**
     * pause 実行前に必要（スーパータイプには無いメソッド）
     */
    function reservePause() {
        /*...*/
    }
};

// Player のサブタイプ
class DashPlayer extends player {
    /* hlsPlayer と同じく reservePause を実装している */
};

const playerFactory = (option) => {
    switch (option.mediaName) {
        case 'hls':
            return new HlsPlayer(option);
        case 'dash':
            return new DashPlayer(opiton);
        default:
            return new Player(option);
    }
};

class Controller {
    constructor(player) {
        this.player = player;
    }
    function start() { /*...*/ }
    function end() {
        const mediaName = this.player.media;
        // playerFactory で受け取る Player を出し分けているのにスーパータイプとサブタイプで振る舞いが
// 違うため受け取り先（Controller内）でも出し分け処理が必要になってしまっている。
        // また新しいサブタイプの Player が作られた時に、下記 if 文に追加する必要が出てきそう。
        // 追加に対して修正が受け取り先に及んでおり、 OCP に違反している
        if (mediaName === 'hls' && mediaName === 'dash') {
            this.player.reservePause();
        }
        // Controller が受け取った Player がサブタイプの場合、スーパータイプと比べて pause を
// 実行するための事前条件が強くなってしまっている。（pause の前に reservePause が実行されている必要がある）
        // LSP に違反している
        this.player.pause();
    }
};

const player = playerFactory(option);
const controller = new Controller(player);
controller.start();
controller.end();
```

LSP に違反しないよう、特に継承する際は心がけましょう。

## 参考資料

* [[1] wikipedia: リスコフの置換原則について](https://ja.wikipedia.org/wiki/%E3%83%AA%E3%82%B9%E3%82%B3%E3%83%95%E3%81%AE%E7%BD%AE%E6%8F%9B%E5%8E%9F%E5%89%87)
* [[2] Qiita: リスコフの置換原則（LSP）と契約による設計（DbC）の関連について](https://qiita.com/hiko1129/items/9b3066feffabccf83c16)
* [[3] w3ki: リスコフの置換原則](https://www.ja.w3ki.com/dip/liskov_substitution_principle.html)
* [[4] togetter: 放送大学のプログラミングの授業とリスコフ置換原則](https://togetter.com/li/778143?page=2)
* [[5] togetter: リスコフ置換原則への取り組み](https://togetter.com/li/1405372)
* [[6] scrapbox: リスコフの置換原則](https://scrapbox.io/haraheniku/%E3%83%AA%E3%82%B9%E3%82%B3%E3%83%95%E3%81%AE%E7%BD%AE%E6%8F%9B%E5%8E%9F%E5%89%87)
* [[7] The Liskov Substitution Principle (LSP)リスコフの置換原則](https://blog1.mammb.com/entry/20090923/1253689152)
* [[8] Qiita: 契約による設計から見た例外](https://qiita.com/Kokudori/items/2e4bd32abf7abea3186f)
* [[9] cmu: Behavioral Notion of Subtyping（リスコフ置換原則の元論文）](https://www.cs.cmu.edu/~wing/publications/LiskovWing94.pdf)
* [[10] cmu: Barbara H. Liskov, Behavioral Subtyping Using Invariants and Constraints](http://reports-archive.adm.cs.cmu.edu/anon/1999/CMU-CS-99-156.pdf)
* [[11] stackoverflow: liskov substitution principle answer 38933690](https://stackoverflow.com/questions/56860/what-is-an-example-of-the-liskov-substitution-principle/38933690#38933690)
* [[12] StackExchange: How to verify the Liskov substitution principle](https://softwareengineering.stackexchange.com/questions/170189/how-to-verify-the-liskov-substitution-principle-in-an-inheritance-hierarchy#comment326550_170191)
* [[13] blackwasp: Liskov Substitution Principle](http://www.blackwasp.co.uk/LSP.aspx)
* [[14] Qiita: 共変戻り値と反変引数](https://qiita.com/7shi/items/39a222b5928bf0b6f745)
* [[15] php manual: 共変性と反変性](https://www.php.net/manual/ja/language.oop5.variance.php)
* [[16] hatena: 共変性(covariance)と反変性(contravariance)](https://www.phper.ninja/entry/2020/03/01/054833)
* [[17] hatena: subtypingとsubclassingの違い](https://kkanda.hatenadiary.org/entry/20040830/p1)
* [[18] stackoverflow: Mutable class as a child of an immutable class](https://stackoverflow.com/questions/2493891/mutable-class-as-a-child-of-an-immutable-class)
* [[19] wikipedia: 派生型](https://ja.wikipedia.org/wiki/%E6%B4%BE%E7%94%9F%E5%9E%8B)
* [[20] xtech: 第4回 関数型言語とオブジェクト指向，およびOCamlの"O"について](https://xtech.nikkei.com/it/article/COLUMN/20061107/252787/)
* [[21] is-a 関係と has-a 関係](https://think-on-object.blogspot.com/2011/11/is-ahas-is-ahas-top-is-a-is-b.html#isa)
* [[22] Liskov substitution principle violation: square extends rectangle](http://stefan-mehnert.de/2016/04/23/liskov-substitution-principle/)
* [[23] 継承と委譲の使い分けと、インターフェースの重要性について](https://ikenox.info/blog/inheritance-and-delegation-and-interface/)
* [[24] Qiita: 異なる2つの型システム「公称型」と「構造的部分型」](https://qiita.com/suin/items/52cf80021361168f6b0e)
