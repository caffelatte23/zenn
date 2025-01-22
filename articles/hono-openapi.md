---
title: "hono-openapiでOpenAPI Specを生成する"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["hono", "openapi"]
published: true
---

## はじめに

[Hono](https://hono.dev/)を使って OpenAPI を生成する際、ネットをさまよっていると[`@hono/zod-openapi`](https://hono.dev/examples/zod-openapi#zod-openapi)を使われる方が多いように見受けられます。

しかしながら個人的にいくつか不満点があったため、今回は[`hono-openapi`](https://hono.dev/examples/hono-openapi#hono-openapi)を試してみました。

## hono-openapi

`hono-openapi`は`@hono/zod-openapi`と同じく OpenAPI Spec の生成を自動化するミドルウェアツールです。`@hono/zod-openapi`と比べて以下のような特徴があります。

- `zod`に限らず、`Valibot`や`ArkType`を含む多様なスキーマライブラリを使用できる。
- Hono の基本クラスを使うことをベースとしている。

<https://github.com/rhinobase/hono-openapi>

### `@hono/zod-openapi`への不満点

個人的な感想として`@hono/zod-openapi`へは以下のような不満点がありました。
少なからず以下の課題は`hono-openapi`で解決できました。

- `OpenAPIHono`クラスを使用する必要がある
  - 機能だけなら`Hono`クラスを使えるが、OpenAPI 生成を必要とする場合このクラスを利用する必要がある
- パスやメソッドに一覧性がない
  - どのメソッドを定義する場合でも`route.openapi()`の形式で定義する必要があり、route 定義の変数名を工夫するなどしないとそのファイル内だけでメソッドがわかりにくい

## 使い方

ここからは`hono-openapi`の使い方を紹介していきます。

### route定義

定義自体は`@hono/zod-openapi`とそれほど変わりませんが、`hono-openapi`ではパスやメソッドが含まれません。
あくまで補足的な形での定義となっており、可読性が損なわれないと感じます。

```ts
import { Hono } from "hono";
import { describeRoute } from "hono-openapi";
import { resolver } from "hono-openapi/zod";

// ドキュメントを定義
const route = describeRoute({
  description: "Say hello to the user",
  responses: {
    200: {
      description: "Successful response",
      content: {
        "text/plain": { schema: resolver(responseSchema) },
      },
    },
  },
});

// routeを登録
const app = new Hono();
app.get("/", route, (c) => {
  // ...
});
```

### validation

現状、`body`, `query`等のリクエストパラメータはスキーマライブラリを使用して定義はできません。
これらのドキュメントはスキーマライブラリごとに用意されるValidatorを追加すると自動で登録されます。

```ts
import { validator as zValidator } from "hono-openapi/zod";

app.get(
  "/{id}",
  route,
  zValidator("param", paramSchema), // validatorを追加
  (c) => {
    const { id } = c.req.valid("param");
    // ...
  }
);
```

レスポンスもデフォルトでは検証されませんが、`validateResponse`を有効にすれば検証が可能です。

```ts
const route = describeRoute({
  // ...
  validateResponse: true, // ← レスポンスの検証を有効化
});
```

### OpenAPI Spec を生成

`@hono/zod-openapi`と同じく OpenAPI Spec の生成・Swagger UI等でのホストが可能です。
同じサーバーでホストしたくない場合は、オブジェクトとして出力し別サーバーでのホストも可能です。

```ts
import { generateSpecs, openAPISpecs } from 'hono-openapi';
import { Hono } from 'hono';
import { swaggerUI } from '@hono/swagger-ui';

const app = new Hono();

const documentation = {
  openapi: '3.1.0',
  info: {
    title: 'My API',
    version: '0.0.1',
  },
};

// オブジェクトとして出力
generateSpecs(app, { documentation });

// ドキュメントをホスト
app.get('/doc', openAPISpecs(app, { documentation }));
```

### その他

`describeRoute`の`hide`プロパティを設定することで特定の条件でのみrouteを有効化できます。

```ts
const route = describeRoute({
  // ...
  hide: process.env.NODE_ENV === "production", // ← 本番環境ではこのルートはアクセスできない
});
```

## 注意点

### `zod-openapi`の代用で`@hono/zod-openapi`は使用できません

`hono-openapi`でも`@hono/zod-openapi`と同様に拡張された`zod`を利用する必要があります。  
どちらも`zod-openapi`をベースとしていますが、内部的に微妙な違いがあるようで`.openapi()`での挙動が異なります。  
`hono-openapi`を利用する場合は、レポジトリに記載の通り`zod-openapi/extend`を読み込む形式で使用しましょう。

<https://github.com/rhinobase/hono-openapi#setting-up-your-application>

## まとめ

今回は`hono-openapi`を紹介しました。
生の`Hono`クラスの書き味をそのままに、定義を追加できる手法が個人的に好みです。
私は hono 歴がまだまだ浅いので hono についても知見をためて行きたいです。
