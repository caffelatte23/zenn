---
title: "Standard Schemaで何が実現できる？"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "typescript"]
published: true
---

## 初めに

本記事では先日`v1.0.0`が公開された`Standard Schame`について紹介します。

### 本記事で紹介すること

- `Standard Schema`とは何か
- `Standard Schema`で実現できること
- `Standard Schema`に準拠済みのツール・フレームワーク

### 本記事で紹介しないこと

- スキーマライブラリでの準拠方法

:::message

本記事はReadmeの雑目なまとめ記事です。
より詳細を知りたい方は[こちら](https://github.com/standard-schema/standard-schema)をご確認ください。

:::

## Standard Schemaとは

[`Standard Schema`](https://github.com/standard-schema/standard-schema)は`zod`や`valibot`といったスキーマバリデーションライブラリの共通インターフェースの仕様です。
主に`zod`や`valibot`, `ArkType`の開発者により設計されています。

先日、`v1.0.0`が公開となり徐々に対応するツールも多くなっています。

<https://standardschema.dev/>

### これにより何を実現したいのか

現状、各ツール・フレームワークはそれぞれのスキーマライブラリに対応するためのロジックやアダプター（ex. `xxx/zod`,`xxx/arktype`,`xxx/valibot`..）を作成する必要があります。

これらは実装コストやメンテナンスコストが非常に高くなります。
`Standard Schema`に準拠することでスキーマライブライブラリ独自のロジックやアダプターの実装が不要となり、エコシステムツールがユーザー定義の型バリデーターの受け入れを容易にします。

### 準拠するとどうなるのか

[`t3-env`](https://env.t3.gg/)を例に準拠前後を比較してみます。  
`t3-env`は`v0.12.0`で`Standard Schema`に準拠しており、公式ドキュメントにも以下のような追記がなされています。

![t3-env-doc](/images/t3-env-doc.png)

準拠前の`v0.11.0`と準拠後の`v0.12.0`とでは`valibot`で指定しているプロパティがエラーにならなくなることがわかります。
このように極端な例ではありますが、例のように複数ライブラリの併用も可能です。

![t3-env-compare](/images/t3-env-compare.png)

## 設計目標

この仕様は以下の目標を満たすよう設計されています。

**ランタイムバリデーションのサポート**
このインターフェースに準拠したバリデータを使うことでデータの検証できます。すべてのエラーは標準化された形式のエラーを出力します。

**静的型インターフェースのサポート**
TypeScriptライブラリの型推論のために、仕様は推論された型を「公開」する標準的な方法を提供し、外部ツールで抽出して使用できるようにします。

**最小限**
ライブラリが既存の関数やメソッドを呼び出す数行のコードでこの仕様を実装することが容易であるべきです。

**APIコンフリクトを回避**
仕様全体は`~standard`という単一のオブジェクトプロパティ内に収められており、既存ライブラリのAPIサーフェスとの名前の競合を避けています。

**開発者体験（DX）を損なわない。**
`~standard`プロパティはチルダ（~）プレフィックスを付けることで、オートコンプリートの優先順位を下げています。対照的に、アンダースコアプレフィックスのプロパティは英数字の名前のプロパティやメソッドより前に表示されてしまいます。

## 準拠済のスキーマライブラリ

2025/01/30時点で 以下のスキーマライブラリは`Standard Schema`に対応済みです。
インストールするバージョンには気を付けてください。

＊サポートリストに追加してもらう場合は、自身でPRを作れば良いようです。([参考](https://github.com/standard-schema/standard-schema/pull/35))

| ライブラリ                                         | バージョン |
| -------------------------------------------------- | ---------- |
| [Zod](https://zod.dev)                             | 3.24.0+    |
| [Valibot](https://valibot.dev/)                    | v1.0+      |
| [ArkType](https://arktype.io/)                     | v2.0+      |
| [Arri Schema](https://github.com/modiimedia/arri)  | v0.71.0+   |
| [TypeMap](https://github.com/sinclairzx81/typemap) | v0.8.0+    |

## 準拠済のツール / フレームワーク

2025/01/30時点で、以下のツール / フレームワークは`Standard Schema`に準拠しています。

＊サポートリストへの追加手順はスキーマライブラリと同じです。

| インテグレーター                 | 説明                             |
| ------------------------------ | -------------------------------- |
| [tRPC](https://trpc.io)                               | 型安全かつ高速で安全なe2eのAPIを簡単に作成                                                                          |
| [TanStack Form](https://tanstack.com/form)            | TS/JS、React、Vue、Angular、Solid、Lit向けのヘッドレスで高性能な型安全なフォーム状態管理                                       |
| [TanStack Router](https://tanstack.com/router/latest) | データフェッチング、stale-while-revalidateキャッシング、ファーストクラスの検索パラメータAPIを備えた完全な型安全なReactルーター |
| [Hono Middleware 🚧](https://hono.dev)                 | Web標準に基づいた高速で軽量なサーバー                                                                                         |
| [Qwik 🚧](https://qwik.dev/)                           | 必要最小限のJavaScriptのみを読み込むことで高速で即時起動が可能なモダンなWebフレームワーク                                                                                            |
| [UploadThing](https://uploadthing.com/)               | モダンなWeb開発者向けのファイルアップロード                                                                                    |
| [T3 Env](https://env.t3.gg/docs/introduction)         | フレームワークに依存しない型安全な環境変数のバリデーション                                                                     |
| [OpenAuth](https://openauth.js.org/)                  | ユニバーサルな標準ベースの認証プロバイダー                                                                                     |
| [renoun](https://www.renoun.dev/)                     | React用ドキュメンテーションツールキット                                                                                        |
| [Formwerk](https://formwerk.dev/)                     | 高品質でアクセシブルなフォームを構築するためのVue.jsフレームワーク                                                             |
| [GQLoom](https://gqloom.dev/)                         | Standard Schemaを使用してGraphQLスキーマとリゾルバーを織り込む                                                                 |
| [Nuxt UI (v3)](https://ui.nuxt.com/)                  | VueとTailwind CSSを使用して構築された、モダンなWebアプリのためのUIライブラリ                                                   |
| [oRPC](https://orpc.unnoq.com/)                       | 型安全なAPIをシンプルに                                                                                                        |
| [Regle](https://reglejs.dev/)                         | Vue.jsのための型安全なモデルベースのフォームバリデーションライブラリ                                                           |
| [upfetch](https://github.com/L-Blondy/up-fetch)       | 小規模で組み合わせ可能なフェッチ設定ツール。適切なデフォルトと組み込みのスキーマバリデーションを備える                         |

## まとめ

今回は`Standard Schema`について紹介しました。
この仕様に準拠することでツール開発者の負担を軽減するだけでなく、使用者としては選択肢が広がるので非常に良い取り組みだなと個人的に感じました。

<!-- textlint-disable -->
今後、ツールの採用基準の1つとして見てみようと思います。
<!-- textlint-enable -->
