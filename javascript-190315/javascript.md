# Javascript で陥りやすい失敗例を振り返る

個人的に今まで JavaScript を書いてて陥った失敗例などを振り返ってみました。  
この記事にあるいくつかの失敗例については恐らく殆どの方が経験してるのではないかなと思います。  
これから JavaScript 勉強するぞ！！という方や、現在進行形でこのような失敗に陥っている方の助けになれば幸いです。  

コードの解説に関しては簡潔に行なっているので、気になった方はググってください。

## DOM の取得及び操作

要素を取得して is-close なスタイルを付与したい。  
しかし、エラーになってスタイルを付与できない

```js
// 失敗例
const hoge = document.getElementsByClassName('hoge');
hoge.classList.add('is-close');

// 正しい例
const hoge = document.getElementsByClassName('hoge')[0];
hoge.classList.add('is-close');
```

解説：  
class は id と違い同じ名前が複数存在するので、`document.getElementsByClassName()`  
で取得できるのは HTMLCollection という配列のような要素の塊（配列ライクなオブジェクト）となります。

## addEventListener を for 文で複数生成時

A要素の n番目をクリックしたら B要素の n番目を is-active なスタイルにしたい。  
しかし、A要素の１番目や２番目をクリックしても、B要素の３番目しか is-active なスタイルにならない

```js
const elmA = document.querySelectorAll('.elmA');
const elmB = document.querySelectorAll('.elmB');

// 失敗例
for (var i = 0; i < 2; i++) {
    elmA[i].addEventListener('click', () => {
        elmB[i].classList.add('is-active');
    });
}

// 正しい例
for (let i = 0; i < 2; i++) {
    elmA[i].addEventListener('click', () => {
        elmB[i].classList.add('is-active');
    });
}
```

解説：  
変数定義に使用する `var` はブロックスコープを持たないため、例えば for 文で定義された `var i = 0` がループ終了後 `i = 2` なった時、  
関数内で使用されている `i` の値も `i = 2` を参照し、全て `2` となる（この現象をクロージャーという）。  
`let` はブロックスコープを持ち for 文でループの都度にスコープを持つので、1週目の値を2週目で引き継ぐということが起きない

## 予期せぬイベントの発火（実例有） その１

「開く」ボタンを押したらポップアップが開き、「閉じる」を押したらポップアップを閉じるようにしたい。  
しかし、「閉じる」を押しても勝手に再度ポップアップが開いてしまう

