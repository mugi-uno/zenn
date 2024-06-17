---
title: "The State of JS 2022 公開など : Cybozu Frontend Weekly (2023-01-17号)"
emoji: "⛄️"
type: "tech"
topics: ["CybozuFrontendWeekly", "frontend"]
published: true
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社 フロントエンドエキスパートチームの [@mugi_uno](https://twitter.com/mugi_uno) です。

# はじめに

フロントエンドエキスパートチームでは毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を社内で開催しています。

今回は、2023/01/17 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## The State of JS 2022

https://2022.stateofjs.com/en-US/

毎年恒例 The State of JS の 2022 版が公開されました。

React や Next.js がより安定感強く利用されているような結果が出ていました。
昨年と比較し、ビルドツールでは webpack が少し後退しているのに対し、Vite は興味・利用実績ともに上昇しています。

知らない API・ライブラリを知る機会にもなるので、一度目を通しておくと良いかもしれません。

## The Turbopack vision – Vercel

https://vercel.com/blog/the-turbopack-vision

Next.js 13 でアルファ版が同梱された Turbopack についての記事です。

webpack が抱えていた課題に加え、Turbopack が何を解決するのか、今後の展望などについて述べられています。
ロードマップは https://turbo.build/pack/docs/roadmap にあるそうです。

## CS Tool のフロントエンドのリプレイスプロジェクトについて | メルカリエンジニアリング

https://engineering.mercari.com/blog/entry/20230112-frontend-replacement/

メルカリ社の CS Tool (Customer Service Tool) のフロントエンドのリプレイスに関してです。

PHP テンプレート Twig 、フロントエンドフレームワーク tupai.js 、一部書き換えた React が混在する環境で抱えていた課題と、Strangler パターンによる置き換えについて書かれています。

## (Twitter) I left Rome Tools. It has been an exciting time, ....

https://twitter.com/MichaReiser/status/1613474278808162304

[Rome](https://rome.tools/) の活動の大半を締めていた [Micha Reiser](https://twitter.com/MichaReiser) 氏が Rome を離れることになりました。
影響を受けてか、定期での 1 月版リリースも来ていないなど、目に見えて Rome の活動が止まっているとのことです。

公式からも特にアナウンスは無い状態で、今後 Rome がどうなるのか心配な状況です。

## Chakra UI - FigPilot

https://figma.chakra-ui.com/

Figma でデザインした UI を、ChakraUI を利用したコードに変換してくれる Figma Plugin についてです。

## Storybook Ecosystem CI

https://storybook.js.org/blog/storybook-ecosystem-ci/

Storybook 自体が運用している CI に関しての記事です。

React・Vue・Angular...といった多くの Renderer や言語、ビルダー、パッケージマネージャなど、さまざまな互換性を持つ必要のある Storybook において変更を加えていく困難さに対し、それに対処するために Vite の Storybook 本体に Ecosystem CI という CI を導入しているとのことです。CI の状況は [Status Page](https://storybook.js.org/status) で誰でも確認できるようです。

## All of Learn Accessibility! is available

https://web.dev/learn-accessibility-available/

web.dev が a11y 学習用コンテンツとして公開していた Learn Accessibility について、以前は一部コンテンツが未公開でしたが、全コンテンツが公開されました。

新たにフォームやテストなどにも言及されており、輪読会などのコンテンツとしても非常に優秀そうです。

## GitHub Next | Code Brushes

https://githubnext.com/projects/code-brushes

GitHub Copilot を利用し、すでに記述されているコードを修正してくれる VS Code Extension についてです。

「Photoshop のブラシで絵を描くように」ということで Brushes だそうです。

## ci: add publint to check packaging on CI by koba04 · Pull Request #2363 · vercel/swr

https://github.com/vercel/swr/pull/2363

[@koba04](https://twitter.com/koba04)が SWR に [publint](https://github.com/bluwy/publint) を導入した際の PR です。

publint では npm パッケージを publish する前の段階で package.json の状態と比較して問題が無いか検証でき、社内で利用するようなプライベートパッケージでも応用できそうです。

# あとがき

公開された The State of JS は眺めているだけでも楽しいのでおすすめです！

一方で、Rome の状況が心配ですね。今後の動向に注目といった印象です。
