---
title: "-------など : Cybozu Frontend Weekly (2024-05-21号)"
emoji: "🐌"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2024 年 5 月 21 日 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## HTML attributes vs DOM properties

https://jakearchibald.com/2024/attributes-vs-properties/

HTML attributes と DOM properties の動作の違いについて解説している記事です。
Validation、値のチェック、デフォルト値、フレームワークに応じた挙動などについてまとめられています。

## Deep Dive into React Stream/Serialize

https://speakerdeck.com/mugi_uno/serialize

React の Stream/Serialize に関連する RSC Payload の読み方を解説している資料です。

## Percel における コアの Rust 置き換えに関して

https://x.com/devongovett/status/1789796247483937045

Percel コアの Rust 置き換えに関しての状況や効果についての X ポストです。
マルチコアを活用できることで、おおよそ 20 倍高速化するとのことです。

## pnpm の node_modules を探検して理解しよう - ドワンゴ教育サービス開発者ブログ

https://blog.nnn.dev/entry/2024/05/10/110000

npm における node_modules の問題点と、pnpm を利用した場合にそれらをどうやって解決しているかについての記事です。実際にディレクトリの内容を例として見ながら、ディレクトリ構造やシンボリックリンクの活用について解説されています。

## The Forensics Of React Server Components (RSCs)

https://www.smashingmagazine.com/2024/05/forensics-react-server-components/

CSR, SSR の特徴と、なぜ RSC が登場したのかに加えて、SSR Streaming を実際にデバッグしながら、どういった流れで Stream が処理されるかを解説した記事です。

## Node.js の進化に伴い不要となったかもしれないパッケージたち

https://zenn.dev/morinokami/articles/npm-uninstall

従来 Node.js での開発時に利用頻度の高かったライブラリ群について、現在では Node.js 標準で利用可能となっている機能をまとめた記事です。Dotenv, Chalk, glob 相当の機能などについて紹介されています。

## WebKit Features in Safari 17.5 | WebKit

https://webkit.org/blog/15383/webkit-features-in-safari-17-5/

Safari 17.5 での WebKit の新機能についての記事です。CSS における `text-wrap: balance` や `light-dark()`、`@import` におけるルールクエリや、WebGL、WebView などの変更を含みます。

## Figma’s journey to TypeScript | Figma Blog

https://www.figma.com/ja-jp/blog/figmas-journey-to-typescript-compiling-away-our-custom-programming-language/

Figma におけるモバイル向けのレンダリングアーキテクチャを、Skew というカスタムのプログラミング言語から、TypeScript へ移行したことに関する記事です。
社外エコシステムとの協調の難しさや、WASM によるパフォーマンスの改善によって Skew のアドバンテージが薄まったことなどが要因とのことです。

## React Compiler – React

https://react.dev/learn/react-compiler

React のドキュメントに React Compiler のページが追加されました。各種フレームワークでの使い方なども紹介されています。Next.js 側でも、React Compiler を利用する [PR](https://github.com/vercel/next.js/pull/65804) が作成されました。

## Merging Remix and React Router

https://remix.run/blog/merging-remix-and-react-router

Remix v3 としてリリース予定だった機能が React Router v7 としてリリースされることになり、その経緯や、今後の Remix の位置づけについての記事です。

## WCAG 3 Introduction

https://www.w3.org/WAI/standards-guidelines/wcag/wcag3-intro/

Web アクセシビリティに関するガイドラインである WCAG 3 のワーキングドラフトが公開されました。

## あとがき

〜〜〜〜〜

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
