---
title: "State of CSS 2023 の結果公開など : Cybozu Frontend Weekly (2023-08-29号)"
emoji: "🍉"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: true
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2023/08/29 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## `<search>` が Chrome118 で実装予定

https://groups.google.com/a/chromium.org/g/blink-dev/c/5CHw-SXYKGc/m/IPYMJeMTAQAJ

検索や絞り込み要素に関するコンテナ要素である `<search>` が Chrome 118 で実装予定とのことです。Firefox や Safari にはすでに実装済みのため、メジャーブラウザの多くで利用可能になります。

## Fresh 1.4 のリリース

https://deno.com/blog/fresh-1.4

Deno の Web フレームワークである Fresh のバージョン 1.4 がリリースされました。従来 Fresh では動的なビルドを行っていましたが、islands-architecture においてパフォーマンス上の問題が生じることがあり、その解消のために事前ビルド（ahead-of-time compilation）が可能となりました。他にも、レイアウト用ファイル（\_layout）のサポート・co-location 向けのルーティングに干渉しないディレクトリ分割のサポート、といった変更が含まれます。

## Bun 0.8.0 のリリース

https://bun.sh/blog/bun-v0.8.0

Bun の v0.8.0 がリリースされました。SvelteKit・Nuxt のサポートや、Node.js 互換性の向上などが含まれます。

## TypeScript 5.2.0

https://devblogs.microsoft.com/typescript/announcing-typescript-5-2/

TypeScript 5.2.0 がリリースされました。新しい構文である `using` の追加・Array を非破壊で操作するためのメソッドのサポート・WeakMap/WeakSet のキーとしての symbol が利用可能に、といった変更が含まれます。

## [@dan_abramov](https://twitter.com/dan_abramov) 氏が Bluesky のエンジニアとして JOIN

https://twitter.com/dan_abramov/status/1695566446007386214?s=20

## Astro が Vercel の公式ホスティングパートナーに

https://astro.build/blog/vercel-official-hosting-partner/

静的サイトジェネレータの Astro が、Vercel の公式ホスティングパートナーとなりました。併せて、Astro ミドルウェアが Vercel のエッジ上で動作するようになり、Vercel 上での画像最適化も利用可能になりました。

## State of CSS 2023 の結果公開

https://2023.stateofcss.com/en-US/

毎年恒例の State of CSS 2023 の Survey 結果が公開されました。新たに Open Props や Awards として Panda などの名前も登場しています。

## \[Fizz/Float\] Float for stylesheet resources #25243

https://github.com/facebook/react/pull/25243

"Float" と呼ばれる、React でのリソース読み込みに関する PullRequest です（PullRequest 自体は 2022 年にマージされています）。React 自体でのスタイルシートのプリロードや重複排除に加え、それらをコントロールするための新しい API が ReactDOM にも追加されており、React を利用している場合のフロントエンド周りでのリソースファイルの取り扱いにいずれ影響を与えるかもしれません。

## Next.js App Router のドキュメントに Form Validation・Error Handling の項目が追加

https://nextjs.org/docs/app/building-your-application/data-fetching/forms-and-mutations#form-validation

Next.js App Router の Server Actions に関係するドキュメントに、Form Validation と Error Handling の項目が追加されました。Form Validation では、zod を利用したスキーマバリデーションが紹介されています。

## VanillaJS での Reactive パターンの紹介

https://frontendmasters.com/blog/vanilla-javascript-reactivity/

"Reactivity" や "Signal" と呼ばれるパターンの VanillJS でのサンプル実装です。
Vue や Svelte などでのリアクティブの仕組みの理解に役立ちそうです。

## Microsoft Playwright Testing

https://techcommunity.microsoft.com/t5/apps-on-azure-blog/introducing-microsoft-playwright-testing-private-preview/ba-p/3905705

Microsoft が、Playwright をベースにした Azure 上でのテスト自動化サービスとして「Microsoft Playwright Testing」のプライベートプレビューを開始しました。すでに Playwright でテストコードを書いている場合はシームレスに統合でき、並列ブラウザでの分散実行による高速化やモバイルも含めた複数環境での実行をサポートしているそうです。

## WordPress が 100 年ドメイン保守プランを発表

https://wordpress.com/100-year/

WordPress が、一度の支払いで 100 年間の保守つきのドメインを契約するプランを発表したそうです。生涯のプレゼントにいかがでしょうか。
