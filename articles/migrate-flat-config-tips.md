---
title: "FlatConfig移行時に知っておくとよいかもしれないこと"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["eslint", "javascript", "typescript"]
published: true
---

<!-- textlint-disable -->
## はじめに

こんにちは。
2024年はESLint v8がEOLになったり、Next.jsがESLint v9をサポートしたりで設定ファイルを`Flat Config`形式に移行した方が多いかもしれません。

私自身、業務でそれなりに大きいプロジェクト2つで移行を経験しました。
今回はそこで得た知見をまだ移行していない方向けに書き連ねてみようと思います。

:::message

本記事ではFlatConfigとLegacy Configの違いに関して詳しく言及はいたしません。
違いについては公式ドキュメントや以下の記事をご確認ください。

<https://zenn.dev/yumemi_inc/articles/eslint-legacy-config-flat-config-structure>

:::

## 知っておくとよいかもしれないこと

### 移行ツールを使う

初手は移行ツールで移してしまうのがオススメです。
公式ドキュメントにある通り、ESLintが移行ツールを提供しています。

こちらは`FlatCompat`を使って、ざっくりと移行できるツールになっています。
`.eslintrc.js`では上手く動作しないケースがあるようですが、とりあえずこれで移行しておくとコマンドを実行できる状態は担保できます。

```sh
npx  @eslint/migrate-config .eslintrc.json 
```

### エディタの設定を変更する

vscodeを利用されている方は`.vscode/settings.json`から`eslint.useFlatConfig`を有効化しましょう。
移行作業中はLegacy Configと行ったり来たりということがよくあるので、プロジェクトフォルダにローカル設定として追加するのがオススメです。
これを忘れるとエディタ上にエラーが表示されなくなりますので、最初のうちに変更してしまいましょう。

### `typescript-eslint`を使用する

typescript向けの設定でよく`@typescript-eslint/parser`や`@typescript-eslint/eslint-plugin`が開発パッケージに含まれています。

