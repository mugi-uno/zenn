---
title: "Chakra が提供する Zag.js とは何者か"
emoji: "📈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["chakraui", "フロントエンド"]
published: false
publication_name: "cybozu_frontend"
---

# The future of Chakra UI

Chakra UI はフロントエンドにおける UI コンポーネントライブラリです。
アクセシビリティに配慮された実装になっており、実際に採用している方も多いのではないでしょうか。

https://chakra-ui.com/

そんな Chakra UI ですが、2023/3/27 に、"The future of Chakra UI" というタイトルで、Chakra UI が今後どういう方向性で進んでいくのかを紹介する記事が公開されました。

CSS の Zero runtime 化を目指す部分（通称 Panda）が特に注目されていた印象ですが、同じ記事内で、**Zag.js** というライブラリが紹介されていました。

# Zag.js？

実際のリポジトリがこちらです。

https://github.com/chakra-ui/zag

記事内では Zag.js については次のように説明されています。

> Zag.js is our low-level state machine library used to build all the components in Chakra UI. We aim to develop a robust set of application and e-commerce components that works in most JS frameworks.
>
> ChatGPT による翻訳: Zag.js は、Chakra UI のすべてのコンポーネントを構築するために使用される低レベルのステートマシンライブラリです。私たちは、ほとんどの JS フレームワークで動作する堅牢なアプリケーションと e コマースのコンポーネントセットを開発することを目指しています。

コンポーネントが必要とする状態とその管理のみを UI に依存しない形で切り出したもの、と理解すると良さそうです。

Chakra UI は React や Vue.js といった複数フレームワークへの対応コストが課題だったとのことで、
その解決のため、機能を分割し、状態管理のみを独自に切り出すというアプローチを取ることになったようです。

# Zag.js を単体で使う

Zag.js 自体は単体で提供されており、Chakra UI を介さずに利用することが可能です。
状態管理のみを Zag.js におまかせすることで、独自にアクセシブルなコンポーネントを作ることができます。
実際にやってみましょう。

## ツールチップをつくってみる

# まとめ

従来、アクセシブルな UI コンポーネントの作成と、任意のスタイリングを両立させたい場合の選択肢としては、Headless な UI コンポーネントライブラリの利用が選択肢としてありました。
[Radix](https://www.radix-ui.com/)などが有名ですね。

Zag.js の登場によって、さらに低いレイヤーでの選択肢が増えることになりました。
ユースケースとして、デザインシステムを構築する際に、状態管理部分のみを Zag.js に頼ることで、コンポーネント構造などは独自に設計しつつ、アクセシブルに構築していく、などが考えられます。

手数のひとつとして抑えておくと、役に立つときがあるかもしれませんね。

## 余談: Ark について

Chakra でも、Ark と呼ばれる Headless UI コンポーネントライブラリを提供しています。
本記事では詳細には触れませんが、こちらでは内部的に Zag.js を利用しています。
Radix 以外の選択肢として、興味があればどうぞ。

https://ark-ui.com/
