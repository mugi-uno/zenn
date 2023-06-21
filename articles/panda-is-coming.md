---
title: "Panda CSS - Chakra UI の Zero Runtime CSS フレームワーク"
emoji: "🐼"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Panda", "CSS", "フロントエンド"]
published: false
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