あまりシビア出ない方は[`typescript-eslint`](https://typescript-eslint.io/packages/typescript-eslint)のパッケージにまとめてしまうことをお勧めします。

`typescript-eslint`で利用できる`config`関数を使うことで、設定を型補完がある状態で記載できます。

この`config`関数は単に型を提供するだけでなく、以下の機能を提供します。

1. extends
   extendsを利用することで、設定をある程度まとめることが可能となります。

   ```js
   // extendsを利用した書き方
   export default tseslint.config({
     files: ['**/*.ts'],
     extends: [
       eslint.configs.recommended,
       tseslint.configs.recommended,
     ],
     rules: {
       '@typescript-eslint/array-type': 'error',
       '@typescript-eslint/consistent-type-imports': 'error',
     },
   });

   // ↓と同じ
   export default tseslint.config(
     eslint.configs.recommended,
     tseslint.configs.recommended,
     {
        files: ['**/*.ts'],
        rules: {
          '@typescript-eslint/array-type': 'error',
          '@typescript-eslint/consistent-type-imports': 'error',
        },
      }
   )
   ```

2. 内部で配列をフラットにしてくれる
   Flat Configでは`tseslint.configs.recommended`のように配列の設定を利用する場合、スプレッド構文を利用する必要があります。
   これは利用する設定により異なりますが、`mjs`で記述する場合、どれが配列でどれがオブジェクトなのかがかなりわかりにくいです。
   このヘルパー関数は内部で配列を展開してくれるため、利用者側がツール側の設定の構造を意識せずに設定することが可能となります。

   :::message
   ESLintは追加設定によりtypescriptで設定を書くことも可能になりましたが、型を使うことができない設定も多く、それほど有用ではありません。
   :::

   ```js
   // ヘルパー関数を利用しない
   export default [
     eslint.configs.recommended,
     ...tseslint.configs.recommended,
     {
       /*... */
     },
     // ...
   ];

   // ヘルパー関数を利用する
   export default tseslint.config(
     eslint.configs.recommended,
     tseslint.configs.recommended,
     {
       /*... */
     },
     // ...
   );
   ```

### 設定ファイル用の`tsconfig`を作成する

typescript用の設定で`parserOptions.project`に既存の`tsconfig.json`を指定していると画像のようなエラーが表示されるケースがあります。
これは対象のファイルが`tsconfig.json`の`includes`に含まれていないことが原因です。

![parse project error](/images/migrate-flat-config-tips/parse-project-error.png)

エラー内容の通りですが、解決方法としては以下のような選択肢があります。

- ESLintの対象ファイルからこのファイルを外す
- `tsconfig.json`の`includes`に設定ファイルを追加する
- 新しく設定ファイルを含める設定をした`tsconfig.xxx.json`を作成し、それを`parserOptions.project`に指定する

個人的には一番最後の`tsconfig.xxx.json`を作成する方法がオススメです。
この方法にしておくとあまり考えることなく追加でき、`tsc`の実行時は含めないということが可能になります。
設定の例は以下の通りです。

```json:tsconfig.eslint.json
{
  "extends": ["./tsconfig.json"],
  "compilerOptions": {
    "allowJs": true
  },
  "include": ["**/*.ts", "eslint.config.mjs"]
}
```

```js
export default tseslint.config({
 // ...
 languageOptions: {
  parserOptions: {
    projectService: "tsconfig.eslint.json",  // 新規作成したtsconfigを指定
  },
},
});
```

### `name`プロパティを指定する

各オブジェクトレベルの設定に`name`を指定できます。
一番の理由は[`eslint/config-inspector`](https://eslint.org/docs/latest/use/configure/debug#use-the-config-inspector)で確認する際にわかりやすいという理由です。
しかしながら`config-inspector`を使わない場合でもこの設定は何の設定なのかの認識コストが下がるのでオススメです。

:::message
`tseslint.config`の`extends`に指定しているものは、`config-inspector`上での名称は`<親name>__<子name>`となります。
以下の設定の場合、`project/root__aaa`になります。

:::details eslint.config.mjs

```js
export default tseslint.config({
  name: "project/root"
  extends: [
    {
      "name": "aaa",
      // ...
    }
  ],
  // ...
});
```

:::

```js
export default [
  {
    name: "project/ignore",
    /*... */
  },
  {
    name: "project/xxx",
    /*... */
  },
  {
    name: "project/yyy",
    /*... */
  },
];
```

### プラグインを自作する

以前の設定では自作のルール・プラグインの導入に[`eslint-plugin-local-rules`](https://github.com/cletusw/eslint-plugin-local-rules)や[`eslint-plugin-rulesdir`](https://github.com/eslint-community/eslint-plugin-rulesdir)といった追加のプラグインが必要でした。
しかしながら、Flat Configではこれらは必要ありません。ローカルルールやプラグインは他のjsファイル同様に`import`, `require`で使用できます。

```js:eslint/rules/yyy.mjs
/** @type {import('eslint').Rule.RuleModule} */
const rule = {
  meta: {/*... */},
  create: () => {/*... */}
}

export default rule;
```

```js:eslint/eslint-plugin-xxx.mjs
import yyyRule from "./rules/yyy.mjs"

/** @type {import('eslint').ESLint.Plugin} */
const plugin = {
  meta: {
    name: "eslint-plugin-xxx"
  },
  rules: {
    yyy: yyyRule,
    // ...
  },
  configs: {/*... */},
}

export default plugin;
```

```js:eslint.config.mjs
import xxxPlugin from "./eslint/eslint-plugin-xxx.mjs"

export default tseslint.config({
  name: "project/root"
  plugin: {
    xxx: xxxPlugin
    // ...
  },
  rules: {
    "xxx/yyy": "error"
    // ...
  }
});
```

### eslintのcliを直接使用する

これは`next lint`を使用している方向けです。
`v15.1.7`時点で`next lint`はESLint v9での追加オプションに対応していません。
例えば、Gitフックでステージ上のファイルに対して実行する際に、指定したファイルが`ignores`に含まれていると警告が出ます。
ESLint v9のcliでは`--no-warn-ignored`というオプションにより無効化できますが、`next lint`では未サポートです。

利用されるプロジェクトによっては同じようなケースがあるかと思いますので、サポートされるまでは直接ESLintのcliを実行することも手段として持っておくとよいです。

## まとめ

今回はFlat Config移行へのTipsをまとめてみました。
ありきたりではありますが、知っていたり気を付けたりすることで設定を移行しながら改善できることは多いです。
何かしら皆さんのお役に立てれば幸いです。

<!-- textlint-enable -->