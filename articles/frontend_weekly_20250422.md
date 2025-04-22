---
title: "Introducing Zod 4 beta など : Cybozu Frontend Weekly (2025-04-22号)"
emoji: "🌷"
type: "idea"
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2025 年 4 月 22 日の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## あらためて理解する ArrayBuffer - JavaScript でバイナリデータを扱う方法

https://ics.media/entry/250408/

JavaScript における ArrayBuffer や TypedArray がなぜ必要なのかや、利用方法、ユースケースなどについて詳細に解説した記事です。

## web-platform-dx/browserslist-config-baseline

https://github.com/web-platform-dx/browserslist-config-baseline

ブラウザのバージョン設定を共通化するための Browserslist 向けに、Baseline に対応したコンフィグが公開されました。

## Introducing Zod 4 beta

https://v4.zod.dev/v4

Zod の v4 beta がリリースされました。内部のアーキテクチャが一新され、多くの問題が解決されているとのことです。
同時に、Tree-shaking に最適化された `@zod/mini` なども公開されています。

## Next.js 15.3

https://nextjs.org/blog/next-15-3

Next.js 15.3 がリリースされました。Turbopack による Build が追加されました。また、experimental ですが、Rspack によるビルドもサポートされました。

## microsoft/vscode-css-languageservice での Baseline サポート

https://github.com/microsoft/vscode-css-languageservice/releases/tag/v6.3.4

VSCode のビルトインの CSS Language Service で、Baseline のステータスをホバーカードで確認できる機能が追加されました。

## Brave Software Asia、日本のユーザーに向けた新たな取り組みを開始

https://brave.com/ja/blog/japan-search-announce/

Brave ブラウザと LINE ヤフーのサービスの連携が強化される旨が発表されました。

## What is a Chrome Finch experiment?

https://developer.chrome.com/docs/web-platform/chrome-finch

Chrome での機能リリース時に、機能動作・コンプライアンス・信頼性などの検証のために利用されている「Finch」と呼ばれるテストについての解説記事です。

## Google が「google.co.jp」の使用を停止、その他国別トップレベルドメインも「google.com」にすべてリダイレクトする処理へ

https://gigazine.net/news/20250416-google-cctld-redirect

Google が国別のトップレベルドメインの利用を廃止、今後は google.com へリダイレクトを開始することを発表しました。

## Claude takes research to new places

https://www.anthropic.com/news/research

Claude における新機能である、Research や Google Workspace 連携などに関しての公式の解説記事です。

## Faster Lazy Loading in React Router v7.5+

https://remix.run/blog/faster-lazy-loading

React Router 7.5 に含まれる route.lazy による遅延ロードの改善についての記事です。

## The cost-effective promise of Model Context Protocol (MCP)

https://www.epicai.pro/the-cost-effective-promise-of-model-context-protocol-mcp-jnwt3

Kent C. Dodds 氏による、MCP におけるコスト面での優位性についての記事です。

## JSX Over The Wire — overreacted

https://overreacted.io/jsx-over-the-wire/

RSC のような Component を返す API の概念と、歴史的どのようなアプローチがあったのか、などについて解説された記事です。

## Vitest の実行時間を 8 倍高速化：同一環境での実行によるパフォーマンス改善

https://zenn.dev/microcms/articles/c3b9d48b5647b4

microCMS における Vitest の実行時間の改善事例です。
実際に発生していた問題と、それぞれどう対処したのかについて解説されています。

## あとがき

Zod 4 の印象が強かったです。MCP 実装でもよく見かけるため、チェックしておくと良いかもしれません！
