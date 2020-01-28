# firebase.initializeApp() に必要な config 情報に appId がいつの間にか追加されていた

## 経緯

`firebase.initializeApp()` に必要な config の確認を行おうとしたところ、
確認の仕方が以前と少し変わっていました。  

結果としては「[ウェブアプリ用の設定スニペットを入手する](https://support.google.com/firebase/answer/7015592?hl=ja)」のドキュメントに沿う形で、「アプリを追加」という項目から「ウェブアプリにfirebaseを追加」という手順を進めると config 情報が確認できたのですが、もう既に firebase で ウェブアプリを公開していたので「アプリを追加」という項目に戸惑ってしまいました。

また表示された config 情報には `appId` という項目が増えていました。
現在 `appId` 無しの config で `firebase.initializeApp()` して問題なく動いているので、
そこもまた謎なので調べてみました。

```js
const config = {
  apiKey: "xxxxxxxxxxxxxx",
  authDomain: "xxxxx.firebaseapp.com",
  databaseURL: "https://xxxxx.firebaseio.com",
  projectId: "xxxxxxxxxxxx",
  storageBucket: "xxxxxx.appspot.com",
  messagingSenderId: "xxxxxxxxxxxxx",
  // appId: xxxxxxxxxxxxxxxxxx（新しく増えていた項目）
};
```

## 取り敢えず分かった事

* 以前ウェブアプリの config の中には `appId` は無かった（後から追加されたもの）
* `appId` は無くても動くが、firebase 関連のサービスの中には `appId` が必須なものがある
* 現在 firebase でウェブアプリを公開する手順として「アプリの登録」という手順を踏む。以前だとその様な手順が無かったために、既に deploy 済みであるウェブアプリは「アプリの登録」がなされていないステータスになってる（アプリの追加（登録）を行う事で appId を含む config 情報を確認できる）

## 参考情報について

firebase の公式ドキュメントを漁りましたが、いつ頃 `appId` が config 情報に含まれる様になったかは記載を見つけられませんでした。firebase を使ったウェブサイトの hosting 関連記事について、幾つかの古い記事、新しい記事の比較に基づいた結論です。

[stack_overflow](https://stackoverflow.com/questions/58680378/firebase-init-js-does-not-contain-appid) でも似たような質問が一つ見受けられました。
