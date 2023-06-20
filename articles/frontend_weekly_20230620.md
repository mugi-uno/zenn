---
title: "Panda CSS リリースなど : Cybozu Frontend Weekly (2023-06-20号)"
emoji: "☂️"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: true
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2023/06/20 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## TypeScript ESLint が Project Governance を公開した

https://github.com/typescript-eslint/typescript-eslint/discussions/7103

プロジェクトへの貢献のレベルやそれに応じた報酬などが明確に定義されています。貢献の種類に応じてポイントが割り振られ、保有しているポイントに応じて報酬や、逆に期待される稼働率などが記載されています。似た事例では、[ESLint](https://eslint.org/docs/latest/contribute/governance) や [Astro](https://github.com/withastro/.github/blob/main/GOVERNANCE.md) などでも Governance を公開しているようです。

## Panda CSS

https://panda-css.com/

Chakra が提供する Zero Runtime CSS フレームワークの Panda CSS が公開されました。TypeSafe であり、Next.js App Router などにも対応されているようです。

## Google I/O 2023 の振り返りオフラインイベントが開催される

https://developersonair.withgoogle.com/events/ioextendedjapan2023

2023/7/11 に、Google I/O 2023 終了後の振り返りイベントが開催されるとのことです。

## Fresh 1.2 のリリース

https://deno.com/blog/fresh-1.2

Deno の Web フレームワークである Fresh の 1.2 がリリースされました。islands コンポーネントの Props に Preact の Signals を渡せるようになったり、islands にサブディレクトリが作れるようにといった機能追加が含まれます。加えて、フルタイムのメンテナとして [Marvin Hagemeister 氏](https://marvinh.dev/)が加わったことも発表されています。

## W3C の新しいサイトがリリース

https://twitter.com/w3c/status/1670828463022915593

## Deno KV を使ったハッカソンが開催

https://deno.com/blog/deno-kv-hackathon

Deno KV を使ったハッカソンが開催され、29 の成果物が応募されたそうです。

## State of CSS 2023 のサーベイが開始

https://survey.devographics.com/en-US/survey/state-of-css/2023

毎年恒例の State of CSS 2023 のサーベイが開始されました。知らなかった CSS 機能を知る機会にもなるため、登録しておくとよいかもしれません。

## なぜ div に onClick がダメなのか？

https://zenn.dev/tm35/articles/64eac4c0570c4d

div タグに onClick を付与してはいけない理由を、a11y などの観点から解説されています。

## Chrome で開発中の Topics API の改善

https://developer.chrome.com/blog/topics-enhancements/

ブラウザ利用者の関心ジャンルを得るための オリジントライアル中の API である Topics API についての改善情報の記事です。Topics API は 広告の最適化に利用されます。ジャンルの分類の見直し・トピックの興味取得の条件の緩和・ユーザーによるブロック制御・パフォーマンス向上、といった内容が含まれます。

## Wasmer Edge

https://wasmer.io/posts/announcing-wasmer-edge

WebAssembly・Wasmer を利用したアプリケーションの分散デプロイ可能なプラットフォームです。既存のクラウドサービスとの比較を踏まえたメリットも記事内で解説されています。現状は Rust ベースのアプリケーションのみ対応していますが、将来的には Flask・Django・WordPress・Rails・Node などにも対応予定とのことです。

## clip-path での三角形の作り方

https://qiita.com/degudegu2510/items/09f34d4b218c9df6bb57

古くは CSS で三角形を作る際に border を利用するテクニックが利用されていましたが、現在は clip-path で作れますよ、という紹介記事です。

## tremor

https://www.tremor.so/

React 製の Dashboard Library です。Chart 表示にも複数対応しており、従来[React Charts](https://react-charts.tanstack.com/)を使っていたケースなどでの新しい選択肢にもなりそうです。
