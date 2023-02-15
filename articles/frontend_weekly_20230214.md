---
title: "React.jsのドキュメンタリー動画公開など : Cybozu Frontend Weekly (2023-02-14号)"
emoji: "🍫"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2023/02/14 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Vercel Meetup #0 with CEO

https://vercel.connpass.com/event/274772/

Vercel User Community によるオフライン meetup が開催されます。オンライン視聴も可能なようです。

Vercel CEO の Guillermo Rauch 氏が来日・参加されるとのことで、なかなかに貴重な機会になりそうです。

## Vue Fes Japan 2023 を一緒に作り上げるコアスタッフを募集します！

https://note.com/448jp/n/nc7d8b591d557

Vue.js の国内カンファレンスである Vue Fes Japan 2023 について、オフライン開催を予定している旨と、コアスタッフの募集についてのお知らせです。

イベント自体は 2023/10/28(土) の開催とのことです。

社内では「オフラインでのイベント・カンファレンスでのスタッフ経験は、人との繋がりを持つきっかけにもなって良いですよ！」という話もしていました。

## A Business Case for SvelteKit

https://elliscs.hashnode.dev/a-business-case-for-sveltekit

Meteor.js を使ったフルスタックなアプリケーションを SvelteKit に移行した事例紹介記事です。

アプリケーションの成長と共に Meteor.js でのビルド時間がかかるようになり、すでに他プロジェクトで利用していた SvelteKit への移行について何を検討したのかと、実際にどうやって導入したのかが説明されています。事前検証には大きく時間を費やしたようですが、実作業は 2.2 週間で終わり、結果としてビルド時間が 1/30、開発時の起動も 1/20 に短縮されたそうです。また、パフォーマンスでの向上も見られたとのことでした。

技術的な話だけではなく、SvelteKit への移行に踏み切るにあたり、ビジネス上どういったインパクトがあるかを KPI として設定したことなどにも触れられてます。

## 「I'm joining Vercel! I'm a longtime admirer of the people and their work. I can't wait to get started.」

https://twitter.com/acdlite/status/1623353741750546439

