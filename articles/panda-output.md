---
title: "Panda CSS の出力結果から注意点を学ぶ"
emoji: "🎋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PandaCSS", "CSS", "フロントエンド"]
published: true
publication_name: "cybozu_frontend"
---

# Panda CSS は何を出力するのか？

先日 Panda CSS がリリースされ、次の紹介記事を書きました。

https://zenn.dev/cybozu_frontend/articles/panda-is-coming

その時点では、ドキュメントの内容に加えて Next.js に組み込んで動きを見てみたものをベースに書きました。ただ、実際に使うことを考えると、一体どんなファイルが出力されるのかについて理解が不足しているため、もう少し把握しておきたいところです。

ということで今回は、Panda CSS が具体的に出力するファイルの内容を覗いて、どういった点に注意すべきか見てみましょう、という記事です。

# CLI のセットアップ

出力内容を確認できれば良いので、特にその他フレームワークなどは導入せず、CLI ベースでのビルドをセットアップします。（※ドキュメント通りに進めるため詳細は割愛します）

https://panda-css.com/docs/installation/cli

## デフォルトでの出力結果を見てみる

まずは、CSS を一箇所も定義しない状態で `pnpm panda` を実行してビルドしてみます。styled-system/ が生成されるため、中身を見てみましょう。

![](/images/panda-output/plain.png)

起点となる styles.css の中身は次の通りです。

```css
@layer reset, base, tokens, recipes, utilities;

@import "./reset.css";

@import "./global.css";

@import "./tokens/index.css";

@import "./tokens/keyframes.css";
```

Panda CSS では Cascade Layers を利用し、そのために必要となるレイヤー名も一定規則に則る必要がありますが、それらが `@layer` で定義されています。また、`@import` で CSS を import しているようです。

import しているファイルはそれぞれ次の通りです。

- reset.css : リセット CSS (約 2.8KB)。margin/padding の打ち消しや box-sizing 設定など
- global.css : グローバル CSS（約 700B）。デフォルトで一部 CSS 変数の定義が含まれる。
- tokens/index.css : Design Token (約 15KB)。カラーやフォントサイズなどの CSS 変数の定義。サイズ大きめ。
- tokens/keyframes.css : アニメーションで使うための keyframe 定義（約 600B）。

