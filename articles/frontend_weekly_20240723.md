---
title: "3rd Party Cookie 廃止の方針変更など : Cybozu Frontend Weekly (2024-07-23号)"
emoji: "🛟"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: true
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2024 年 7 月 23 日 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Temporal を取り巻く仕様を整理する

https://speakerdeck.com/sajikix/temporalwoqu-rijuan-kushi-yang-wozheng-li-suru

ECMAScript Stage 3 の Temporal に関する発表資料です。仕様そのものの整理や、タイムゾーン・カレンダーのサポート、Intl との関係性について解説されています。

## Fastly が開発者向けの無料プランを提供開始

https://www.publickey1.jp/blog/24/fastlycdnddoswasmkv.html

Fastly が開発者向けに無料プランの提供を開始しました。エッジランタイムや、KV ストア、といった各種機能が利用可能です。

## Chrome DevTools で表示されるエラーを生成 AI の Gemini が解説してくれる新機能、日本でも利用可能に

https://www.publickey1.jp/blog/24/chrome_devtoolsaigemini.html

Chrome DevTools でのエラーメッセージの Gemini による解説機能について、日本でも機能提供が開始されました。

## react-big-calendar のコアが TanStack/time になるかも

https://github.com/jquense/react-big-calendar/issues/2255

リポジトリだけ作成されている状況の [TanStack/time](https://github.com/TanStack/time)ですが、react-big-calendar のコアで利用する形で検討が進められているようです。

## MS が WebView2 の Mac・Linux 版の公開中止を発表

https://github.com/MicrosoftEdge/WebView2Feedback/issues/1314#issuecomment-2211683486

OS 提供の WebView を抽象化する層として MS が開発していた WebView2 について、Mac・Linux 版のパブリック向けの公開が中止されることがアナウンスされました。

## Deno 1.45: Workspace and Monorepo Support

https://deno.com/blog/v1.45

Deno 1.45 がリリースされ、Workspace と Monorepo がサポートされました。
社内のメンバーが実際に試したところ、現状では VSCode 連携などで一部問題もあるようですが、CLI で操作する限りでは一般的な Monorepo としての機能は問題なく利用できたようです。

## Private Browsing 2.0 | WebKit

https://webkit.org/blog/15697/private-browsing-2-0/

Safari がプライベートブラウジングに関してこれまで強化してきた機能をふりかえる記事です。
Link Tracking Protection、Advanced Fingerprinting Protection、などに加え、Topics API の問題点の指摘なども含まれています。

## Node v22.5.0 のリリース

https://nodejs.org/en/blog/release/v22.5.0

Node.js v22.5.0 がリリースされました。`node:sqlite` が追加されており、SQLite の操作が標準で行えるようになりました。

## A new path for Privacy Sandbox on the web

https://privacysandbox.com/news/privacy-sandbox-update/

Google が、廃止に向けて進めてきた 3rd Party Cookie ですが、方針が切り替わり、ユーザーの選択によってプライバシー設定を管理する仕組みを Chrome に導入することになったようです。

## TypeSpec、Orval、Storybook を使ってフロントエンドのモック生成を自動化する

https://zenn.dev/funteractiveinc/articles/63ace4f19ba2b9

TypeSpec を用いて OpenAPI スキーマを作成 → Orval で API クライアントとモックを生成 → Storybook で利用、というフローを解説している記事です。

## Astro 4.12 のリリース

https://astro.build/blog/astro-4120/

Astro 4.12 がリリースされました。Experimental な機能として、動的にバックエンド側で描画する部分を設ける Server Islands が導入されています。

## あとがき

Google の 3rd Party Cookie の方針転換はビッグニュースですね…。今後の動向が注目されます。
