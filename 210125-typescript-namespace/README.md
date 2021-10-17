# TypeScript の namespace は非推奨ではない

## 結論から

公式の [TypeScript ドキュメント](https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html)の何処にも namespace は非推奨と示唆する記載は無い。
尚且つ [microsoft/TypeScript#30994(comment)](https://github.com/microsoft/TypeScript/issues/30994#issuecomment-491998238) で TypeScript team のリード開発者である Ryan Cavanaugh 氏が述べているように将来的に廃止される事も無い。

TypeScript の namespace について調べると公式では無い記事や投稿で非推奨（deprecated）というワードが目立つ事、tslint や typescript-eslint を使用していると namespace の使用で注意されてしまう事などから非推奨と認識されてしまっていると推測する。

個人的に「非推奨」という強いワードは「使用するな」というメッセージ性を感じる。開発者同士の間で誤解を生みかねないため namespace は非推奨では無いと伝えるためこの記事を書こうと思った。

## namespace の使用が敬遠される理由

非推奨という訳ではないが namespace の使用が推奨されている訳でも無い。なぜならば JavaScript 標準仕様である ESModules の import / export の使用で大体は namespace を使わずとも解決するからだ。現に typescript-eslint でも [no-namespace](https://github.com/typescript-eslint/typescript-eslint/blob/v3.10.1/packages/eslint-plugin/docs/rules/no-namespace.md) というルールが存在しており、import / export を優先して使用するよう設定されている。

## namespace が便利な時

標準的な import / export で殆ど事足りる事は事実である。  
それでも namespace の方が適しているケースもあると思っている。以下いくつかのパターンを紹介する。

### ① オブジェクトとして型を名前付き export したい時

```ts
/**
 * modules/hoge.ts
 */
export namespace Hoge {
    export type LowerCase = 'a' | 'b' | 'c';
}

/**
 * example.ts
 */
import { Hoge } from './modules/hoge.ts';

let text: Hoge.LowerCase = 'a';
```

```ts
/**
 * modules/types.ts
 */
export type LowerCase = 'a' | 'b' | 'c';

/**
 * example.ts
 */
export * as Hoge from './modules/types';

let text: Hoge.LowerCase = 'a';
```

前者も後者も `Hoge` という名前で型オブジェクトを import している。  
namespace を使用する前者は `Hoge` という名前付き export しているのに対して、後者では as 句を使用して `Hoge` という名前で import するようにしている。私個人的な意見としては、この場合は前者の方が優れていると考える。後者の場合、自由に名前をつけて import できてしまう事から一貫性が損なわれてしまう。

### ② ネストした型オブジェクトを export したい時

```ts
/**
 * modules/sample.ts
 */
export namespace Sample {
    namespace Hoge {
        export type LowerCase = 'a' | 'b' | 'c';
    }
}

/**
 * example.ts
 */
import { Sample } from './modules/sample';

let text: Sample.Hoge.LowerCase = 'a';
```

```ts
/**
 * modules/types.ts
 */
export type LowerCase = 'a' | 'b' | 'c';

/**
 * modules/re-export.ts
 */
export * as Hoge from './modules/types';

/**
 * example.ts
 */
import * as Sample from './modules/re-export';

let text: Sample.Hoge.LowerCase = 'a';
```

`Sample.Hoge.LowerCase` のような形で型を扱いたい場合に、前者の namespace ではネストした形で表現できるのに対し、後者の ESModules 形式で同じ形を実現させるには re-export を使用する必要が出てくる。各ファイルをモジュールとして扱う ESModules の仕様的には何もおかしくはないが、単純にそこまで分割管理したく無い場合には namespace を活用できる。

### ③ d.ts で型をグローバルに扱いたい時

```ts
/**
 * @types/example.d.ts
 */
namespace Hoge {
    export type LowerCase = 'a' | 'b' | 'c';
    export type UpperCase = 'A' | 'B' | 'C';
}
```

```ts
/**
 * @types/example.d.ts
 */
type LowerCase = 'a' | 'b' | 'c';
type UpperCase = 'A' | 'B' | 'C';
```

型定義ファイル（d.ts）なので ESModules との比較という意味では内容が少し逸れますが、型をグローバルに定義したい場合 d.ts が有効です。その際、後者のように型定義してしまうとグローバルで複数の型が散乱し名前衝突を起こしてしまう可能性が増えます（俗に言うグローバル汚染）。また、IDE 上でも複数の型がサジェストされるようになってしまう。[型の国のTypeScript（幽霊 namespace）](http://typescript.ninja/typescript-in-definitelyland/definition-file.html#fn-ghost-module) でも同じ様に namespace の活用事例が紹介されている。
