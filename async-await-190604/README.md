# async / await メモ

await が登場する前の then を使った Promise からの値の取り出しについては割愛

**前提**

* Promise を返す => その中の値ではなく、Promise というオブジェクト  
* resolve は Promise の値を解決する  
* await は Promise の値の解決を待ち、その値を取り出します  

```js
/**
 * async 関数は Promise を返します。return した値が resolve されます（promise の値の解決に使用されます）
 */
const normalAsyncFunc01 = async () => {
    return 'Apple'
}

/**
 * await を使用することで Promise を返す関数が Promise の値を解決するまで待ちます
 * await は resolve（promise の値の解決）を待つ。そして Promise の中の値を取り出すことができる
 * await は async 関数の中でしか使用できない
 */
(async() => {
    console.log(normalAsyncFunc01()); // Promise {<resolved>: "Apple"}
    console.log(await normalAsyncFunc01()) // Apple
})();

/**
 * setTimeout, XmlhttpRequest, addEventListener('DOMContentLoaded') など、
 * 同期的に処理できない関数を Promise を返す関数化することで、await でそれを待つことができる
 */
const normalAsyncFunc02 = async () => {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve('Box1');
        }, 3000)
    });
};

/**
 * 余談。normalAsyncFunc02 と normalAsyncFunc03 の構文は同じです。
 * return のみ行う関数（return までに何らかの処理をしない）の場合 return 文は必要ない。
 */
const normalAsyncFunc03 = async () => new Promise((resolve) => {
    setTimeout(() => {
        resolve('Box2');
    }, 3000)
});

(async() => {
    console.log(normalAsyncFunc02()); // Promise {<pending>}
    console.log(await normalAsyncFunc02()) // Box1
    console.log(await normalAsyncFunc03()) // Box2
    console.log('Box3'); // Box1, Box2 の resolve（Promiseの解決）を待つのでそのあとに実行される。
})();

/**
 * NG例
 * async 関数は Promise を返すが、setTimeout() の関数内の値を return できないため、
 * setTimeout 内の処理を待つことができない。そのため、new Promise() 内で setTimeout を書く必要がある。
 */
const normalAsyncFunc04 = async () => setTimeout(() => {
    // この Box4 は normalAsyncFunc04() ではなく setTimeout() の引数に定義されている無名関数に返る。
    return 'Box4';
}, 3000);

(async() => {
    console.log(await normalAsyncFunc04()); // Box4 は出力されない
})();
```
