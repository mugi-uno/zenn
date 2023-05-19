---
title: "Chakra が提供する Zag.js でアクセシブルなコンポーネントを自由に作る"
emoji: "📈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["chakraui", "フロントエンド"]
published: true
publication_name: "cybozu_frontend"
---

# The future of Chakra UI

Chakra UI はフロントエンドにおける UI コンポーネントライブラリです。
アクセシビリティに配慮された実装になっており、実際に採用している方も多いのではないでしょうか。

https://chakra-ui.com/

そんな Chakra UI ですが、2023/3/27 に、"The future of Chakra UI" というタイトルで、Chakra UI が今後どういう方向性で進んでいくのかを紹介する記事が公開されました。

https://www.adebayosegun.com/blog/the-future-of-chakra-ui

CSS の Zero runtime 化を目指す部分（通称 Panda）が特に注目されていた印象ですが、同じ記事内で、**Zag.js** というライブラリが紹介されていました。

# Zag.js？

実際のリポジトリがこちらです。

https://github.com/chakra-ui/zag

記事内では Zag.js については次のように説明されています。

> Zag.js is our low-level state machine library used to build all the components in Chakra UI. We aim to develop a robust set of application and e-commerce components that works in most JS frameworks.
>
> ChatGPT による翻訳: Zag.js は、Chakra UI のすべてのコンポーネントを構築するために使用される低レベルのステートマシンライブラリです。私たちは、ほとんどの JS フレームワークで動作する堅牢なアプリケーションと e コマースのコンポーネントセットを開発することを目指しています。

コンポーネントが必要とする状態とその管理のみを UI に依存しない形で切り出したもの、と理解すると良さそうです。

Chakra UI は React や Vue.js といった複数フレームワークへの対応コストが課題だったとのことで、その解決のため、機能を分割し、状態管理のみを独自に切り出すというアプローチを取ることになったようです。

# Zag.js を単体で使う

Zag.js 自体は BETA バージョンではありますが、単体で提供されており、Chakra UI を介さずに利用することが可能です。状態管理のみを Zag.js におまかせすることで、独自にアクセシブルなコンポーネントを作ることができます。

実際にやってみましょう。

## 独自の Popover コンポーネントをつくってみる

