---
title: "Panda CSS - Chakra UI の Zero Runtime CSS フレームワーク"
emoji: "🐼"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Panda", "CSS", "フロントエンド"]
published: true
publication_name: "cybozu_frontend"
---

# 🐼 パンダが来た / Panda CSS

Chakra UI から、新しい CSS フレームワークである **Panda CSS** がリリースされました。

https://panda-css.com/

2023 年 3 月に公開された Chakra UI の今後のロードマップに関するブログ内でも[すでに言及されていました](https://www.adebayosegun.com/blog/the-future-of-chakra-ui#zero-runtime-css-in-js-panda)が、今回それが正式にリリースされた形となります。

# Panda CSS の特徴

リポジトリ内 README からの抜粋となりますが、次のような特徴があります。

> ⚡️ Write style objects or style props, extract them at build time

→ Style 用のオブジェクトや Props を定義してビルドで抽出

> ✨ Modern CSS output — cascade layers @layer, css variables and more

→ Cascade Layers や CSS Variable といったモダンな CSS を用いた出力

> 🦄 Works with most JavaScript frameworks

→ JavaScript フレームワーク上での動作

> 🚀 Recipes and Variants - Just like Stitches™️ ✨

→ [Stitches](https://stitches.dev/)のような、Recipes / Variants 機能

> 🎨 High-level design tokens support for simultaneous themes

→ Design Tokens によるテーマサポート

> 💪 Type-safe styles and autocomplete (via codegen)

→ TypeSafe と オートコンプリート

昨今の開発で CSS in JS として求められる機能は一通り揃っている印象です。ドキュメント上記載のあるサポートライブラリ/フレームワークは次の通りです。

- Next.js
- Gatsby
- Solid
- Vite
- Preact
- Svelte
- Astro
- Remix
- Qwik
- Redwood
- Vue
- Storybook

Qwik・Redwood あたりまで含まれているのは手広いサポートという感じがしますね。

## 実際に試してみる

ドキュメントの Getting Started が丁寧に書かれているため、それを参考にすれば簡単に試せます。

Vite ベースのプロジェクトで試してみましょう。

https://panda-css.com/docs/getting-started/vite

```sh
cd {実験用ディレクトリ}
pnpm create vite . --template react-ts
pnpm install
```

[`panda init`](https://panda-css.com/docs/references/cli#panda-init) で設定ファイルを生成できます。

```sh
pnpm install -D @pandacss/dev
pnpm panda init --postcss
```

npm install 時やパッケージの publish 時に panda がビルドされるよう prepare コマンドを追加します。

```json:package.json
{
  "scripts": {
    "prepare": "panda codegen",
    ...
  }
}
```

レイヤーを設定するため、次の内容で src/index.css を作成します。

```css
@layer reset, base, tokens, recipes, utilities;
```

ドキュメントの手順上では認識させるファイルパターンを指定するための panda.config.ts の変更などもありますが、一旦はデフォルトのままでも大丈夫でした。

あとはコンポーネント側で実際に利用します。

```tsx:main.tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.tsx";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root") as HTMLElement).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

```tsx:App.tsx
import { css } from "../styled-system/css";

function App() {
  return (
    <div className={css({ fontSize: "2xl", fontWeight: "bold" })}>
      Hello 🐼!
    </div>
  );
}

export default App;
```

`styled-system/css` ディレクトリは、panda でビルドした結果出力されます。一度ビルドを実行してから、Vite を起動します。

```sh
pnpm panda codegen
pnpm dev
```

画面を見てみます。

![](/images/panda/init.png)

コンポーネントに記述した `fontSize` `fontWeight` が Cascade Layers を利用して適用されていますね。

とりあえず導入するだけならとても簡単でした。

# Concepts

ドキュメント上、`Concepts` としていくつか抑えておくべき Panda の概念を解説するページがあります。
Panda を理解する上で重要そうなので、ひとつずつ確認してみます。

## Cascade Layers / レイヤー

https://panda-css.com/docs/concepts/cascade-layers

Panda CSS では、Cascade Layers を利用して CSS の優先順位を制御しています。つまり、利用可能なブラウザは Cascade Layers をサポートしている必要があるため注意が必要となりそうです。mdn によると、メジャーブラウザでは次のバージョンが必要です。

- Chrome ≧ 99
- Edge ≧ 99
- Firefox ≧ 97
- Safari ≧ 15.4

https://developer.mozilla.org/ja/docs/Web/CSS/@layer

レイヤーは次の５つで固定で、上にあるもののほうが優先度が低いようです。

- `reset` : Reset CSS。設定で `preflight` を有効にすると出力される（デフォルトでは true）
- `base` : グローバル CSS
- `tokens` : Design Token として定義した CSS 変数
- `recipes` : "Recipes" として定義された値（後述）。例として "ボタンのスタイル" など
- `utilities` : 個々に限定的に定義した CSS（後述）

なお、レイヤーの優先順については、先述の Vite での例を見ると index.css などで `@layer` で利用側で定義する形のため、やろうと思えば順序の入れ替えもできそうです。ただ、可能だとはいえさすがに避けたほうが良さそうです。

Cascade Layers のおかげで、リセット CSS やグローバル CSS を上書きできない！といった問題は発生しづらそうですね。

## Writing Styles / css()での記述とアトミック CSS

Panda を使う上で一番単純な CSS 適用方法が、`css()` 関数を利用してオブジェクト記法で記述していくものでしょう。Type Safe に取り扱え、VSCode などでの補完もバッチリです。

![](/images/panda/atomic.png)

この利用方法に関しては Atomic CSS として取り扱われます。CSS プロパティに対して個々に class などが割り振られるため、同じスタイルを複数箇所で適用しても CSS サイズが肥大化しないといった恩恵が得られます。

実際次のようなコンポーネントを作ってビルドしてみると確認できます。

```tsx
<div>
  <div className={css({ color: "red" })}>red</div>
  <div className={css({ fontWeight: "bold" })}>bold</div>
  <div className={css({ color: "red", fontWeight: "bold" })}>red bold</div>
</div>
```

生成された CSS では、利用しているもののみが出力され、かつ重複が排除されています。

```css
/* 確認のため整形しています */
@layer utilities {
  .text_red {
    color: red;
  }
  .font_bold {
    font-weight: var(--font-weights-bold);
  }
}
```

Tailwind CSS で得られる恩恵に近いものを得ることができる、と思うと理解しやすいかもしれません。実際 Tailwind は ユーティリティファーストな CSS フレームワークと説明されることもありますが、Panda で `css()` を利用して記述したスタイルは `utilities` レイヤーに出力されるため、思想のコアな部分は近いと思って良さそうです。Panda の場合はそれをさらに型安全に利用できるのが嬉しいですね。

## Conditional Styles / 条件付きスタイリング

ホバーやフォーカスなど、CSS を特定の条件のときのみ適用する、というのは避けては通れません。Panda でも、擬似クラスなどは一通りサポートしているようです。

基本的には対応するプロパティに定義すれば OK なようです。

```tsx
<button className={css({ color: "red", _hover: { color: "blue" } })}>
  Button
</button>
```

![](/images/panda/button.gif)

[ネストして記述](https://panda-css.com/docs/concepts/conditional-styles#nested-condition)することもでき、「ホバーかつフォーカスしているとき」なども簡単に表現できそうです。

他にも、Media Queries に対応していたり、data 属性を利用して独自の条件スタイリングを行ったりもできるようです。

## Patterns / パターン

CSS を書いてると「よくあるスタイル」が頻出することがありますが、それらを **Patterns** として再利用できます。独自で定義することもできますが、一般的に利用されがちなものはデフォルトで用意されています。

たとえば、要素を中央揃えにするには `center()` Pattern が利用できます。

```tsx
import { center } from "../styled-system/patterns";

function App() {
  return (
    <div
      className={center({
        bg: "gray",
        color: "white",
        inlineSize: "200px",
        blockSize: "200px",
      })}
    >
      text
    </div>
  );
}
```

![](/images/panda/pattern.png)

`css()` 関数と同様のプロパティを任意に定義できているのもポイントです。中央揃えという前提を継承しつつ、任意でカスタマイズできます。

バージョン 0.3.2 の時点では次の Pattern がデフォルトで利用できるようです。

- `container`: 最大幅を持ち中央揃えのコンテナ
- `stack`: 垂直または水平方向のスタックレイアウト
- `hstack`: 水平方向のスタックレイアウト
- `vstack`: 垂直方向のスタックレイアウト
- `wrap`: 要素間のスペース確保と自動折返し
- `aspectRatio`: アスペクト比の指定 (`aspectRatio` CSS プロパティの利用の方が推奨されるようです)
- `flex` : Flex レイアウト
- `center`: 中央揃え
- `float` : Float によるレイアウト
- `grid` : Grid レイアウト
- `gridItem` : Grid コンテナ内の子要素のスタイリング
- `divider` : 区切り
- `circle` : 円の作成
- `square` : 正方形の作成

Pattern を独自定義する際には、デフォルトの Pattern を継承したり、拒否するプロパティを指定したりできます。それほど難しくないため、興味があればドキュメントを確認してみてください。

https://panda-css.com/docs/customization/patterns

## Recipes / レシピ

「ボタン」「メニュー」「ダイアログ」といったコンポーネントのスタイリングなど、特定の用途のパーツに関しての基本的なスタイルは共通化を行いたいケースが多いです。Design System を構築するケースなどでは必須になるでしょう。

こういった場合、Panda では Recipes という形でスタイルを定義できます。サイズとタイプをそれぞれ２パターン持つボタンのスタイルを定義してみましょう。

```ts:button.css.ts
import { cva } from "../styled-system/css";

export const button = cva({
  base: {
    display: "flex",
    borderWidth: "1px",
    borderColor: "gray",
  },
  variants: {
    type: {
      default: { color: "gray" },
      danger: { color: "red", borderColor: "red" },
    },
    size: {
      small: { padding: "8px", fontSize: "12px" },
      large: { padding: "16px", fontSize: "16px" },
    },
  },
  defaultVariants: {
    type: "default",
    size: "small",
  },
});
```

タイプとして "default" と "danger" の２つと、サイズも "small" と "large" の２つを定義しています。指定しなかった場合のデフォルト値は タイプが "default" でサイズは "small" になるよう定義しています。なお、もちろんこれらの定義もすべて型安全に書いていけます。

![](/images/panda/recipe-safe.png)

実際にコンポーネントから利用してみます。

```tsx
import { hstack } from "../styled-system/patterns";
import { button } from "./button.css";

function App() {
  return (
    <>
      <div className={hstack({ gap: "8px", padding: "16px" })}>
        <button className={button({ size: "small", type: "default" })}>
          Button
        </button>
        <button className={button({ size: "large", type: "default" })}>
          Button
        </button>
        <button className={button({ size: "small", type: "danger" })}>
          Button
        </button>
        <button className={button({ size: "large", type: "danger" })}>
          Button
        </button>
        <button className={button()}>Button</button>
      </div>
    </>
  );
}
```

![](/images/panda/recipe.png)

それぞれのプロパティが適用されてスタイリングできていますね。

なお、DesignSystem で作成したコンポーネントなどで利用する場合には、実際にはこれらの値は Props として受け取りたいケースが出てくると思います。そういった場合でも、`RecipeVariantProps` を利用することで型だけ抽出することができます。

```tsx
import { ReactNode } from "react";
import { RecipeVariantProps } from "../styled-system/css";
import { button } from "./button.css";

type Props = {
  children: ReactNode;
} & RecipeVariantProps<typeof button>;

export const Button = ({ children, ...recipeVariantProps }: Props) => {
  <button className={button(recipeVariantProps)}>{children}</button>;
};
```

```tsx
import { Button } from "./Button";

function App() {
  return (
    <Button size="small" type="default">
      Button
    </Button>
  );
}
```

スタイリングに必要な分岐をきれいに Panda での定義側にすべて分離できるのが嬉しいポイントですね。

## JSX Style Props / JSX との統合

ここまで紹介したものはすべて基本的に Panda 経由で生成された class を`className` に適用する形でスタイリングを行っていました。加えて、Panda ではそれらすべてを JSX のプロパティとしても利用することができます。

利用するには、panda.config.ts に jsxFramework オプションを追加し、対応するフレームワークを指定します。

```ts:panda.config.ts
export default defineConfig({
  ...
  jsxFramework: 'react'
})
```

すると、`styled.xxx` の形で、任意の JSX 要素を作成可能になります。また、Patterns に関しても同名のコンポーネントが利用できます。

```tsx
import { VStack, styled } from "../styled-system/jsx";

function App() {
  return (
    <VStack gap="8px">
      <styled.a href="https://example.com" color="red">
        Link
      </styled.a>
      <styled.button type="button" color="blue">
        Button
      </styled.button>
    </VStack>
  );
}

export default App;
```

`<styled.a>` などを見てみると、`<a>` 要素としての `href` を受け取りつつ、スタイル用の `color` も受け取れていますね。実際画面で確認してみても、正しく `<a>` タグで描画されつつスタイルが適用されています。

![](/images/panda/jsx1.png)

また、`styled` 自体を関数としても実行可能で、それを利用して JSX スタイルでの Recipes を作成することもできます。さきほどの Recipes の例で作成した Button コンポーネントとまったく同様のものは、次の形で定義できます。

```tsx:Button.tsx
import { styled } from "../styled-system/jsx";

export const Button = styled("button", {
  base: {
    display: "flex",
    borderWidth: "1px",
    borderColor: "gray",
  },
  variants: {
    type: {
      default: { color: "gray" },
      danger: { color: "red", borderColor: "red" },
    },
    size: {
      small: { padding: "8px", fontSize: "12px" },
      large: { padding: "16px", fontSize: "16px" },
    },
  },
  defaultVariants: {
    type: "default",
  },
});
```

この場合は最初からコンポーネントとして成立しているため、`RecipeVariantProps` を利用しなくても Props も最初から適切に受け取ることができます。

Chakra UI などではスタイリング用のプロパティをコンポーネントで受け取る形になっていますが、それに近いことを簡単に独自のコンポーネントで実現できるイメージです。

# 所感

触ってみた感触としては、かなり体験が良かったです。個人的に Tailwind は好きな方でしたが、よく問題点として挙げられる "class 定義が非常に煩雑になる" "型安全ではない" といった点は気になるポイントでした。Panda CSS ではそれらの課題を解決しつつも、Atomic CSS の恩恵を受けられるのは大きいメリットに感じました。この記事では紹介していませんが、デフォルトで [Design Tokens の定義もサポート](https://panda-css.com/docs/theming/tokens)しているのも大きく、Design System を構築する際などにも恩恵を受けられそうです。
また、className に独自で付与する方法と JSX スタイルで記述する方法のどちらも利用できるのはとても嬉しいポイントに感じます。チームの好みによって使い分けることができますし、元々使っていたフレームワークに応じて、体験が近い方を選択できるのはとても良さそうです。

一方で、独自での記法になってしまうのは避けられない点です。型安全にする以上はやむを得ない部分になると思っていますが、できる限り純粋な CSS に近い形で書くのが好みな場合は適さず、CSS Modules などを利用したほうが望ましいかもしれません。

これは完全に個人の意見ですが、これから採用するプロダクトは増えていくのではないかなと思いました。がんばれパンダ！
