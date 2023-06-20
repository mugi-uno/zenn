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