React.js のコアチームメンバーである Andrew Clark 氏 ([@acdlite](https://twitter.com/acdlite)) が、Vercel に入社したとのことです。

引き続き OSS を通じて React チームには所属し、60%は従来通りの作業をしつつ、残りの 40%で Next.js など Vercel に関する作業に取り組むとのことです。

社内からは、「Meta 社と Vercel 社の関係性はどういう感じなんだろう？」という疑問が出ていました。

## ヤフーショッピングのフロントエンドを支える共通配信技術

https://techblog.yahoo.co.jp/entry/2023020830411464/

ヤフーショッピングにおける、共通 UI 配信システム「Ptah」が誕生するまでの経緯と、具体的な実装まで踏み込んだ記事です。マイクロフロントエンドを採用し、かつ Custom Elements で React を Wrap して実現したとのことです。サイボウズ社内でも[マイクロフロントエンドを実践](https://blog.cybozu.io/entry/2022/12/21/110000)しており、レイアウトシフトなど、同じような課題感に納得する声が挙がっていました。

## React.js: The Documentary

https://www.youtube.com/watch?v=8pDqJVdNa44

React.js 公式による、React.js のドキュメンタリームービーが公開されました。誕生の経緯や辿ってきた歴史などを、メンテナなど内部のメンバーの心情を交えつつ追いかけることができます。

React 登場時には JSX に対して拒否感が強く持たれていたことに言及しており、当時を知るメンバーが懐かしがっていました。

## What's new in Lighthouse 10

https://developer.chrome.com/en/blog/lighthouse-10-0/

Web サイトのパフォーマンス計測ツールである Lighthouse の新しいバージョンである Lighthouse 10 に関しての記事です。Google Chrome では 112 以降で利用可能となる予定とのことです。

大きい変更点として、スコアの基準から TTI(Time To Interactive)が外され、代わりに CLS(Cumulative Layout Shift)のウェイトが大きくなります。

TTI はアクティブなネットワークの数などに大きく影響を受けてしまい、実際にページのコンテンツが読み込まれているかの指標としては最適ではなくなったようです。
アイランドアーキテクチャなどの登場により、コンテンツの配信・描画方法が変化しているのも影響しているかもしれません。

他にも、ブラウザの戻る・進む操作が適切にキャッシュから表示できるかや、ペースト操作の抑制が含まれていないかなども監査対象になりました。

## shadcn/ui

https://github.com/shadcn/ui

ヘッドレスな UI コンポーネントライブラリである [Radix](https://www.radix-ui.com/) と TailwindCSS を組み合わせたコンポーネント集です。

パッケージなどで配布されているわけではなく、コードサンプルとして公開されています。

頻繁に登場しそうなコンポーネントは一通り網羅されており、役立つケースは実際にありそうです。

ただ、利用する場合には Radix と TailwindCSS の更新にどの程度追従されていくのかは注意が必要かもしれない、との声もありました。

## stc : Roadmap to the alpha

https://stc.dudy.dev/docs/roadmap

Rust で開発中の TypeScript タイプチェッカーである stc のロードマップが公開されました。

「型の推論が正しく行われる」「誤検知 (false-positive）がない」「正しく型定義をロードできる」の３点が達成された時点で α 版の language server を公開する予定とのことです。

## Design Patterns in TypeScript

https://refactoring.guru/design-patterns/typescript

TypeScript でのデザインパターンのサンプル集です。

Factory、Builder、Singleton など、それぞれのパターンの概要がイラストつきで説明されており、かつ TypeScript でどのように実装すればよいかのコードサンプルが公開されています。

## core-js : So, what's next?

https://github.com/zloirock/core-js/blob/65e806202f68e751016d76a27b9a6bef89d2bf16/docs/2023-02-14-so-whats-next.md

大企業を含め非常に多くのサービスで利用されている core-js のメンテナである zloirock 氏の考えを記したページです。

OSS を少しでも使ったことがある方であれば、ぜひ一度ご自身で目を通すことを推奨します。

（なお、サイボウズは 2021 年から [core-js を支援](https://opencollective.com/cybozu)させて頂いています。）

## Bringing Javascript to WebAssembly for Shopify Functions

https://shopify.engineering/javascript-in-webassembly-for-shopify-functions

Shopify がバックエンドロジックのカスタマイズのために提供する [Shopify Functions](https://shopify.dev/docs/api/functions) について、以前は Rust を推奨言語としていましたが、新たに JavaScript がサポートされました。

動作するための実体は WebAssembly/WASI であり、JavaScript で書かれた Shopify Functions を動作させるために用いられている、[Javy](https://github.com/Shopify/javy)や[QuickJS](https://bellard.org/quickjs/)についても言及されています。

## Improved type safety in Storybook 7

https://storybook.js.org/blog/improved-type-safety-in-storybook-7/

Storybook における新しい記法である CSF 3.0 と TypeScript 4.9 で追加された `satisfies` オペレータを組合わせることで、Storybook 上での Props の指定漏れを検知したり、別の Story の `play()` 関数を呼び出した際に未定義エラーになることを回避できたりすることを紹介する記事です。

`as` との違いで混乱されることもある `satisfies` ですが、ユースケースとしてわかりやすい事例でした。

# あとがき

個人的には、core-js の記事に心を持っていかれた回でした。

企業規模に問わず OSS には大変お世話になっていますので、自分たちのできることを考えていきたいですね。

---

フロントエンドエキスパートチームでは毎月最終火曜の 17 時から Frontend Monthly というイベントを YouTube Live で開催しています。
その月のフロントエンド注目ニュースやゲストを呼んでの対談などフロントエンドに関する発信をしていますので是非どうぞ！
https://www.youtube.com/playlist?list=PLPTndynQK4dxLZFEZgOZjt_zKG-0JWoWy

またフロントエンドエキスパートチームでは、一緒にサイボウズのフロントエンドを最高にする仲間を募集しています。興味ある方はこちら ↓ から！
https://cybozu.github.io/frontend-expert/