この記事の執筆時点（2023 年 5 月）の段階では、Zag.js では次のコンポーネントがサポートされています。※ドキュメントの [Components](https://zagjs.com/overview/introduction) に列挙されています。

- Accordion
- Checkbox
- Combobox
- Dialog
- Editable
- Hover Card
- Menu
- Context Menu
- Nested Menu
- Number Input
- Pagination
- Pin Input
- Popover
- Pressable
- Radio Group
- Range Slider
- Rating
- Select
- Slider
- Switch
- Tabs
- Tags Input
- Toast
- Tooltip

独断と偏見により、Popover の実装をしてみましょう。

## セットアップ

[Getting Started](https://zagjs.com/overview/installation) の通りセットアップしましょう。（その他の環境は、`npm create vite@latest` で実験用に React 向け環境を構築しています。）

まず、コアとなるコンポーネントに対応するステートマシンパッケージを追加します。

```
npm install @zag-js/popover
```

コアパッケージ自体は UI フレームワークに依存しない形で実装されていますが、Zag.js では、React・Vue.js・Solid.js についてはアダプタを提供しています。

今回は React 用のアダプタを追加します。

```
npm install @zag-js/react
```

## Popover コンポーネントを作る

ドキュメントの Popover のページの説明をベースに `<MyPopover>` コンポーネントを定義します。

```tsx
import * as popover from "@zag-js/popover";
import { normalizeProps, useMachine } from "@zag-js/react";
import { useId } from "react";

export function MyPopover() {
  const [state, send] = useMachine(popover.machine({ id: useId() }));

  const api = popover.connect(state, send, normalizeProps);

  return (
    <div>
      <button {...api.triggerProps}>ひらく</button>
      <div {...api.positionerProps}>
        <div {...api.contentProps} style={{ border: "1px solid gray" }}>
          <div {...api.titleProps} style={{ fontWeight: "bold" }}>
            タイトルです
          </div>
          <div {...api.descriptionProps}>詳細です</div>
          <button {...api.closeTriggerProps}>とじる</button>
        </div>
      </div>
    </div>
  );
}
```

これを画面で描画して見てみると...

![](/images/chakra-zagjs/popover.gif)

見た目は簡素ですが、なんだかもう Popover としては動いてますね。完成です、ありがとうございました。

フォーカス制御も行われており、アクセシビリティ用の aria 属性も付与されています。

![](/images/chakra-zagjs/a11y-tree.png)

![](/images/chakra-zagjs/a11y-element.png)

アクセシブルな UI コンポーネントをきちんと実装するのは、なかなか大変な作業となります。フォーカス制御などで苦しんだ方も多いのではないでしょうか。そのあたりがサクっとできてしまうのはすごいですね...

タグは単純な `<button>` や `<div>` であり、かつ Zag.js では CSS は提供しないため、自由にスタイリングできます。今回のサンプルでは、独自で付与している `style` 属性のスタイルのみが適用されています。

では、コンポーネントのコードを詳細に見てみましょう。

## コードをみてみる

### ステートマシンの生成

```tsx
const [state, send] = useMachine(popover.machine({ id: useId() }));
```

`machine()` で、Popover 用のステートマシンを生成しています。その後、アダプタ (`useMachine`) を介して React 上の State として取り扱いができる形としています。

### プロパティの取得と各フレームワーク向けの変換

```tsx
const api = popover.connect(state, send, normalizeProps);
```

`connect()` によって、DOM 要素に適用できる形に変換しています。

なお、`connect()` そのものは各フレームワークは意識しない形に変換しますが、一部プロパティはフレームワークに適した形にリネームする必要があります。たとえば`for` 属性は、React の場合は `htmlFor` とする必要がありますが、Vue では `for` のままで OK だったりといった差異があります。このあたりは、各フレームワークのアダプタが提供する `normalizeProps` を引数で渡すことで変換されています。

詳細はドキュメントに説明があります。

https://zagjs.com/overview/faq#why-the-need-for-normalizeprops

ちなみに、`normalizeProps` のコードを実際に見てみると、React 向けのアダプタでは何も変換をしていないようでした。

https://github.com/chakra-ui/zag/blob/main/packages/frameworks/react/src/normalize-props.ts#L9

おそらく、デフォルトでは React 向けのプロパティを生成しているのかもしれません。

### DOM への適用

`connect()` で得られた値は、そのままプロパティとして適用可能な形になっています。

どういったプロパティが得られるかは、ドキュメントの [`Anatomy`](https://zagjs.com/components/react/popover#anatomy) を見ると理解しやすいです。

Popover の場合、次のような要素に対応するプロパティが得られます。

- Trigger: Popover を表示するためのトリガー要素
- Positioner: Popover のポジショニングをするための要素
- Content: Popover のコンテンツ
- Title: Popover のタイトル
- Description: Popover の詳細
- Close Button: Popover を閉じるためのボタン

つまり、次のコードでは、`<button>` 要素に対して Trigger として振る舞うために必要なプロパティを指定しています。

```tsx
<button {...api.triggerProps}>Click me</button>
```

実際に `api.triggerProps` の中身を見てみると、`<button>` 要素に適用可能な aria 属性やイベントハンドラなどが得られることがわかります。

![](/images/chakra-zagjs/a11y-prop.png)

単純に Rest Spread として渡しているだけなので、必要に応じて一部プロパティを上書きすることも容易そうですね。

# まとめ

従来、アクセシブルな UI コンポーネントの作成と、任意のスタイリングを両立させたい場合の選択肢としては、Headless な UI コンポーネントライブラリの利用が選択肢としてありました。
[Radix](https://www.radix-ui.com/)などが有名ですね。

Zag.js の登場によって、さらに低いレイヤーでの選択肢が増えることになりました。
ユースケースとして、デザインシステムを構築する際に、状態管理部分のみを Zag.js に頼ることで、コンポーネント構造は独自に設計しつつ、アクセシブルに構築していく、などが考えられます。

手数のひとつとして抑えておくと、役に立つときがあるかもしれませんね。

## 余談: Ark について

Chakra でも、Ark と呼ばれる Headless UI コンポーネントライブラリを提供しています。本記事では詳細には触れませんが、こちらでは内部的に Zag.js を利用しています。Radix 以外の選択肢として、興味があればどうぞ。

https://ark-ui.com/
