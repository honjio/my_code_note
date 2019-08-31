# ES6 class 構文のパフォーマンスについて

## 概要

この記事は「class 構文」と「クロージャによるカプセル化」のパフォーマンス比較記事です。
検証ブラウザーは Chrome のみになります。

何か１つの処理を作るのに態々 class 化しなくても、クロージャ（関数）を作成しカプセル化するなどの方法もある。しかしパフォーマンスの観点で考えると、どちらで実装していくのが良いのだろうか...という疑問があった。本記事では、その検証結果をまとめる。  
検証前に「class 構文」と「クロージャによるカプセル化」に対する考えを述べる。

### class 構文に対する考え（検証前）

class 構文は実質 prototype の糖衣構文なので、同じようなオブジェクトを幾つも生成する場合は class の方が良さそうではある。しかし、class から instance を生成する new はそこそこのコストを孕んでいそうなので、オブジェクトの生成数が１〜２つ程度なら態々 class 構文で書かなくても良さそう。
また、プライベートなプロパティーを作成できない点が気に掛かる。

```js
// 例：value には外部からもアクセスできる
class Sample {
    constructor(val) {
        this.value = val;
    }
    getValue() {
        return this.value.toUpperCase();
    }
}
const test = new Sample('hello world');
test.getValue(); // HELLO WORLD
test.value // hello world（アクセスできちゃう）
```

### クロージャによるカプセル化に対する考え（検証前）

class で実現できないプロパティーのプライベート化（外部からアクセス不可）を実現することができる。class と異なり、実行の都度毎回新しいオブジェクト（prototypeではない）を返すので、オブジェクトの生成数が多ければ多いほどメモリの消費が気に掛かる。

```js
// 例：value には外部から直接アクセスできない getValue を通す必要がある
const sample = (val) => {
    const value = val;
    return {
        getValue() {
            return value.toUpperCase();
        }
    };
};
const test = sample('hello world');
test.getValue(); // HELLO WORLD
```

## パフォーマンス検証

「class 構文」と「クロージャによるカプセル化」の２通りについて同一の処理を用意し、「メモリの使用量」と「オブジェクト生成速度」を比較する。なお、下記コードが比較対象のコードである。

```js
class Shape {
    constructor(option) {
        this.shape = document.createElement('div');
        this._text = document.createTextNode(option.text);
        this._width = option.width;
        this._height = option.height;
        this._backgroundColor = option.backgroundColor;
        this.parent = option.parent;
    }
    add() {
        this.width().height().backgroundColor();
        this.shape.appendChild(this._text);
        this.parent.appendChild(this.shape);
    }
    text(t) {
        this._text.nodeValue = t;
        return this;
    }
    width(w = this._width) {
        if (w !== this._width) this._width = w;
        this.shape.style.width = `${this._width}px`;
        return this;
    }
    ...略（コードが長いため）
}
```

```js
const shape = (opiton) => {
    const el = document.createElement('div');
    const _text = document.createTextNode(option.text);
    const _width = option.width;
    const _height = option.height;
    const _backgroundColor = option.backgroundColor;
    const parent = option.parent;
    return {
        add() {
            this.width().height().backgroundColor();
            el.appendChild(_text);
            parent.appendChild(el);
        },
        text(t) {
            _text.nodeValue = t;
            return this;
        },
        width(w = _width) {
            if (w !== _width) _width = w;
            el.style.width = `${_width}px`;
            return this;
        },
        ...略（コードが長いため）
    }
};
```

### メモリの使用量とオブジェクト生成速度の計測

