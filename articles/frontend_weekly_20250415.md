---
title: "フロントエンドカンファレンス東京の開催決定など : Cybozu Frontend Weekly (2025-04-15号)"
emoji: "🌸"
type: "idea"
topics: ["CybozuFrontendWeekly", "frontend"]
published: true
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2025 年 4 月 15 日の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Intent to ship: Temporal

https://groups.google.com/a/mozilla.org/g/dev-platform/c/RtsRo93ygO4/m/2YzM42GUBwAJ

Temporal API が Firefox 139 でデフォルトで有効化される予定とのことです。

## [css-fonts-5] Feature for making text always fit the width of its parent #2528

https://github.com/w3c/csswg-drafts/issues/2528#issuecomment-2769621512

CSSWG で、親のコンテナサイズに応じてフォントサイズを決める仕様が策定されることが決まりました。
詳細な仕様は未定ですが、`font-size: fit(8px, 48px);` といった形式が提案されています。

## WebKit Features in Safari 18.4

https://webkit.org/blog/16574/webkit-features-in-safari-18-4/

Safari 18.4 での WebKit の新機能についてのまとめです。
CSS での `text-autospace`、View Transitions、Iterator Helpers などが追加されています。

## Biome partners with Vercel to improve type inference

https://biomejs.dev/blog/vercel-partners-biome-type-inference/

Vercel が Biome の プラチナスポンサーとなりました。
型情報に基づく Lint ルールの作成に取り組んでいるとのことです。

## Material UI v7 is here 🚀

https://mui.com/blog/material-ui-v7-is-here/

Material UI v7 がリリースされました。ESM や CSS Layers への対応が追加されています。

## How to build apps with AI

https://read.technically.dev/p/how-to-build-apps-with-ai

v0 や Vercel を使ったアプリケーション構築の流れについてのガイド記事です。

## Parcel + React Server Components

https://x.com/devongovett/status/1906097640980549789

Parcel 上で React Server Components をビルドして Cloudflare Workers で動作させるサンプルリポジトリが公開されました。

## devongovett/rsc-html-stream

https://github.com/devongovett/rsc-html-stream

RSC Payload を HTML Stream に注入するためのライブラリです。

## React 19.1.0

https://github.com/facebook/react/releases/tag/v19.1.0

React 19.1 がリリースされました。
デバッグ用の `captureOwnerStackAPI` の追加や、Suspense の改善、RSC の機能強化などが含まれます。

## VoidZero and NuxtLabs join forces on Vite Devtools

https://voidzero.dev/posts/voidzero-nuxtlabs-vite-devtools

Anthony Fu 氏が、Vite DevTools の開発に取り組むことが発表されました。

## MDN / Customizable select elements

https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Customizable_select

MDN に Customizable select elements のページが追加されました。

## Vite Dev Server に関する脆弱性パッチが公開

https://x.com/vite_js/status/1907736728628043981

Vite Dev Server に脆弱性が発見され、6.2.5、6.1.4、6.0.14、5.4.17、4.5.12 でパッチが公開されました。

## web-infra-dev/rstest

https://github.com/web-infra-dev/rstest

Rspack前提でのテストフレームワークである Rstest のリポジトリが公開されました。
ロードマップも併せて公開されています。

## Making `:visited` more private

https://developer.chrome.com/blog/visited-links?hl=en

`:visited` links history の Partitioning が Chrome 136 から Ship されることに際しての記事です。top-level site, frame origin からの移動でないと :visited が効かなくなる点などについて詳細に解説されています。

## フロントエンドカンファレンス東京2025の開催が決定

https://x.com/fec_tokyo/status/1908082128815812618

9月21日(日)に、フロントエンドカンファレンス東京2025の開催が決定しました。
詳細については後日公開されるとのことです。

## 社内デザインシステムをMCPサーバー化したらUI実装が爆速になった

https://zenn.dev/ubie_dev/articles/f927aaff02d618

Ubie 社にて、デザインシステムをMCPサーバーとして実装した事例についての紹介記事です。
Figma MCP と組み合わせることで UI を高速に実装できるとのことです。

## ESLint v9.24.0 released

https://eslint.org/blog/2025/04/eslint-v9.24.0-released/

ESLint v9.24.0 がリリースされました。新しい Lint ルールを段階的に適用していくための Bulk Suppressions の機能追加などが含まれます。

## Menu elements (Explainer)

https://open-ui.org/components/menu.explainer/

Open UI に新たに Menu コンポーネントの Proposal がマージされました。
Anchor Positioning や Popover などが積極的に利用されていたり、
キーボード操作やa11y周りについても今後策定されていくようです。

## Playwright for Browser Rendering now available

https://developers.cloudflare.com/changelog/2025-04-04-playwright-beta/

Cloudflare Workers から Playwright を実行してブラウザ自動操作を行えるようになりました。

## grafana/mcp-grafana

https://github.com/grafana/mcp-grafana

Grafana 公式の MCP サーバーが公開されました。

## TanStack Pacer の alpha リリース

https://tanstack.com/pacer/latest

TanStackから、関数の実行タイミングのみにフォーカスしたユーティリティライブラリが公開されました。

## RedwoodSDK と RedwoodGraphQL

https://x.com/RedwoodJS/status/1908164013990535470

フルスタックのJSフレームワークである RedwoodJS から、
RedwoodSDK および RedwoodGraphQL への分割が発表されました。

## Add support for installing JSR packages directly

https://github.com/pnpm/pnpm/issues/8941

pnpm の JSR パッケージサポートに関しての Issue や PR が作成され議論されているようです。

## Why We Moved off Next.js - Documenso

https://documenso.com/blog/why-we-moved-off-next-js

Documenso というプロダクトが Next.js から React Router に移行した理由について解説されたブログです。Server Actions のデバッグが困難であったり、開発体験の悪化などが原因として挙げられています。

## Better typography with text-wrap pretty | WebKit

https://webkit.org/blog/16547/better-typography-with-text-wrap-pretty/

Safari Technology Preview での `text-wrap:pretty` のリリースに伴い、
タイポグラフィ自体で伝統的にどういったものが良いとされていたかや、
`text-wrap` に指定可能な `pretty` `balance` `text-wrap` の比較や使い所について解説されています。

## フロントエンドカンファレンス北海道2025の開催が決定

https://note.com/\_n13u\_/n/n7ae05b22c09d

昨年に引き続き、9/6(土)にフロントエンドカンファレンス北海道2025の開催が決定しました。

## "Just use Vite”… with the Workers runtime

https://blog.cloudflare.com/introducing-the-cloudflare-vite-plugin/

Cloudflare のネイティブ相当の開発環境を手元で実現できる Cloudflare Vite Plugin についての解説記事です。
また、併せて React Router v7 の正式サポートも発表されています。


## あとがき

MCPの話題が多い一ヶ月でしたね！
