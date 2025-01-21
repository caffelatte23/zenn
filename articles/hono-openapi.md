---
title: "hono-openapiでOpenAPI Specを生成する"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["hono", "openapi"]
published: false
---

## はじめに

[Hono](https://hono.dev/)を使って OpenAPI を生成する際、ネットをさまよっていると[`@hono/zod-openapi`](https://hono.dev/examples/zod-openapi#zod-openapi)を使われる方が多いように思います。

しかしながら個人的にいくつか不満点があったため、今回は[`hono-openapi`](https://hono.dev/examples/hono-openapi#hono-openapi)を試してみました。

## hono-openapi

`hono-openapi`は`@hono/zod-openapi`と同じく OpenAPI Spec の生成を自動化するミドルウェアツールです。`@hono/zod-openapi`と比べて以下のような特徴があります。

- `zod`に限らず、多様なスキーマライブラリを使用できる。
- Hono の基本クラスを使うことをベースとしている。

https://github.com/rhinobase/hono-openapi

### `@hono/zod-openapi`への不満点

個人的な感想として`@hono/zod-openapi`へは以下のような不満点がありました。
詳述しますが、少なからず以下の課題は`hono-openapi`で解決することができました。

- `OpenAPIHono`クラスを使用する必要がある
  - 機能として`Hono`クラスを使えないわけではありませんが、OpenAPI 生成を必要とする場合このクラスを利用する必要があります。
- パスやメソッドに一覧性がない
  - どのメソッドを定義する場合でも`route.openapi()`の形式で定義する必要があり、route 定義の変数名を工夫するなどしないとそのファイル内だけでメソッドがわかりません。

## 比較

ここからは`hono-openapi`と`@hono/zod-openapi`の使い方を比較していきます。
作業環境は以下を参照ください。

### ドキュメント定義

```ts
/**
 * hono-openapi
 */
```

### OpenAPI Spec をホスト
