---
title: "JSR の Public Beta 公開など : Cybozu Frontend Weekly (2024-03-05号)"
emoji: "🎎"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2023 年 3 月 5 日 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Tempo

https://tempo.formkit.com/

新しく登場した Date 操作用のライブラリである Tempo が公開されました。Intl を利用しつつ、Intl 自体の使いづらさを内部で吸収してるのが特徴です。
フォーマット用 Token が Unicode 準拠ではなく day.js に沿った仕様なのが注意点になりそう、というのも話題に挙がりました。

## Pass ref as normal prop

https://github.com/facebook/react/pull/28348

React の Feature Flags に `enableRefAsProp` が追加され、有効化することで JSX runtime が ref を props として受け取れるようになる変更の PR がマージされました。正式に採用されるのは React 19 より先のバージョンになりそうですが、将来的に `forwardRef()` が不要になるかもしれません。

## rsc-html-stream

https://github.com/devongovett/rsc-html-stream

Server 側での RSC Payload を HTML Stream への注入と、Client 側で RSC Payload を読み込むための軽量なユーティリティパッケージです。バンドラーやフレームワークへの依存がなく、非常にシンプルな実装となっています。

## An HTML Switch Control

https://webkit.org/blog/15054/an-html-switch-control/

Safari 17.4 で 新しい HTML フォームコントロールとして Switch が導入されました。
input 要素の switch 属性については、WHATWG にて[提案](https://github.com/whatwg/html/pull/9546)されています。

## Lift-off: The MDN Curriculum launch

https://developer.mozilla.org/en-US/blog/mdn-curriculum-launch/

MDN からフロントエンド開発を学習するためのカリキュラムが公開されました。大きく Getting Started / Core / Extensions の３つに分類されており、レベル感に応じて学習に活用できます。

## Million Lint is in public beta

https://million.dev/blog/lint

React のパフォーマンス計測を行うためのツールである Million Lint が Public Beta として公開されました。コードをコンパイルすると自動的にプロファイリング用のコードが差し込まれ、それを VSCode 上に Lint 結果として表示できるようになります。`Lint++` という名前で、最適化のアドバイスを行う仕組みも有償公開されるようです。

## Introducing JSR - the JavaScript Registry

https://deno.com/blog/jsr_open_beta

TypeScript に最適化された JavaScript Registry である JSR が Public Beta になりました。TypeScript で作成したパッケージを、ビルドステップを踏まずに直接公開できるのが大きな特徴です。

## TanStack/time

https://github.com/TanStack/time

さまざまなフレームワーク・ライブラリを公開している TanStack ですが、新たに `time` リポジトリが作成されました。まだ中身はありませんが、時間やカレンダーを構築するためのヘッドレスコンポーネントが追加されていくようです。

## Tailwind CSS v4 について

https://twitter.com/adamwathan/status/1764383146559017048

Tailwind CSS v4 でのアルファ版についてのツイートです。Vite と組み合わせることで、postcss.config.js や tailwind.config.js といったファイルを必要とせず、非常に軽量な設定で利用可能となるようです。

## Why React Server Components Are Breaking Builds to Win Tomorrow

https://www.builder.io/blog/why-react-server-components

RSC が何を解決するのかを、CSR や SSR が抱えるデメリットと併せて解説した記事です。RSC を用いた場合の初期ロードと差分更新のシーケンスイメージも記載されており、RSC の理解を深める参考になりそうです。

## Apple が iOS の PWA サポート削除を撤回

https://developer.apple.com/support/dma-and-apps-in-the-eu/

EU 圏での iOS における WebKit 以外の許可に関連し、先日 iOS での PWA のサポートが削除される旨がアナウンスされていましたが、これが撤回され、引き続き PWA サポートは継続される方向となりました。リンク内の `Why don't users in the EU have access to Home Screen web apps?` から詳細な内容を確認できます。

## Intent to ship: Popover Attribute

https://groups.google.com/a/mozilla.org/g/dev-platform/c/hrCP5RBO7mA/m/7w-1OsmbAQAJ
https://bugzilla.mozilla.org/show_bug.cgi?id=1866993

Firefox 125 で [Popover 属性](https://developer.mozilla.org/ja/docs/Web/HTML/Global_attributes/popover)がサポートされる見込みのようです。サポートされた場合、主要ブラウザすべてで利用可能となります。

## Exhaustive branch checks with TypeScript

https://www.jackfranklin.co.uk/blog/typescript-exhaustive-branches/

TypeScript で Enum を Union 定義する際などに、default ケースに never を用いる関数を挟むことで、列挙漏れを防ぐテクニックの紹介記事です。

## Effect-TS/effect

https://github.com/Effect-TS/effect

TypeScript で関数型のような要素と、型システムを用いたエラーハンドリング・リソース解放などを行うライブラリです。

## polyfill.io now available on cdnjs: reduce your supply chain risk

https://blog.cloudflare.com/polyfill-io-now-available-on-cdnjs-reduce-your-supply-chain-risk

Cloudflare が polyfill.io 互換のエンドポイントを CDN で公開されました。polyfill.io が売却されたことにより様々な懸念が話題となっていますが、対応手段の一つとなり得そうです。

また、Fastly も同様のサービスを提供開始したようです。
https://community.fastly.com/t/new-options-for-polyfill-io-users/2540

## あとがき

個人的には JSR が気になるトピックでした。TypeScript のまま Publish できるのはとても便利そうで、試してみたいですね！

---

サイボウズでは毎月、最終火曜日の 17 時から Frontend Monthly というイベントを YouTube Live で開催しています。その月のフロントエンド注目ニュースや、ゲストを呼んでの対談など、フロントエンドに関する発信をしていますので是非どうぞ！

https://cybozu.github.io/frontend-monthly/

また、サイボウズでは一緒にサイボウズのフロントエンドをより良くする仲間を募集しています。興味ある方はこちら ↓ から！

**【フロントエンドエキスパート】**

サイボウズのフロントエンドを最高にするためのチームです。

https://cybozu.github.io/frontend-expert/

https://cybozu.co.jp/recruit/entry/career/front-end-expert.html

**【フロントエンドエンジニア（kintone アーキテクチャ刷新 PJ）】**

https://cybozu.co.jp/recruit/entry/career/front-end-engineer-kintone.html

**【フロントエンドエンジニア（サイボウズ Office フロントエンド刷新 PJ）】**

https://cybozu.co.jp/recruit/entry/career/front-end-engineer-office.html
