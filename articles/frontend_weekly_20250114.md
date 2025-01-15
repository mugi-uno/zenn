---
title: " WinterCG の WinterTC への移行など : Cybozu Frontend Weekly (2025-1-14号)"
emoji: "🏂"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: true
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2025 年 1 月 14 日 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Blog: DoubleClickjacking: A New Era of UI Redressing

https://www.paulosyibelo.com/2024/12/doubleclickjacking-what.html

悪意のあるページによるダブルクリックを利用した攻撃手法に関しての解説です。

## 5 tips to effectively optimize INP in React

https://calendar.perfplanet.com/2024/5-tips-to-effectively-optimize-inp-in-react/

React で INP を改善するための tips です。DOM サイズの削減が推奨されており、計測方法についても紹介されています。
また、Suspense の利用や useEffect 利用時の注意事項などについても触れられています。

## CSS Wish List 2025

https://meyerweb.com/eric/thoughts/2025/01/08/css-wish-list-2025/

Eric Mayer が 2025 年の CSS に期待する機能について解説したページです。2023 年にも同様の記事が公開されており、含まれていた多くの機能が実際に利用可能となりました。
2025 年版では、句読点のぶら下がりをコントロールする Hanging punctuation や、Masonry layout などが挙げられています。

## Firefox 134.0 および 135.0a1 Nightly

https://www.mozilla.org/en-US/firefox/134.0/releasenotes/
https://www.mozilla.org/en-US/firefox/135.0a1/releasenotes/

Firefox 134.0 がリリースされました。`RegExp.escape()` や `Promise.try()` のサポートなどが含まれます。
また、135.0a1 Nightly では Temporal も実装されており、`javascript.options.experimental.temporal` フラグを有効化することで試せます。

## New Front-End Features For Designers In 2025 — Smashing Magazine

https://www.smashingmagazine.com/2024/12/new-front-end-features-for-designers-in-2025

ここ数年で新たに利用可能となった HTML および CSS の機能のまとめです。
anchor-positioning, Container Queries, `<dialog>` といった機能に触れられています。

## MDN 2024 content projects | MDN Blog

https://developer.mozilla.org/en-US/blog/mdn-2024-content-projects/

MDN が、HTTP, MathML, Web Manifest のドキュメントに絞り、2024 年 7 月から 12 月にかけてコンテンツを刷新したとのことです。

## What is an Input Method Editor?

https://garai.ca/what_is_ime/

文字入力に使われる IME の基本的な概念や API に関しての解説記事です。

## Announcing Supporters of Chromium-based Browsers

https://blog.chromium.org/2025/01/announcing-supporters-of-chromium-based.html
https://www.linuxfoundation.org/supporters-of-chromium-based-browsers

Google と The Linux Foundation がパートナーシップを締結しました。
また、Chromium エコシステムの発展を目指す「The Supporters of Chromium-Based Browsers」も同時にアナウンスされました。

## Goodbye WinterCG, welcome WinterTC

https://deno.com/blog/wintertc

JS ランタイムの相互運用性を推進するコミュニティグループであった WinterCG が Ecma International に移行し、新たに WinterTC という名称となりました。
これにより、標準規格の発行も可能となります。

## standard-schema/standard-schema

https://github.com/standard-schema/standard-schema

zod や valibot などに代表されるスキーマライブラリなどの標準インタフェースを定義した仕様です。
tRPC や TanStack Router など、すでに一部のライブラリはこの仕様に則っているようです。

## 2024 JavaScript Rising Stars

https://risingstars.js.org/2024/en

2024 年に GitHub での Star 数の増加が大きかった JavaScript 関連リポジトリです。
shadcn/ui が 2 年連続で 1 位となりました。また、Back-end/Full-Stack 部門で Hono が 2 位となっています。

## Updates to the customizable select API

https://una.im/select-updates/

2024/9 に出された[RFC](https://developer.chrome.com/blog/rfc-customizable-select?hl=ja)からの差分をまとめた、「Customizable Select Element」のアップデート記事です。
主に要素・擬似要素名の変更や、ベーススタイルの変更などがピックアップされています。
Customizable Select Element は、複数の関連する技術から構成されており、それらは現在も仕様策定中となっています。

## あとがき

2024 JavaScript Rising Stars で Hono が急上昇していましたね！自分も応援してます！
