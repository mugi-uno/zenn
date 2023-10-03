---
title: "Announcing Deno Queues など : Cybozu Frontend Weekly (2023-10-03号)"
emoji: "🎑"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2023/10/03 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## vite-plugin-ssr が Vike にリネームされた

https://vike.dev/

vite-plugin-ssr が Vike にリネームされました。機能的な変化は特になく、名前が変更されただけのようです。公式ドキュメント上にも `Functionally, nothing changes: it's just a new name for the same tool.` との[記述](https://vite-plugin-ssr.com/vike)があります。

## Announcing Deno Queues

https://deno.com/blog/queues

Deno で利用できる Key-Value Store である Deno KV に関して、キューイングを実現する Deno Queues が発表されました。ユースケースとして、E メール送信や webhook、Slack での Bot などが挙げられています。Deno Queues 自体への課金はなく、それによって実行された Deno KV や Deno Deploy の操作が課金対象となるようです。

## D1: open beta is here

https://blog.cloudflare.com/d1-open-beta-is-here/

Cloudflare が提供するエッジで動作する DB サービスの D1 がオープンベータとなり、誰でも利用可能となりました。
記事内では、D1 が提供するロールバック機能である Time Travel や、価格体系についても紹介されています。

## Hyperdrive: making databases feel like they’re global

https://blog.cloudflare.com/hyperdrive-making-regional-databases-feel-distributed/

DB へのコネクションを提供するサービスである Hyperdrive が発表されました。Cloudflare Workers から接続可能で、Hyperdrive 側でコネクションプールを管理することで、コネクション時のオーバーヘッドの削減が期待できます。[aiji42](https://zenn.dev/aiji42)さんが検証結果を Zenn スクラップにまとめられており、大変参考になります。

- [Hyperdrive 試してみたよ](https://zenn.dev/aiji42/scraps/62411e4b0daaed)

## `<selectlist>` のプロトタイプ実装

https://twitter.com/Una/status/1706777335762997283

OpenUI で仕様策定中の `<selectlist>` 要素に関するサンプル実装です。スタイリングを伴うドロップダウンを提供する際、従来では独自実装するしかありませんでしたが、`<selectlist>` を用いるとブラウザ標準機能として使えるようになります。

## `<select>` に対して showPicker を実行できるように

https://groups.google.com/a/chromium.org/g/blink-dev/c/qew_ILTXWSY

従来 `<select>` 要素で選択するためのドロップダウンはユーザーイベントによるクリックが必要でしたが、`showPicker` API を呼び出すことで JavaScript から展開可能になります。Chrome では 119 で [shipping 予定](https://groups.google.com/a/chromium.org/g/blink-dev/c/qew_ILTXWSY)のようです。

## A Socket API that works across JavaScript runtimes — announcing a WinterCG spec and Node.js implementation of connect()

https://blog.cloudflare.com/socket-api-works-javascript-runtimes-wintercg-polyfill-connect/

エッジ環境からの DB 接続などで利用可能な TCP Socket 接続を提供する `connect()` API に関して、Winter CG での仕様策定が進んでいる旨の紹介記事です。（`connect()` 自体に関しては以前に[別の記事](https://blog.cloudflare.com/workers-tcp-socket-api-connect-databases/)で紹介されています。）

## The Saga of the Closure Compiler, and Why TypeScript Won

https://effectivetypescript.com/2023/09/27/closure-compiler/

Closure Compiler と TypeScript を比較し、なぜ TypeScript が普及したのか？などについて言及している記事です。
Closure Compiler がどういったものなのか、といった説明から始まり、TypeScript が開発者向けツールに注力したことや、コミュニティをうまく巻き込んで開発を進めた点など、さまざまな観点で解説しています。

## 【Next.js】管理者用ページを Route Groups で実現する

https://zenn.dev/chot/articles/next-layout-for-admin-page

Next.js App Router で利用可能な Route Groups 機能について、具体的にどういったケースで役立つか？という観点を踏まえて解説している記事です。記事内では、管理者 or 一般ユーザーでの切り分けを行うケースが紹介されています。

## Nue JS

https://github.com/nuejs/nuejs

軽量な JavaScript ライブラリ。公式では Vue.js、React、Svelte のような位置付けであると説明されています。
HTML ベースのテンプレートを利用するのが特徴なようです。

## Expert CSS: The CPU Hack

https://dev.to/janeori/expert-css-the-cpu-hack-4ddj

CSS アニメーションと CSS Variables を組み合わせた場合に、どういったタイミングで再計算が行われるのかと、それをハックして意図したタイミングで再計算させる手法を紹介しています。
アニメーションの指定方法によって、特定の順序で CSS ホバーを行った場合にみ再計算されることを利用し、最終的には HTML と CSS のみでブロック崩しゲームが実現されています。