> サイズを削減できる方法が無いか調べたところ、config に [optimize](https://panda-css.com/docs/references/config#optimize)や[minify](https://panda-css.com/docs/references/config#minify) といったオプションがあるようでした。しかし、試してみたところ CSS 自体の出力サイズはほぼ変化ありませんでした（内部的には postcss-normalize-whitespace が実行され、整形だけされる）。Next.js などを介して利用した場合はそちら側のビルドで minify されるためそれほど問題にはならないかもしれませんが、個別に利用する場合は少々気になる点かなと思いました。

# `css()` の出力結果

Panda におけるスタイル記述の基本となる `css()` を利用した場合の出力結果を見てみましょう。

`css()` さえ実行できれば良いので、簡単な内容とします。

```ts
import { css } from "../styled-system/css";

css({ color: "red", fontWeight: "bold" });
```

すると、Atomic CSS として、利用されているプロパティのみ出力されていることがわかります。

```css:styled-system/style.css(出力結果)
...

@layer utilities {
  .text_red {
    color: red
    }

  .font_bold {
    font-weight: var(--font-weights-bold)
    }
}
```

同じ CSS プロパティを違う箇所で重複して利用してみると、重複が排除されていることも確認できます。

```ts:CSSプロパティを重複して利用
css({ color: "red", fontWeight: "bold" });
css({ color: "red", fontWeight: "light" });
css({ color: "blue", fontWeight: "bold" });
css({ color: "blue", fontWeight: "light" });
```

```css:styled-system/style.css(出力結果)
...

@layer utilities {
  .text_red {
    color: red
    }

  .font_bold {
    font-weight: var(--font-weights-bold)
    }

  .text_blue {
    color: blue
    }

  .font_light {
    font-weight: var(--font-weights-light)
    }
}
```

では、Arbitrary Values として任意の値を指定したらどうなるでしょうか？

```ts:Arbitrary Values
css({ color: "#FF0000", fontWeight: "bold" });
css({ color: "rgb(255,0,0)", fontWeight: "light" });
```

```css:styled-system/style.css(出力結果)
...

@layer utilities {
  .text_\#FF0000 {
    color: #FF0000
    }

  .font_bold {
    font-weight: var(--font-weights-bold)
    }

  .text_rgb\(255\,0\,0\) {
    color: rgb(255,0,0)
    }

  .font_light {
    font-weight: var(--font-weights-light)
    }
}
```

単純に値がそのままクラス名として出力されるようです。"同じ値なら同じ定義" とシンプルに考えておけば良さそうです。

# Pattern の出力結果

Panda では [Pattern](https://panda-css.com/docs/concepts/patterns) という形で、頻繁に使うスタイルを定義できます。中央揃え・水平/垂直レイアウト・Flex/Grid レイアウト、といったものがあらかじめ用意されています。

Pattern は独自に作成することができますが、独自定義の Patten を利用した場合の出力結果はどうなるのでしょうか。（Atomic CSS になるのか？）

実際に確認してみましょう。テキストサイズに関する簡単な Pattern を用意してみます。

```ts:patterns/myText.ts
import { PatternConfig } from "./../styled-system/types/pattern.d";

export const myTextPattern: PatternConfig = {
  description: "My Text Pattern",
  properties: {
    big: { type: "boolean" },
  },
  transform({ big }) {
    return {
      fontSize: big ? "32px" : "16px",
      fontWeight: "bold",
    };
  },
};
```

fontSize と fontWeight をセットしてくれる Pattern です。`big` プロパティを持っており、`big` が `true` の場合は fontSize が少し大きくなります。

panda.config.mjs から読み込み、Pattern として登録しておきます。

```js:panda.config.mjs
import { defineConfig } from "@pandacss/dev";
import { myTextPattern } from "./patterns/myText";

export default defineConfig({
  patterns: {
    extend: {
      myTextPattern,
    },
  },
  ...
});
```

では利用した上で出力してみましょう。

```ts
import { myTextPattern } from "../styled-system/patterns";

myTextPattern({ big: true });
```

```css:styled-system/style.css(出力結果)
@layer utilities {
  .fs_32px {
    font-size: 32px
    }

  .font_bold {
    font-weight: var(--font-weights-bold)
    }
}
```

使っているものだけ出力されていますね！

Pattern を介した場合でも、内部的に利用している CSS プロパティが何かまで考慮してくれるようです。
`css()` で同じプロパティを利用していた場合もきちんと重複が排除されます。

```ts
import { css } from "../styled-system/css";
import { myTextPattern } from "../styled-system/patterns";

css({ fontWeight: "bold", color: "red" });
myTextPattern({ big: true });
```

```css:styled-system/style.css(出力結果)
...

@layer utilities {
  .text_red {
    color: red
    }

  .fs_32px {
    font-size: 32px
    }

  .font_bold {
    font-weight: var(--font-weights-bold)
    }
}
```

# Recipes の出力結果

Pattern に加えて、Panda では [Recipes](https://panda-css.com/docs/concepts/recipes)という方法でもスタイルのテンプレートを定義できます。Pattern はドメインに依存しない汎用的なスタイルを定義するのに使えますが、Recipes では複数の条件(`variants`)に応じて多くのスタイルを切り替えるケースで非常に便利です。

また Recipes の大きな特徴として**variants に対応する CSS をすべて出力する**という点が挙げられます。実際の利用有無に関わらず、Recipes を定義した時点ですべてが出力対象となります。

Pattern で定義した例を、似た形で今度は Recipes として定義して出力結果を見てみましょう。

```ts:recipes/myText.ts
import { cva } from "../../styled-system/css";

export const myText = cva({
  base: {
    fontWeight: "bold",
  },
  variants: {
    size: {
      big: { fontSize: "32px" },
      small: { fontSize: "16px" },
    },
  },
  defaultVariants: {
    size: "small",
  },
});
```

`myText({ size: "big" })` のような形で利用可能ですが、今回はこのまま利用箇所が存在しないままビルドしてみると、次の内容で出力されます。

```css:styled-system/style.css(出力結果)
...

@layer utilities {
  .font_bold {
    font-weight: var(--font-weights-bold)
    }

  .fs_32px {
    font-size: 32px
    }

  .fs_16px {
    font-size: 16px
    }
}
```

`variants` の `size` に定義した値がすべて出力されていますね。ここは Pattern とは大きく異なる挙動となるため、Panda を使う際は抑えておいたほうが良さそうです。

# 条件分岐を交えた出力結果

アプリケーション内で CSS フレームワークを利用する際、Props など条件に応じてスタイルを切り替えることはよくあります。そのような場合 Panda はどういった出力になるのでしょうか。

例で作成した Pattern の `big` プロパティを、コンポーネント経由で切り替えるケースで確認してみます。

```tsx:MyText.tsx
import { myTextPattern } from "../styled-system/patterns";

export const MyText = ({ big }: { big: boolean }) => {
  return <div className={myTextPattern({ big })}>SampleText</div>;
};
```

出力結果を見てみると、期待通りではない内容となります。

```css:styled-system/style.css(出力結果)
...

@layer utilities {
  .fs_16px {
    font-size: 16px
    }

  .font_bold {
    font-weight: var(--font-weights-bold)
    }
}
```

`big` プロパティに true/false のいずれが渡ってくるかは不明ですが、出力結果には `font-size: 16px` のスタイルしか含まれていません。そのため、仮に `big` プロパティに true を渡した場合、正しくスタイリングできないことになります。

## Dynamic Styling の制約

この挙動に関しては、Panda CSS のドキュメントの [Dynamic styling](https://panda-css.com/docs/guides/dynamic-styling) にも記載があります。

Panda CSS は静的解析を行った上でのビルドを前提としているため、ランタイムの値に直接依存する形で利用すると意図しない出力となる可能性があります。知らないとうっかり踏んでハマりそうですね。

## Runtime conditions

ランタイムの値を直接利用した形では静的解析時点で入りうる値が特定できませんが、記述方法によっては可能性のある分岐をすべて網羅する形で CSS を出力してくれます。

たとえば先ほどの例の場合、次のように分岐を書いた上で明示的に true or false であるように記述しておくと、それぞれの分岐に対応したスタイルが網羅的に出力されます。

```tsx:MyText.tsx
import { myTextPattern } from "../styled-system/patterns";

export const MyText = ({ big }: { big: boolean }) => {
  return (
    <div className={myTextPattern({ big: big ? true : false })}>SampleText</div>
  );
};
```

出力結果を見てみると、`big` プロパティが true の場合の `font-size: 32px` と、false の場合の `font-size: 16px` が両方出力されていることがわかります。

```css:styled-system/style.css(出力結果)
...

@layer utilities {
  .fs_32px {
    font-size: 32px
    }

  .fs_16px {
    font-size: 16px
    }

  .font_bold {
    font-weight: var(--font-weights-bold)
    }
}
```

ちなみにこれは変数として宣言しても大丈夫です。

```tsx:MyText.tsx
import { myTextPattern } from "../styled-system/patterns";

export const MyText = ({ big }: { big: boolean }) => {
  const isBig = big ? true : false; // ★変数で定義

  return <div className={myTextPattern({ big: isBig })}>SampleText</div>;
};
```

### 条件分岐を含めた解析の注意点

このような条件分岐を含めた解析を Panda に任せる方法は便利そうではありますが、少し記述を変えるだけで対応外となってしまい、意図しない出力となる可能性があります。

[ドキュメント](https://panda-css.com/docs/guides/dynamic-styling#referenced-values)上でも次の注意書きがあります。

> Here's a short list of things to avoid:
>
> - Variables that are not defined in the same file
> - Variables resulting from a function call (e.g. const color = getColor())

つまり、別ファイルで宣言された変数や関数呼び出しを介して得た値などでは解析できません。

試しに、変数宣言を別ファイルに定義し export して利用してみます。

```ts:big.ts
export const isBig = true;
export const isSmall = false;
```

```tsx:test.ts
import { myTextPattern } from "../styled-system/patterns";
import { isBig, isSmall } from "./big";

myTextPattern({ big: isBig });
myTextPattern({ big: isSmall });
```

出力結果を見てみると、パターンが網羅できていないことがわかります。

```css:styled-system/style.css(出力結果)
...

@layer utilities {
  .fs_16px {
    font-size: 16px
    }

  .font_bold {
    font-weight: var(--font-weights-bold)
    }
}
```

これは私見ですが、個人開発であれば問題ないかもしれませんが、チームで複数人で開発するケースでは、この挙動を全員が完璧に理解して使うのは少々難しいように感じます。レビューで確認しても、見落としを完全に防ぐのは難しそうです。

特に、分岐を解析できないケースでもしれっと出力されてしまうのが怖く、「動かしてみたらスタイルが出てこない！」といった事態に繋がるリスクがあるため、基本的には `css()` と Pattern 関数は極力ランタイムに影響を受けない形で実行するポリシーで利用したほうがリスクが低そうです。

なお、Recipes の出力結果でも解説しましたが、Recipes の場合は利用有無に関係なく `variants` で定義されている値はすべて出力結果に含まれます。
そのため、ランタイムの値を用いたとしても期待する CSS が出力結果に含まれていない、といった問題は発生しません。

「分岐が発生するなら Recipes」と覚えておきたいな、と思いました。
