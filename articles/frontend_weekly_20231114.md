---
title: "Next.js 14 PPR の紹介など : Cybozu Frontend Weekly (2023-11-14号)"
emoji: "🧣"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2023/11/14 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Building towards a new default rendering model for web applications

https://vercel.com/blog/partial-prerendering-with-next-js-creating-a-new-default-rendering-model

先日の [Next.conf](https://www.youtube.com/watch?v=gfU1iZnjRZM) でも発表された、Next.js 14 以降で Experimental 機能として利用可能な PPR (Partial Prerendering) に関する Vercel による説明記事です。
`<Suspence>` を境界に静的シェルのプリレンダリングを行うことで、静的部分を Edge 環境から高速に配信可能となります。時間・タグ・パスをベースにした revalidate にも対応しているようです。

## Partial Prerendering in Next.js 14 (Preview)

https://www.youtube.com/watch?v=wv7w_Zx-FMU

上記の記事に加えて PPR について、[@leerob](https://twitter.com/leeerob) 氏による解説動画です。
実例をもとに、PPR を用いることでレンダリングの効率化を行えるケースを紹介しています。

## Rust で Prettier の 95% 以上のテストに合格すると $10k

https://twitter.com/Vjeux/status/1722733472522142022

Rust を使って Prettier の 95% 以上のテストケースをパスした場合、$10k の賞金を個人で提供する旨のツイートがされていました。
現在は Vercel CEO の Guillermo Rauch 氏も賛同し $20k になっているようです。チャンスです！

## CSS nesting relaxed syntax update

https://developer.chrome.com/blog/css-nesting-relaxed-syntax-update/

Chrome 120 以降、CSS nesting の構文が更新され、ネストした場合でも `&` なしで要素名が使えるようになるようです。

## Prettier に `--experimental-ternaries` オプションが追加

Prettier v3.1 以降で `--experimental-ternaries` オプションが追加され、三項演算子のフォーマットが可能になりました。
併せてフィードバック用の Google Forms も公開されているようです。

フォーマット例

```
const animalName =
  pet.canBark() ?
    pet.isScary() ?
      'wolf'
    : 'dog'
  : pet.canMeow() ?
    'cat'
  : 'probably a bunny';
```

## Astro 3.5

https://astro.build/blog/astro-350/

Astro の 3.5 がリリースされました。

> This may be one of the biggest minor releases in Astro history!

と説明があり、マイナーリリースの中ではかなり大きいリリースとのことです。

i18n に対応したルーティングや、以前から Integration として提供されていた Prefetch のコアへの組込、`<form>` での View Transition のサポートなどが含まれます。

---

今週はコンパクトな会でした。

来週は [JSConf JP](https://jsconf.jp/2023/) や、[フロントエンドカンファレンス沖縄 2023](https://frontend-conf.okinawa.jp/)も開催され、楽しみですね！