class `Shape` と関数 `shape` で生成したオブジェクトを `savingVariable` に for 文で任意数追加し終わる速度と、その時のメモリ使用量を計測する。  
メモリの使用量に関しては Chrome の「タスクマネージャ」の「JavaScript メモリ」の値を計測する（[タスクマネージャによるメモリ使用量のリアルタイム監視](https://developers.google.com/web/tools/chrome-devtools/memory-problems/?hl=ja#chrome_%E3%82%BF%E3%82%B9%E3%82%AF_%E3%83%9E%E3%83%8D%E3%83%BC%E3%82%B8%E3%83%A3%E3%81%AB%E3%82%88%E3%82%8B%E3%83%A1%E3%83%A2%E3%83%AA%E4%BD%BF%E7%94%A8%E9%87%8F%E3%81%AE%E3%83%AA%E3%82%A2%E3%83%AB%E3%82%BF%E3%82%A4%E3%83%A0%E7%9B%A3%E8%A6%96)）。

* 計測は片方ずつ行う。片方計測時はもう片方はコメントアウトしておく。
* 10 回ループと 100000 回ループで数値を取る

```js
// 作成したオブジェクトを保存する（メモリ使用量の計測用）
window.savingVariable = [];

console.time("no new");
for (let i = 0; i < 100000; i++) {
    savingVariable.push(shape(option));
}
console.timeEnd("no new");

console.time("new");
for (let i = 0; i < 100000; i++) {
    savingVariable.push(new Shape(option));
}
console.timeEnd("new");
```

### オブジェクト生成速度のみの計測

「メモリの使用量とオブジェクト生成速度の計測」では、メモリの圧迫による速度低下が考えられるので、こちらでは毎回作成したオブジェクトを永続的に参照可能な変数には保存しないようする（GC（メモリ解放）させるようにする）。  

※ なお、メモリの圧迫による速度低下は 10回ループ程度場合では考えられないため、こちらでは 100000 回ループのみ計測。

* 計測は片方ずつ行う。片方計測時はもう片方はコメントアウトしておく。
* 100000 回ループで数値を取る。

```js
console.time("no new");
for (let i = 0; i < 100000; i++) {
    const box = shape(option);
}
console.timeEnd("no new");

console.time("new");
for (let i = 0; i < 100000; i++) {
    const box = new Shape(option);
}
console.timeEnd("new");
```

## 検証結果

検証の結果は下記のようになった。

### メモリの使用量とオブジェクト生成速度の計測（結果）

|コード|ループ回数|メモリ使用量|速度（３回計測）|
|---|---|---|---|
|関数 `shape`  |10     |4.594MB  |0.198ms, 0.171ms, 0.175ms |
|class `Shape`|10     |4.590MB  |0.148ms, 0.172ms, 0.161ms|
|関数 `shape`  |100000 |62.518MB |288.227ms, 232.893ms, 244.277ms|
|class `Shape`|100000 |20.369MB  |107.515ms, 93.580ms, 131.524ms|

class `Shape` は関数 `shape` に比べて圧倒的にメモリ使用量が少ない。大体予測は付いていたが、実際数値で見ると prototype の使用は圧倒的に効率がいいなと再認識させられた。  
また、速度についてもメモリの圧迫に引っ張られてなのか class `Shape` の方が圧倒的に早い。  
10 回ループについては、大した差は出なかった。

### オブジェクト生成速度のみの計測（結果）

|コード|ループ回数|速度（３回計測）|
|---|---|---|
|関数 `shape`  |100000 |86.072ms, 83.144ms, 92.013ms|
|class `Shape`|100000 |79.072ms, 77.062ms, 85.856ms|

やはり、メモリ使用量の計測と同時に行った結果については速度に差が出たが、純粋にオブジェクト生成速度の計測のみを行うコチラでは大きく速度の差は開かなかった。  
その結果、new による instance の生成はコストというほどパフォーマンスに影響を与えるものではない事が分かった。

## 追加・補足検証

一番最初の例で出した class `Sample` と関数 `sample` についても計測してみた。  
理由としては、class `Shape` や関数 `shape` のように多くの機能を持ったオブジェクトの生成比較とは別に、小さなオブジェクト生成の比較結果を得るため。

## 追加・補足検証結果

|コード|ループ回数|メモリ使用量|速度（３回計測）|
|---|---|---|---|
|関数 `sample`  |10     |4.713MB  |0.198ms, 0.171ms, 0.175ms |
|class `Sample`|10     |4.713MB  |0.148ms, 0.172ms, 0.161ms|
|関数 `sample`  |100000 |22.388MB |7.521ms, 7.259ms, 6.287ms|
|class `Sample`|100000 |11.488MB  |10.986ms, 8.089ms, 14.462ms|

class `Sample` / 関数 `sample` 自体、メンバ変数 / 関数 の数が少ないので、100000 回 for 文で処理してもあまり差は無かった。メモリ使用量を見るとやはり class `Sample` の方が少なくはあるが、メモリの圧迫も然程無い為かそれによる速度差は特に見られなかった（誤差の範囲）。その為、class `Shape` / 関数 `shape` で行なった速度のみの計測は行わなかった。

結果としてやはりオブジェクトが大きいものであるほど、それが多数生成された際に class 構文による恩恵が大きく見られることが分かった。

## パフォーマンス検証を終えて

第一に class 構文優秀じゃん！と思いました。個人的に new ってコストのかかるイメージを固定概念的に持っていたので、それが払拭されたのが１つと、検証結果の数値を見るに prototype (class) を使用することの効率の良さ（無駄にメモリを喰わない）を改めて再認識できた点が大きいです。  

プロパティーのプライベート化については現状 class 構文で使用できない（Chrome74以降のみ可）ですが、将来的に実装もされそうですし、この検証結果の元もっと class 構文を積極的に使っていこう！という考えを自分の中で促進することができました。
