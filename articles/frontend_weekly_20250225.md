---
title: "TBD : Cybozu Frontend Weekly (2025-02-25号)"
emoji: "🌸"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2025 年 2 月 25 日の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Deno が提出していた JS 商標取消申立てに対して、Oracle が正式に却下申立てを提出

Oracle が Deno の JavaScript 商標取消申立てに対して却下申立てを提出しました。

- 2019 年に Oracle が提出した商標更新の証拠（Node.js HP のスクリーンショット）が無関係かつ詐欺的という主張に対し、Oracle は現在は関係ないとの立場を示しています
- 代わりに Oracle JET のページを商標維持の証拠として提出し、これが十分な証拠であると主張しています
- これに対して Deno 側も反論を行っています

https://deno.com/blog/deno-v-oracle2

## Opera introduces Opera Air

Opera 社が新しいブラウザ Opera Air を発表しました。ウェルビーイングに焦点を当てた新しいブラウザとなっています。

https://blogs.opera.com/news/2025/02/opera-air-browser/

## Intent to Prototype: Smooth corners

スムースな角丸を実現する `corner-shape` プロパティのプロトタイプ実装が提案されました。

- superellipse による滑らかな角丸の実現が可能に
- `border-radius` のプログレッシブエンハンスメントとして実装される予定

https://groups.google.com/a/chromium.org/g/blink-dev/c/UKLLVghiHYQ/m/tv0ANlToEQAJ

## New to the web platform in January

2025 年 1 月に、安定版およびベータ版のブラウザに搭載された機能をピックアップして紹介している記事です。次のような内容が取り上げられています。

- `Promise.try` が Baseline NA
- `writing-mode: sideways-rl | sideways-lr` の追加
- Popover API の Safari バグ修正
- `bytes()` メソッド
- device-posture media queries
- WebAuthn Signal API

https://web.dev/blog/web-platform-01-2025

## Next.js の脆弱性 CVE-2024-46982 の改修が不十分との指摘

Next.js のキャッシュポイズニングに関する脆弱性（CVE-2024-46982）の改修について、不十分との指摘がありました。

- Confidentiality が None と評価されている点について、誤りであるとの指摘がされています

https://x.com/bulkneets/status/1888768328010956882

## State of React 2024

State of React 2024 が公開されました。

https://2024.stateofreact.com/ja-JP/

## LINE ヤフーのフロントエンド技術を明らかにする State of LY Frontend 2024 実施レポート

LINE ヤフー社のフロントエンド技術の現状についての調査結果レポートです。
次のような内容が挙げられています。

- 元の組織により採用技術の割合が異なっていること
- Vitest が急速に普及している
- 自己評価では JavaScript と比較して CSS に対して不安を感じる人が多いこと
- 社内向けの生成 AI ツールが提供されており、広く利用されていること

https://techblog.lycorp.co.jp/ja/20250210a

## あとがき

Opera Air が予想してなかった方向性での進化という感じでしたね。今後が楽しみです。