<p class="codepen" data-height="407" data-theme-id="0" data-default-tab="css,result" data-user="yuki153" data-slug-hash="NJYGKj" style="height: 407px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="test-sropPropagation">
  <span>See the Pen <a href="https://codepen.io/yuki153/pen/NJYGKj/">
  test-sropPropagation</a> by yuki153 (<a href="https://codepen.io/yuki153">@yuki153</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

```js
const popupWrap = document.querySelector('.popup-wrap');
document.querySelector('.fn-open').addEventListener('click', () => {
    setTimeout(() => {
        popupWrap.classList.add('is-show')
    }, 500)
})

// 失敗例
document.querySelector('.fn-close').addEventListener('click', () => {
    popupWrap.classList.remove('is-show');
})

// 正しい例
document.querySelector('.fn-close').addEventListener('click', (e) => {
    e.stopPropagation(); // 重要
    popupWrap.classList.remove('is-show');
})
```

解説：  
通常クリックなどの様々なイベントは JavaScript の仕様として子要素から親要素へ伝搬してしまう（バブリングという）。従って、fn-close をクリックした時、親要素に fn-open 要素が存在するので、fn-open 要素でもクリックイベントが発生してしまう。  
`e.stopPropagation()` というメソッドはこのバブリングを無効にしてくれる。  
※ `return false;` でも同じ動きを担う

追記:  
上記例では `e.stopPropation()` を使用して解決していますが、親要素と子要素で同じイベント（クリックなど）を監視している場合は親要素のみに addEventListener を指定し、そこに子要素がクリックされた時の処理も含めた方がバブリングによる他の addEventListener の発火を気にせず解決できます。また、このような動きを実装する際はバブリングなどの仕様も加味した上で HTML の構造を見直すのが良いでしょう

## 予期せぬイベントの発火（実例有） その２

ページを下までスクロールした後「TOP」ボタンを押すと、緩やかにページのトップへ戻る。  
その際にボタンは綺麗にフェードアウトする。しかし、ボタンは一度「点滅」を挟んでフェードアウトする  
<p class="codepen" data-height="265" data-theme-id="0" data-default-tab="html,result" data-user="yuki153" data-slug-hash="Ygejqw" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="test-preventDefault">
  <span>See the Pen <a href="https://codepen.io/yuki153/pen/Ygejqw/">
  test-preventDefault</a> by yuki153 (<a href="https://codepen.io/yuki153">@yuki153</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

```js
const btn = $('.scroll-button');
$(window).scroll(() => {
  if ($(this).scrollTop() > 500) {
    btn.fadeIn();
  } else {
    btn.fadeOut();
  }
});

// 失敗例
btn.click (() => {
  $('body, html').animate({ scrollTop: 0 }, 500);
});

// 正しい例
btn.click ((e) => {
  e.preventDefault(); // 重要
  $('body, html').animate({ scrollTop: 0 }, 500);
});
```

解説：  
「TOP」へ戻るボタンを押した際に、a タグに設定している href="#" が動作してしまっているのが原因。一度 href="#" でトップへ戻ったことで `fadeOut()` が実行され、次に jQuery の animate でまた下から上へ戻る処理を行うので `fadeIn()` が差し込まれて「点滅」してしまう。
`e.preventDefault()` はブラウザーが元々持つ機能を無効化してくれるので、href="#" の処理が行われなくなる  
※ `return false;` でも同じ動きを担う

## Object の中身が空であるか if 文での条件判定

Object の中身が空の場合 else 文を実行したい。  
しかし、意図する else 文の処理は行われず、条件を満たした方の処理が実行されてしまう

```js
let obj = {};

// 失敗例
if (obj) {
    console.log('obj の中身は存在する')
} else {
    console.log('obj の中身は空だ');
}

// 正しい例
if (Object.keys(obj).length) {
    console.log('obj の中身は存在する')
} else {
    console.log('obj の中身は空だ');
}
```

解説：
実は空なオブジェクト（`{}`）は真偽値で表すと `true` となるため if 文の条件を満たしてしまう。  
そのため `Object.keys(obj).length` で、その Object が key を持っていないかどうかを見る

## this が意図した値と違う

class 構文を用いて hoge 要素をクリックで Hello を出力するインスタンスを生成する。  
しかし、実際には Hello ではなく undefined が出力されてしまう

```js
// 失敗例
class Hoge {
    constructor() {
        this.word = 'Hello';
        this.el = document.getElementById('hoge');
        this.hoge();
    }
    hoge() {
        this.el.addEventListener('click', this.func)
    }
    func() {
        console.log(this.word);
    }
}
new Hoge();

// 正しい例
class Hoge {
    constructor() {
        this.word = 'Hello';
        this.el = document.getElementById('hoge');
        this.hoge();
    }
    hoge() {
        // アロー演算子で関数定義することで、定義時の this の値を拘束できる
        this.el.addEventListener('click', () => { this.func() });
    }
    func() {
        console.log(this.word);
    }
}
new Hoge();
```

解説：  
`element.addEventListener` の第二引数に定義する `function` の内の `this` はその `element` 自身を参照してしまう。  
なので `func()` 内の `this.word` は `element.word` という解釈になり undefined となる。そこで、アロー演算子を使うと定義時の `this` の値を拘束できるため、正しく `func()` が実行される

## addEventListener が重複して登録される

class 構文の中で addEventListener を使用していると、再度 new する前に remove を忘れがちになる  
結果、イベントが重複し処理も重複した分だけ実行されてしまう

```js
// 失敗例
class Hoge {
    constructor() {
        this.word = 'hello';
        document.addEventListener('keydown', () => { this.output() });
    }
    output() {
        console.log(this.word);
    }
}
let hoge = new Hoge();
hoge = new Hoge();
hoge = new Hoge(); // 一回 keydown を行うと hello が三回出力される

// 正しい例
class Hoge {
    constructor() {
        this.word = 'hello';
        // 無名関数は remove できないので、変数に格納する
        this.output = () => { this._output() };
        document.addEventListener('keydown', this.output);
    }
    _output() {
        console.log(this.word);
    }
    removeListener() {
        document.removeEventListener('keydown', this.output);
    }
}
let hoge = new Hoge();
hoge.removeListener();
hoge = new Hoge();
hoge.removeListener();
hoge = new Hoge(); // 一回 keydown を行うと hello が一回出力される
```

解説：  
class 構文内で `addEventListener` を実行する際は、`removeEventListener` できるようにしておこう。無名関数の場合は remove しようにも参照しようが無いので、変数などに格納するなどしよう

## async await を使用した非同期処理

非同期な処理を行う関数に async を付与し Promise を返すようにして await 構文で処理を待ちたい。  
しかし、処理は非同期処理は待たれず undefined を返してしまう

```js
// 失敗例
async function ajax() {
    const xhr = new XMLHttpRequest();
    const reqParam = 'https://XXXXXXX.jp/XXXX.json';
    xhr.addEventListener('loadend', () => {
      if (xhr.status === 200) {
        const res = JSON.parse(xhr.responseText);
        return res;
      }
    });
    xhr.open('GET', reqParam);
    xhr.responseType = 'text';
    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    xhr.send();
}

async function func() {
    const data = await ajax();
    console.log(data);
}

// 正しい例
async function ajax() {
    // きちんと Promise で非同期処理を包む必要がある
    return new Promise((resolve) => {
        const xhr = new XMLHttpRequest();
        const reqParam = 'https://XXXXXXX.jp/XXXX.json';
        xhr.addEventListener('loadend', () => {
        if (xhr.status === 200) {
            const res = JSON.parse(xhr.responseText);
            // 解決したい（待ちたい）値を resolve() する必要がある
            resolve(res);
        }
        });
        xhr.open('GET', reqParam);
        xhr.responseType = 'text';
        xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        xhr.send();
    });
}

async function func() {
    const data = await ajax();
    console.log(data);
}
```

解説：  
非同期な処理を行う関数に `async` を付与することで `Promise` を返すようになるが、  
処理自体を解決するまで待ってくれる訳では無い。きちんと待ちたい非同期な処理を `Promise()` で囲み、  
`resolve()` する必要がある。  
失敗例では処理は待ってくれず、返す値が無いので `Promise` でラップされた `undefined` が返る

## ある API からの返り値が null な時

普段は配列で返ってくる API からの返り値が null な時  
（例）ユーザーの購入商品を配列で返す API だが、何も購入していないユーザーの場合は null が返ってくる場合

```js
async function getPurchasedItems() {
    /* 取得処理*/
}

// 失敗例
async function func() {
    const res = await getPurchasedItems();
    if (res.length) {
        /* 続きの処理 */
    }
}

// 正しい例
async function func() {
    const res = await getPurchasedItems();
    if (res && res.length) {
        /* 続きの処理 */
    }
}
```

解説：  
これでエラーに会った時は「配列で返ってきてるのに、そもそも無い時は null なの！？」ってなった。  
API の返り値はきちんと確認しておこう！
