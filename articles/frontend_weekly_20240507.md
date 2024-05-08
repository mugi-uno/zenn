---
title: "React 19 Beta の公開など : Cybozu Frontend Weekly (2024-05-07号)"
emoji: "🐌"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: true
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2024 年 5 月 7 日 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## React の `fetch()` 拡張が削除

https://github.com/facebook/react/pull/28896

React に標準で含まれていた `fetch()` に対するパッチが削除されました。
一方で、Next.js では同様の対応を別のパッチとして取りこむことで従来通りの挙動を維持するようです。

## Google、サードパーティ cookie 廃止を 3 度目の延期

https://www.itmedia.co.jp/news/articles/2404/24/news102.html

年内で完了予定だった Chrome での Third Party Cookie 廃止について、延期が発表されました。
業界・規制当局・開発者といったさまざまな関係者を踏まえた調整が続いているのが理由であり、現状では 2025 年初頭から廃止を進める想定とのことです。

## Deep Dive into Rspack & Webpack Tree Shaking

https://github.com/orgs/web-infra-dev/discussions/17

Webpack と Webpack 互換の Rspack の Tree Shaking の概念について解説されている記事です。

## TSKaigi のタイムテーブルが公開

https://tskaigi.org/talks

2024/5/11 に開催される TSKaigi のタイムテーブルが公開されました。

## WebAssembly は次世代のコンテナ技術になれるか？

https://zenn.dev/mizchi/articles/wasm-component-model-futures

昨今 WebAssembly に期待されるコンテナ代替としての活用について、現時点で WebAssembly 可能なことと問題点を踏まえて考察した記事です。

## Accessibility Visualizer Browser Extension

https://github.com/ymrl/a11y-visualizer

アクセシビリティ情報を可視化するブラウザ拡張機能です。[日本語のユーザーズガイド](https://github.com/ymrl/a11y-visualizer/blob/main/docs/ja/UsersGuide.md)も用意されています。代替テキスト・見出しレベル・フォームのラベルといった、アクセシビリティ上では重要ですが視覚的に確認できないものを可視化することができます。

## builderscon 2024 をやります！

https://blog.builderscon.io/entry/2024/04/25/130353

2024 年 8 月 10 日に、builderscon 2024 開催が発表されました。2019 年に開催されて以来の 5 年ぶりとなり、現在[トークを募集中](https://twitter.com/builderscon/status/1787727675337101376)とのことです。

## eslint-plugin-import をフォークした eslint-plugin-i が eslint-plugin-import-x にリネームし、独立パッケージとして開発が開始される

https://github.com/un-ts/eslint-plugin-import-x/issues/24

eslint-plugin-import をフォークした eslint-plugin-i が以前から存在していましたが、課題解決などに限界があることから、独立した eslint-plugin-import-x というパッケージとして開発が開始されました。

## Tracking: ESLint v9 support

https://github.com/eslint/eslint/issues/18391

メジャーな ESLint プラグインについて、ESLint v9 への対応状況をトラッキングしているページです。
対応済みかどうかには Flat Config のサポートが前提とされており、すでにかなり対応が進んでいることがわかります。

## React 19 Beta の公開

https://react.dev/blog/2024/04/25/react-19

React 19 Beta が公開され、Next.js App Router ユーザーでは以前から利用していた `useActionState` `useFormStatus` `useOptimistic` `use` などの API や、`forwardRef` を用いない `ref` の受け渡しといった変更が含まれます。

## Announcing TypeScript 5.5 Beta

https://devblogs.microsoft.com/typescript/announcing-typescript-5-5-beta/

TypeScript 5.5 Beta が公開されました。配列の `filter()` 関数の推論がより正確となる Inferred Type Predicates、正規表現の構文チェックを行える Regular Expression Syntax Checking、ファイル内で型推論を閉じる代わりに高速かつシンプルに型定義ファイルを出力する Isolated Declarations などが含まれます。

## Web フロントエンド版 DX Criteria が公開

https://prtimes.jp/main/html/rd/p/000000024.000081310.html

Web フロントエンドの技術領域観点について、全 100 項目（大テーマ 5 つ x 小テーマ 5 つ x 観点 4 つ）からなるガイドラインが公開されました。
改善すべき課題の発見・チーム内での認識の統一など、様々なシチュエーションで活用できそうです。

## あとがき

TSKaigi や builderscon など、オフラインのカンファレンスが活発になってきた印象ですね！

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
