---
title: "最近のVue.jsのフォームライブラリについて（2025/12）"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "vue"
  - "nuxt"
published: false
---

:::message

本記事は [AI ソリューション Advent Calendar 2025](https://qiita.com/advent-calendar/2025/ais) の記事になります。

:::

## 初めに

皆さんは Vue.js で画面を構築される際、フォームライブラリは使用されますでしょうか。
React だと[`React Hook Form`](https://react-hook-form.com/)、[`Conform`](https://conform.guide/tutorial)が思い当たります。

私がここ 1,2 年で参画した Vue.js のプロジェクトではどれも[`vee-validate`](https://vee-validate.logaretm.com/v4/)が採用されていましたが、いくつか課題点を感じています。

最近では[`TanStack Form`](https://tanstack.com/form/latest) のような複数の UI フレームワークで利用可能なライブラリの登場などがあるので、改めて Vue.js で利用可能なフォームライブラリを調べてみます。

## vee-validate のしんどいところ

1. **暗黙的な値の共有**: vee-validate は内部的にグローバルな状態を持つため、コンポーネント間で予期しない値の共有が発生することがある

2. **スキーマ全体へのバリデーション**: useForm にスキーマを渡すと、フォーム全体に対してバリデーションが適用されるため、これらを制御する場合少し工夫が必要になってしまう

## 求めるもの

今回、新しいフォームライブラリを検討するにあたり、以下の条件を重視します。

1. **Standard Schema への対応**

   - Zod、Yup、Valibot など、異なるバリデーションライブラリ間で共通のインターフェースを提供する [Standard Schema](https://github.com/standard-schema/standard-schema) に対応していること
   - これにより、バリデーションライブラリの切り替えや、既存のスキーマ定義の再利用が容易になる

2. **明示的な値の管理**
   - グローバルな状態や暗黙的な値の共有を避け、各フォームインスタンスが独立して状態を管理すること
   - コンポーネントの再利用性が高く、予期しない副作用が発生しにくい設計であること
   - どこで値が共有されているかが明確で、デバッグしやすいこと

## 利用可能なフォームライブラリ

### Regle

[Regle](https://regle.dev/)は、Vue.js 向けに設計されたモデルベースのバリデーションライブラリです。Composition API を活用し、型安全性を重視した設計が特徴です。

**主な特徴:**

- **型安全性**: TypeScript のサポートが充実しており、バリデーションルールやエラーメッセージの型推論が強力
- **軽量**: 小さなバンドルサイズで、パフォーマンスに優れている
- **柔軟なバリデーション**: カスタムバリデーションルールの作成が容易
- **Composition API ネイティブ**: Vue 3 の Composition API と自然に統合
- **リアクティブ**: Vue のリアクティブシステムと完全に統合され、バリデーション状態が自動的に更新される

Regle は Vuelidate の後継として位置づけられており、より現代的な API と型安全性を提供しています。

**使用例:**

```vue
<script setup lang="ts">
import { useRegle } from "@regle/core";
import { required, email } from "@regle/rules";
import { reactive } from "vue";

const form = reactive({
  email: "",
});

const { r$, errors } = useRegle(form, {
  email: { required, email },
});

async function handleSubmit() {
  const isValid = await r$.$validate();
  if (isValid) console.log("Form is valid", form);
}
</script>

<template>
  <form @submit.prevent="handleSubmit">
    <input v-model="form.email" type="email" placeholder="Email" />
    <span v-if="r$.email.$error">{{ errors.email }}</span>
    <button type="submit" :disabled="r$.$invalid">Submit</button>
  </form>
</template>
```

#### 良い点・悪い点

- ✅ `required` や `between` などのビルトインルールも存在し、共通化も容易そう
- ✅ 値の連携が v-model ベースなため、記述が最小限でよく、暗黙的な値共有は見た限りなさそう
- ✅ Standard Schema 登場初期から対応済みリストに名を連ねており、更新頻度の高さが伺える
- ✅ ドキュメントが充実しており、様々な利用例が記載されており、学習コストが大きくなさそう

### TanStack Form

[TanStack Form](https://tanstack.com/form/latest)は、フレームワークに依存しないヘッドレスなフォーム管理ライブラリです。React、Vue、Solid、Angular など複数のフレームワークに対応しています。

**主な特徴:**

- **ヘッドレス UI**: UI コンポーネントを含まず、ロジックのみを提供するため、デザインの自由度が高い
- **型安全性**: TypeScript による強力な型推論をサポート
- **フレームワーク非依存**: 複数のフレームワークで同じ API を使用可能
- **パフォーマンス**: 最小限の再レンダリングで効率的なフォーム管理を実現
- **豊富な機能**: フィールド配列、ネストされたオブジェクト、条件付きフィールド、非同期バリデーションなどをサポート
- **統合しやすい**: Zod、Yup、Valibot などの既存のバリデーションライブラリと統合可能

TanStack Form は React Hook Form の思想を引き継ぎつつ、より汎用的なアプローチで設計されています。

**使用例:**

```vue
<script setup lang="ts">
import { useForm } from "@tanstack/vue-form";
import { z } from "zod";

const form = useForm({
  defaultValues: { email: "" },
  onSubmit: async ({ value }) => console.log("Form submitted", value),
});
</script>

<template>
  <form @submit.prevent="form.handleSubmit">
    <form.Field name="email" :validators="{ onChange: z.string().email('Invalid email') }">
      <template v-slot="{ field, state }">
        <input
          :value="field.state.value"
          @input="(e) => field.handleChange(e.target.value)"
          @blur="field.handleBlur"
          type="email"
          placeholder="Email"
        />
        <span v-if="state.meta.errors.length">
          {{ state.meta.errors[0] }}
        </span>
      </template>
    </form.Field>
    <button type="submit">Submit</button>
  </form>
</template>
```

#### 良い点・悪い点

- ✅ フレームワーク非依存で、React などの知見を活かせる
- ✅ debounce など他のフレームワークでは見られない豊富な機能がある
- ✅ blur, change などイベントごとに同期・非同期のルールを設定可能
- ❌ Vue 専用ではないため、書き方が Vue らしくない（v-model が使えない）
- ❌ コード量が多くなりがち
- ❌ フレームワーク非依存なこともあり、ドキュメントの更新が遅れがち

### その他

その他、検証機能を持つ & Standard Schema に対応しているライブラリを紹介します。

#### Nuxt UI

[Nuxt UI](https://ui.nuxt.com/)は、Vue, Nuxt 向けの UI コンポーネントライブラリです。
検証機能だけでなく、コンポーネントライブラリも併せて探されている場合はおすすめです。

#### Formwerk

[Formwerk](https://formwerk.dev/)は、Vue.js 向けのフォーム開発フレームワークです。
Vee-validate の作者の方が作成しているもので、Vee-validate では実現できなかったアクセシビリティと国際化対応を重視した設計が特徴です。

### 最後に

今回は 2025/12 時点の Vue.js で使えるフォームライブラリについて調査しました。
好みもありますが、現時点だと**暗黙的な値共有がない**、**ドキュメントが豊富**なことから Regle が有力候補かなといった印象です。
プロジェクトの技術選定の機会はあまり多いわけではないので、個人で積極的に利用して知見をためていきたいところです。
