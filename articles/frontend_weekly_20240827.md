---
title: "フロントエンドカンファレンス北海道開催など : Cybozu Frontend Weekly (2024-08-27号)"
emoji: "🐻"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: true
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2024 年 8 月 27 日 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Boosting performance: Faire’s transition to NextJS

https://craft.faire.com/boosting-performance-faires-transition-to-nextjs-3967f092caaf

https://www.faire.com/ が React Router から Next.js App Router へ移行した際の解説記事です。Remix や Fresh/Deno なども検討した結果、スピードやフレームワークの成熟度、移行作業のボリュームや開発者経験などから Next.js の採用に至ったようです。
具体的に改善されたパフォーマンスについても言及されており、結果として離脱率の減少にも繋がったようです。

## W3C opens community-wide survey | 2024 | News | W3C

https://www.w3.org/news/2024/w3c-opens-community-wide-survey/

W3C がコミュニティ全体に関連するサーベイを実施するとのことです。誰でも匿名で回答が可能です。

## Intent to Ship: FedCM (was WebID)

https://groups.google.com/a/chromium.org/g/blink-dev/c/URpYPPH-YQ4

先日 3rd Party Cookie の方針転換がアナウンスされましたが、それを受けて、Google Chrome の FedCM のメンバーが FedCM 自体の継続についてアナウンスを追加しました。

## State of React 2023

https://stateofreact.com/en-US

State of React 2023 サーベイの結果が公開されました。Pain Point として挙げられているものが React v19 の追加要素として解消されていたり、UI Component Library 周りでは Headless 系のものが特に関心が強いものとして挙げられているようです。

## I'm Tired of Node Builtin APIs - Bjorn Lu

https://bjornlu.com/blog/im-tired-of-node-builtin-apis

Node.js において、テストランナーや CLI 実行時の引数のパーサなど、元々は外部パッケージに頼るケースの多かった機能が標準でサポートされつつあります。それらについて、広く利用されているパッケージとの使用感の違いなどについて言及されているブログ記事です。

## ジャンプ TOON Web アプリケーションの全体像〜採用技術と開発方針〜

https://developers.cyberagent.co.jp/blog/archives/49294/

https://jumptoon.com/ で Next.js App Router を採用して開発した際の知見についての記事です。Next.js App Router が現状抱えている課題や、それに対してどう対処したかについても触れられています。また、SNS 上では記事内容を受けて、App Router と GraphQL の関係性について言及している方も多かった印象です。

## Rolldown の開発状況に関して

https://x.com/youyuxi/status/1824023061714399292

Vue で将来的に利用される予定のバンドラである Rolldown の開発状況について 開発者である Evan you 氏 よりアナウンスがありました。Rolldown でバンドルしたパッケージが Vue エコシステムのテストをパスしているなど、開発は順調なようです。Vite への統合も進んでおり、ViteConf で詳細が発表されるそうです。

## Now in Baseline: animating entry effects

https://web.dev/blog/baseline-entry-animations

トランジション開始時のプロパティを指定するための CSS での @ rule である `@starting-style` が Baseline Newly Available となり、全ブラウザで利用可能となりました。弊社フロントエンドエキスパートチームのメンバーである BaHo が別途解説記事も書いているため、そちらも参考になります。

https://zenn.dev/cybozu_frontend/articles/20240812_starting-style

## Announcing Official Puppeteer Support for Firefox – Mozilla Hacks - the Web developer blog

https://hacks.mozilla.org/2024/08/puppeteer-support-for-firefox/

puppeteer が v23 で Firefox をサポートしました。従来では Chrome Devtools Protocol を用いた実装でしたが、W3C で標準化が進められている WebDriver BiDi が採用されている点などについても解説されています。

## Import Assertions が Chrome 126 から取り除かれた

https://developer.chrome.com/release-notes/126?hl=ja#dreprecate_and_remove_import_assertion_assert_syntax

import の後ろに `assert` を付与しインポートするファイルタイプを付与する構文について、実際にはアサーション以外の機能が必要となったことなどから `with` キーワードがサポートされ、最終的には Chrome 126 で `assert` が削除されました。

これを受けて test262 からも削除されるようです。

https://github.com/tc39/proposal-import-attributes/pull/161
https://github.com/tc39/test262/pull/4203

## Server actions are here! - Waku

https://waku.gg/blog/server-actions-are-here

React Framework である Waku が Server Actions をサポートしたことを紹介する公式アナウンス記事です。

## Next.js の考え方

https://zenn.dev/akfm/books/nextjs-basic-principle

Next.js App Router においての設計のベストプラクティスについてまとめられた Zenn Book です。mugi 個人的にも App Router を学ぶ際にはとりあえず読んでおくことをオススメします。

## フロントエンドカンファレンス北海道 2024 公開資料・X アカウントリンクまとめ

https://zenn.dev/yumemi_inc/articles/2024-08-25-frontend-conf-hokkaido-2024

2024/8/24 に開催されたフロントエンドカンファレンス北海道 2024 での公開資料についてまとめられている記事です。弊社からも複数名登壇しているので、ぜひチェックしてみてください。

https://speakerdeck.com/ryo_manba/5fen-defen-karu-react-aria-no-liang-itokorokorekaranatokoro
https://speakerdeck.com/sajikix/the-future-of-frontend-i18n-intl-dot-messageformat
https://speakerdeck.com/sakito/component-driven-design-and-development
https://speakerdeck.com/mugi_uno/new-order-in-cascade-sorting-order

## Deno 1.46: The Last 1.x Release

https://deno.com/blog/v1.46

Deno 1.46 がリリースされました。

CLI 操作の簡潔化、ヘルプや進捗状況の表示の改善、`deno serve` のマルチスレッド化、`deno fmt` での HTML・CSS・YAML などのサポート追加、Node.js・npm との互換性の向上、パフォーマンスの改善、といった多くの変更が含まれます。

なおこれは 1.x 系で最後のリリースとなり、近日中に Deno 2 Release Candidate が公開されるようです。

## あとがき

今週はフロントエンドカンファレンス北海道で盛り上がった感のある一週間でした！
セッション自体も非常に充実した内容でしたので、公開されている資料を見てみると楽しめるかもしれません。
