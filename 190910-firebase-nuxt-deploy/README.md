# Firebase への nuxt project の deploy 設定

自身の覚え書きとして残します。なので下記３点のみに絞った内容になっています。  
Functions での処理（index.js）などについては書きません。

* Functions への deploy にいて
* Hosting への deploy について
* firebase.json の設定について

## 前提

1. nuxt は SSR モード
2. Server Side Rendering を行うため functions から html を配信
3. 静的リソースに関しては Hosting から配信。**HostingURL/assets/** から配信する。

### ディレクトリ構成

```jsonc
.
├── functions  // firebase init command で生成
│   ├── nuxt   // 後で追加
│   └── src
├── public     // firebase init command で生成
│   └── assets // 後で追加
└── src        // local nuxt project
    ├── assets
    ├── components
    ├── layouts
    ├── middleware
    ├── pages
    ├── plugins
    ├── static
    └── store
```

* `./functions`
  * Functions へ deploy するファイルおよびディレクトリ群
* `./public`
  * Hosting へ deploy するファイルおよびディレクトリ群
* `./src`
  * 開発用 nuxt project ディレクトリ

## Functions へ deploy するもの

nuxt project をビルドした後にできる成果物 `./src/.nuxt` が SSR の為に必要。  
`./functions/nuxt` ディレクトリを作成し、その中に `./src/.nuxt` の内容をコピーしておく。
`./src/package.json` に下記 npm script を記載しておく。

```jsonc
"scripts": {
    // ...
    "build" :"nuxt build && cp -R .nuxt/ ../functions/nuxt",
    // ...
}
```

## Hosting へ deploy するもの

Functions により SSR で配信されたコンテンツに必要な静的リソースを deploy する必要がある。  
配信 path は **HostingURL/assets/** にしたいので `./public/assets` ディレクトリを作成する。  
nuxt project をビルドした後、静的リソースは `./src/.nuxt/dist/client` に生成されるので、  
`./public/assets` へ内容をコピーしておく。

## deploy 前の build command の作成

`./package.json` へ deploy する前の build コマンドを作っておくと楽。  
下記コマンドは deploy の為に必要な準備を行う。

```jsonc
  "scripts": {
    // ...
    "build": "rm -rf public/* && npm --prefix src run build && cp -R functions/nuxt/dist/client/ public/assets",
    // ...
  }
```

※ 「Functions へ deploy するもの」で `./src/package.json` の "build" に予め加筆しておいたので、  
`npm --prefix src run build` 実行後 `./functions/nuxt` が作成されます。

## deploy 設定ファイル（firebase.json）

Functions / Hosting へ deploy するものを踏まえた上で下記のようになる。

```json
{
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*"
      "**/node_modules/**"
    ],
    "rewrites": [{
      "source": "**",
      "function": "ssrapp"
    }]
  },
  "functions": {
    "predeploy": "npm --prefix \"$RESOURCE_DIR\" run build"
  }
}
```

### firebase.json のルールについて

```jsonc
{
  /**
   * firebase Hosting への deploy 設定
   */
  "hosting": {
    /**
     * public: には deploy したいディレクトリを設定。
     */
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*"
      "**/node_modules/**"
    ],
    /**
     * firebase Hosting の URL から関数を提供
     * source: に指定されたパスにアクセスされた際に、
     * function: に指定された関数を提供。
     */
    "rewrites": [{
      "source": "**", // https://HostingURL/**
      "function": "ssrapp"
    }]
  },
  /**
   * firebase Functions への deploy 設定
   */
  "functions": {
    /**
     * TypeScript Project の場合は predeploy に記載されているコマンド実行後
     * "$RESOURCE_DIR" を functions へ deploy する。なお、$RESOURCE_DIR は
     * "functions" とハードコーディングしても問題ない
     */
    "predeploy": "npm --prefix \"$RESOURCE_DIR\" run build"
    /**
     * functions に deploy するファイルが通常の js で記載の場合 predeploy の
     * 必要は特になく、下記のように source: に対して deploy したいディレクトリー
     * を設定する。
     */
    // "source": "functions"
  }
}
```

### 基本ドキュメント

* [Hosting 設定](https://firebase.google.com/docs/hosting/full-config?hl=ja)
* [Functions 設定](https://firebase.google.com/docs/functions/manage-functions?hl=ja)
* [Functions with typescript 設定](https://firebase.google.com/docs/functions/typescript?hl=ja)

## 覚え書きとして記事を書いた経緯

自身で個人開発している nuxt project の Firebase Hosting への deploy 設定が間違っていたためです。  
SSR コンテンツは `assets/img/*.gif` からリソースを取得しているのに対し、Hosting されたリソースのパスは `assets/client/img/*.gif` となっていたため、Hosting から配信すべき静的リソースが、Functions から動的に配信されていました。  
そのため、Hosting パスを間違っていても画像や js は読み込まれてしまうため、中々気付きませんでした。

## 参考URL

* [nuxt.js + firebase で最小構成SSR | firebase について](https://marunouchi-tech.i-studio.co.jp/5237/)
* [github issue "$RESOURCE_DIR" について](https://github.com/firebase/firebase-tools/issues/610#issuecomment-441477295)
* [qiita Nuxt.js を Firebase にデプロイする時の注意点](https://qiita.com/okamuuu/items/c9f2989241af4a1bf588#nuxtjs-%E3%82%92-firebase-%E3%81%AB%E3%83%87%E3%83%97%E3%83%AD%E3%82%A4%E3%81%99%E3%82%8B%E6%99%82%E3%81%AE%E6%B3%A8%E6%84%8F%E7%82%B9)
