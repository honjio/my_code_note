# CircleCI 初学者が躓いた点と覚えておいた方が良さそうと感じた点をピックアップ

半分自分の忘備録みたいなものですが、同じく初学者の方であったり他の誰かの役にたてば幸いです。

## 躓いた点

実際に自分が躓いた点いくつかの紹介です。

### indent ミス

```yaml
# NG
workflows:
  example:
    jobs:
      - build:
        context: EXAMPLE

# OK
workflows:
  example:
    jobs:
      - build:
          context: EXAMPLE
```

コンテキストに設定した環境変数を使おうと思ったが上手く動かなかった。単純ですが yml ファイルを書き慣れていなかったので indent ミスに気付かなかった。

### サブディレクトリで依存関係のインストール

```yaml
# NG
steps:
  - checkout

  # functions 配下へ移動
  - run: cd functions

  - restore_cache:
      keys:
        - yarn-packages-v1-{{ checksum "yarn.lock" }}

  - run: yarn install --frozen-lockfile

# OK
steps:
  - checkout

  - restore_cache:
    keys:
      - yarn-packages-v1-{{ checksum "functions/yarn.lock" }}

  - run: yarn --cwd functions install --frozen-lockfile

# OK
working_directory: ~/project/functions
steps:
  - checkout:
      path: ~/project

  - restore_cache:
      keys:
        - yarn-packages-v1-{{ checksum "yarn.lock" }}

  - run: yarn install --frozen-lockfile
```

ディレクトリ階層構造の都合上、プロジェクト直下ではなくサブディレクトリで依存関係をインストールする必要があった。`cd` コマンドでサブディレクトリへ移動した後に `yarn install` を実行するように書いたつもりだったが、依存関係のインストールができていなかった。

run (command) の実行は `working_directory` で指定されたパス（デフォルトは ~/project）で実行されるため `cd` ではなく任意のパスで実行する `yarn --cwd`（npm なら `npm --prefix`）や、依存関係インストール先のパスを `working_directory` へ指定しておき `checkout` で別途パスを設定する方法などを取る必要がある。

### run (command) で環境変数が展開されない

```yaml
# NG
- run:
    name: sed による置換処理
    # single quote
    command: sed -i 's/XXXXXXXXXX/${API_KEY}/g' ./src/plugins/firebase.js

# OK
- run:
    name: sed による置換処理
    # double quote
    command: sed -i "s/XXXXXXXXXX/${API_KEY}/g" ./src/plugins/firebase.js
```

これは CircleCI というよりも単純な shell script ミスだが、${API_KEY}という文字列でそのまま置換されてしまった。シングルクオートで囲んだ部分は全てエスケープされてしまう。PHP なども確か同じような挙動でしたね。

## 覚えておくと良さそうな点

資料を読んだり、実際に動かしてみて覚えておくと良さそうと感じたリストです。

* CircleCI 2.1 で yml ファイルを記述する場合は workflows にバージョン指定は必要ない。
* 無料枠では job の並列実行が不可能なので、workflows で各 job に requires を設定しない場合は順序不同で同期的に処理される。
* job が１つなら workflows を使用しなくても暗黙的に実行される。ただし job の名前は" build" である必要がある。そうでなければ処理は失敗する。
* job が２つ以上で workflows の記述がない場合、"build" という名前の job のみが暗黙的に実行される。
* CircleCI が提供する docker イメージ（Node.js）では yarn はプリインストールされている。
* yarn や npm を使って依存関係をインストールする場合は restore_cache / save_cache 用いてキャッシュを活用すると良い。
* restore_cache / save_cache を利用する場合 yarn では --frozen-lockfile フラグを利用し yarn.lock ファイルの更新を防ぐことを推奨する。npm でも同じ理由から npm install ではなく npm ci コマンドを利用するのが良さそう？

## 学習にあたり参考になった資料

* [CircleCi official document | Hello World](https://circleci.com/docs/ja/2.0/hello-world/)
* [CircleCI official document | ワークフロー](https://circleci.com/docs/ja/2.0/workflows/)
* [CircleCI official document | 環境変数の使用](https://circleci.com/docs/ja/2.0/env-vars/)
* [CircleCI official document | コンテキストの使用](https://circleci.com/docs/ja/2.0/contexts/)
* [CircleCI official document | Yarn（npm の代替）の使用](https://circleci.com/docs/ja/2.0/yarn/)
* [CircleCI official document | Node.js - JavaScript チュートリアル](https://circleci.com/docs/ja/2.0/language-javascript/)
* [CircleCI official document | 依存関係のキャッシュ](https://circleci.com/docs/ja/2.0/caching/)
* [Qiita | CircleCIに入門したので分かりやすくまとめてみた](https://qiita.com/gold-kou/items/4c7e62434af455e977c2)
* [Qiita | CircleCI 2.1のすゝめ](https://qiita.com/yumikokh/items/3c1c6576db55d7db947d)
* [Qiita | npm ciを使おう](https://qiita.com/mstssk/items/8759c71f328cab802670)
* [tech.recruit-mp.co.jp | CircleCI 2.1 の新機能](https://tech.recruit-mp.co.jp/dev-tools/post-14868/)
* [Stackoverflow | CircleCi 2.0 working with project inside the subdirectory](https://stackoverflow.com/questions/50570221/circleci-2-0-working-with-project-inside-the-subdirectory)
* [Github | yarnpkg/yarn#692 | Analog to npm's --prefix command](https://github.com/yarnpkg/yarn/issues/692)
