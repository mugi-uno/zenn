---
title: "State of CSS 2024 公開など : Cybozu Frontend Weekly (2024-11-19号)"
emoji: "🧣"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2024 年 11 月 19 日 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Deno Deploy で Next.js がデプロイできるように

https://x.com/deno_land/status/1849469080257900704?s=12&t=Rreh9a4Ymw8GBLg-X9K6iw

Node.js との互換性が日々向上している Deno ですが、Deno Deploy に対して Next.js がデプロイできるようになりました。[テンプレートリポジトリ](https://github.com/nextjs/deploy-deno)も用意されているようです。

## We're forking Flutter. This is why.

https://getflocked.dev/blog/posts/we-are-forking-flutter-this-is-why/

Flock と呼ばれる Flutter の Fork プロジェクトの立ち上げについてのブログです。Google 側の Flutter への開発がやや停滞気味になっており、レビューなども遅れていることなどが理由として挙げられています。Flock では Flutter チームが実装しないバグ修正や機能追加を行い、コントリビュートしやすい環境整備も行っていくとのことです。

## State of CSS 2024

https://2024.stateofcss.com/en-US

CSS の利用状況などに関してのサーベイである State of CSS の 2024 年度版が公開されました。新しい機能のなかでは、CSS Filter Effects や `:has()` などが多く利用されるようになったようです。機能単位での Baseline 状況も確認でき、知らない機能を知るきっかけにもなりそうです。

## Save the Date for JSConf North America 2025: Where JavaScript Meets Adventure on the Chesapeake Bay

https://openjsf.org/blog/why-attend-jsconf-na-2025

2025 年 10 月 14 日から 16 日にかけて、JSConf North America が開催されるとのことです。

## Should JavaScript be split into two languages? New Google-driven proposal divides opinion

https://devclass.com/2024/10/22/should-javascript-be-split-into-two-languages-new-google-driven-proposal-divides-opinion/?ref=hackernoon.com

JavaScript を JavaScript JS0・JSSugar と呼ばれる Engine と Tooling の 2 つに分割するという TC39 への提案に関する解説記事です。

## We’re building the future of JavaScript packages

https://www.vlt.sh/

新しい JavaScript 向けのパッケージマネージャである vlt が公開されました。

## Amazon Route 53 announces HTTPS, SSHFP, SVCB, and TLSA DNS resource record support - AWS

https://aws.amazon.com/jp/about-aws/whats-new/2024/10/amazon-route-53-https-sshfp-svcb-tlsa-dns-support/|

Amazon Route 53 が HTTPS、SSHFP、SVCB、TLSA DNS リソースレコードのサポートを発表しました。

## Q3 2024 Summary from Chrome Security

https://groups.google.com/a/chromium.org/g/chromium-dev/c/KOmfh5BW6Mw?pli=1

Chrome での 2024 Q3 におけるセキュリティに関する情報のサマリです。[Device Bound Session Credentials](https://wicg.github.io/dbsc/)や 2 つの Root CA の削除などを紹介されています。

## Cascade Layers/レイヤーによる優先順位の制御

https://gihyo.jp/article/2024/11/ride-modern-frontend-03

Cascade Layers に関しての機能自体やユースケースに関しての解説記事です。書きました！

## Docusaurus 3.6

https://docusaurus.io/blog/releases/3.6

ドキュメントサイトに特化した静的サイトジェネレータの Docusaurus の 3.6 がリリースされました。Rust ベースのツールへの移行によるビルド時間やメモリ消費の改善などが含まれます。

## Intent to Prototype: aria-actions

https://groups.google.com/a/chromium.org/g/blink-dev/c/cYs7hgcwgcU

新たな aria 属性である `aria-actions` に関する提案です。タブの閉じるボタンなどのような、セカンダリであるアクションボタンの発見を容易にすることが目的のようです。

## The Different (and Modern) Ways To Toggle Content | CSS-Tricks

https://css-tricks.com/the-different-and-modern-ways-to-toggle-content/

トグルしてコンテンツを表示する要素や API（Details/Summary, Dialog API, Popover）などについて、使用方法やユースケースについて具体例を交えて解説されている記事です。

## Storybook 8.4

https://storybook.js.org/blog/storybook-8-4/

Storybook 8.4 がリリースされました。テストを一括実行するボタンの追加や、依存パッケージの大幅な削減、Svelte 5 のサポートなどが含まれます。

## TypeScript 5.8 で条件付き戻り値型に対するナローイングができるようになりそう（特定の制約を満たす場合）

https://bufferings.hatenablog.com/entry/2024/11/11/232139

TypeScript 5.8 で追加される可能性のある、関数の戻り値での Conditional Type の利用に関しての紹介ブログです。同機能については、先日開催された TS Kaigi Kansai での uhyo さんの登壇資料「[TypeScript の次なる大進化なるか！？　条件型を返り値とする関数の型推論](https://speakerdeck.com/uhyo/typescriptnoci-naruda-jin-hua-naruka-tiao-jian-xing-wofan-rizhi-tosuruguan-shu-noxing-tui-lun)」も参考になりそうです。

## Framer Motion is now independent, introducing Motion

https://motion.dev/blog/framer-motion-is-now-independent-introducing-motion

React のアニメーションライブラリである Framer Motion が Framer から独立し、Motion と呼ばれるプロジェクトで開発されることになりました。React 以外での利用に向けた API セットの提供や、サポート・ドキュメントの改善などに取り組んでいくとのことです。

## あとがき

State of XXX のサーベイや公開が始まると年末だなという感じがしてきますね。
