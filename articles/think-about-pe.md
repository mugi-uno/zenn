---
title: "Progressive Enhancement について考える"
emoji: "🚶‍♀️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["フロントエンド", "React"]
published: false
publication_name: "cybozu_frontend"
---

最近になって、**Progressive Enhancement（プログレッシブ・エンハンスメント）** という言葉をよく耳にします。

モダンなフロントエンド開発で利用されるフレームワークである Remix や Next.js を利用している際にも頻繁に登場します。

https://remix.run/docs/en/main/discussion/progressive-enhancement

https://nextjs.org/docs/app/api-reference/functions/server-actions#progressive-enhancement

Progressive Enhancement という概念は最近になって登場したものではなく、Wikipedia によると最初に登場したのは 2003 年のことだそうです。

https://ja.wikipedia.org/wiki/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%AC%E3%83%83%E3%82%B7%E3%83%96%E3%82%A8%E3%83%B3%E3%83%8F%E3%83%B3%E3%82%B9%E3%83%A1%E3%83%B3%E3%83%88

## Progressive Enhancement とは

Progressive Enhancement は特定の仕組みや機能のことではありません。設計哲学・戦略・概念、といった言葉で説明されることが多いようです。

Google などで調べると簡単に情報が出てくるので詳細な説明は割愛しますが、Wikipedia では次のように説明されています。

> Progressive enhancement is a strategy in web design that puts emphasis on web content first, allowing everyone to access the basic content and functionality of a web page, whilst users with additional browser features or faster Internet access receive the enhanced version instead.
> ChatGPT による翻訳: プログレッシブ・エンハンスメントは、ウェブデザインの戦略で、まずウェブコンテンツを重視し、全ての人がウェブページの基本的なコンテンツと機能にアクセスできるようにしつつ、追加のブラウザ機能やより高速なインターネットアクセスを持つユーザーには、代わりに強化されたバージョンを提供するものです。

古い環境・ネットワーク回線が低速といった場合でも Web ページのコアとなるコンテンツは利用できる状態をベースとして設計し、逆に新しい環境・ネットワーク回線が高速といった場合には、段階的により充実した機能を漸進的に提供する、といったイメージです。

# モダンなフロントエンド開発における Progressive Enhancement

昨今のフロントエンド開発においては、意識しないと瞬く間に Progressive Enhancement とはかけ離れた状態になります。

たとえば、Polyfill を導入せずに ECMAScript の最新機能をモリモリ利用した場合、レガシーブラウザでは一切動かない形になります。

他にも、すべてがクライアント側で完結する SPA として実装した場合を例に考えてみても、何らかの理由で JavaScript が利用できない、あるいはネットワーク回線の都合で .js ファイルの取得に凄まじく時間がかかるといった場合、前者の場合はコンテンツを一切利用できず、後者の場合でもコンテンツが利用可能になるまで著しく遅延することになります。

# ブラウザで JavaScript が無効な状態でもサポートすれば OK？

「そうか！じゃあ Progressive Enhancement に徹底的にこだわろう！」と思いたくもなりますが、現実的にどこまで考慮すべきなのか？というのは難しい問題です。

主観ですが、Progressive Enhancement について調べたり話したりしていると、暗黙的に "Progressive Enhancement = JavaScript が無効化されている環境でも動く" という前提になることが多い印象です。

しかし、現実的に JavaScript が完全に無効なケースはどこまで考えるべきでしょうか。たとえば、Web アクセシビリティに関するサーベイなどを行っている WebAIM の 2019 年時点の調査では、スクリーンリーダー利用ユーザーのうち、JavaScript を無効化しているユーザーの割合は全体のうちの 0.7% だったそうです。
https://webaim.org/projects/screenreadersurvey8/#javascript

これを多いと見るか少ないと見るかは様々な前提条件によって変わりそうではあります。

ただ、Progressive Enhancement が提唱された当時は JavaScript が現在ほど活用されておらず、ブラウザの設定によって JavaScript が無効化されているシチュエーションを想定する比重は高かったのではないかと予想されます。しかし、現代においてはかなり状況が変わっており、当時の思想をそのまま適用すると少し歪みが生まれるようにも感じます。

## 「JavaScript が無効」とは

現代ではほぼすべてのユーザーが JavaScript が利用可能な環境でブラウジングをしていると考えても差し支えないかもしれませんが、一方で、JavaScript が動作していないシチュエーション自体は日常的にありえます。

Remix の Progressive Enhancement のページには次の一文があります。

> While your users probably don't browse the web with JavaScript disabled, everybody has JavaScript disabled until it has finished loading.
> ChatGPT による翻訳: ユーザーの多くは JavaScript を無効にしてウェブを閲覧することはないでしょうが、JavaScript が完全に読み込まれるまでは、全員が JavaScript を無効にしています。

常時 JavaScript が無効かどうか、という観点ではなく、読み込まれて初期化が完了するまでは JavaScript に期待する動作は実現できません。この観点で考えると、特定のユーザーに限ったものではなく、すべてのユーザーにとって関係のある話になってきます。

## JavaScript だけではない

Remix や Next.js の観点だと JavaScript を中心にした話になりがちですが、Progressive Enhancement は JavaScript だけケアするものではありません。

たとえば、古い環境での動作を想定する場合には CSS についても考慮が必要です。モダンな CSS が利用できない環境では大幅に表示が崩れてしまい、まともに操作ができない...といった状態になると、Progressive Enhancement とは乖離します。

# アクセシビリティ観点での Progressive Enhancement

アクセシビリティ観点で Progressive Enhancement を考えてみます。

まず、Progressive Enhancement を実現しようと試みた場合、伴って一定のアクセシビリティが担保されやすいと思われます。フォームでの入力・送信を例に考えた場合、仮に "ボタンの `click` イベントで form の `submit()` を実行" といった実装をしていると、JavaScript が有効化される前の段階では動作しません。そのため、できるかぎり純粋な `<form>` タグや `<input type="submit">` などを用いる必要が出てきます。バリデーションなども同様で、最低限のものであれば HTML で利用可能なフォームバリデーション（`require` 属性、`type` 属性、など）を用いて実現しなければいけません。結果的に基本的なマークアップでの実装が増えることで、ブラウザが提供する標準的なアクセシビリティの動作に沿うことができます。

また、通信回線が遅い環境でのユーザーへの配慮がアクセシビリティ文脈で必要となる可能性もあります。自分が調べた限りでは WCAG や JIS X 8341-3 では回線速度についての直接の言及はない（違ったらすいません）のですが、現実的としては配慮しているケースも多く、国内の公共の Web ページのサイトポリシーなどで言及されていることも珍しくありません。いくつか例を紹介します。

環境省 / ウェブサイト作成ガイドライン

> 配慮の対象となる利用者 - "ダイヤルアップ接続など通信速度が速くない環境で利用している"

https://www.env.go.jp/other/gyosei-johoka/web_gl/02_1.html

岐阜県 / 岐阜県ウェブアクセシビリティに関するガイドライン

> そのほか配慮が必要な人 - "通信速度が遅い（速くない）環境で利用している人"

https://www.pref.gifu.lg.jp/page/6842.html

豊川市 / ウェブアクセシビリティへの取り組み

> 主な配慮事項 - "通信速度の遅い回線を使っている方でも画像やファイル等の画像表示が遅くならないように、画像やファイルの容量をできるだけ小さくする"

https://www.city.toyokawa.lg.jp/aboutweb/webaccessibility.html

2024 年 4 月以降は改正された障害者差別解消法が施行され、事業者としては Web アクセシビリティへの対応も一定の義務が発生します。
https://www8.cao.go.jp/shougai/suishin/sabekai.html

そういった背景を踏まえると、通信回線が遅い環境への対処を一概に無視することはできず、結果的に Progressive Enhancement を意識した設計・実装がエンジニアにも求められてくるのではないかと思います。

## ユーザビリティ観点では？

ユーザビリティ観点での Progressive Enhancement はどうでしょうか。

書籍 [Web アプリケーションアクセシビリティ](https://gihyo.jp/book/2023/978-4-297-13366-5)では、ユーザビリティは JIS Z 8521:2020 より抜粋し次のように説明されていました。

> 特定のユーザが特定の利用状況において，システム，製品又はサービスを利用する際に，効果，効率及び満足を伴って特定の目標を達成する度合い。

アクセシビリティ上で課題がないようなユーザーに対して、よりリッチであったり快適なサービス体験の提供などが含まれそうです。

Progressive Enhancement に基づいて実装した場合、必然的にサーバーからは JavaScript がない状態でも動作可能な HTML を返す必要があるため、Core Web Vitals に含まれる FID および CLS の改善が期待できます。これは回線速度などに不都合のないユーザにとっても恩恵を受けられるものであり、純粋にユーザー体験の向上に寄与しそうです。

# 現実的な実装観点からの Progressive Enhancement

では現実的に Progressive Enhancement を実現するには何をすべきなのでしょうか。

実際問題として、現代のフロントエンド開発において JavaScript を一切用いないことを前提として完全なサービス提供を行うのはかなり厳しいものがあります。

ブラウザ側でも Popover API や `<selectlist>` といった新しい機能が日々追加され、従来 JavaScript が必須であった機能について、ネイティブで提供可能になるケースもあります。

参考: JavaScript なしで動作するモダンなコードの書き方 / [@azukiazusa](https://twitter.com/azukiazusa9) さん
https://speakerdeck.com/azukiazusa1/javascript-nasidedong-zuo-surumodannakodonoshu-kifang

しかし、現実的なユースケースとしてはまだ十分出揃っているとは言えず、ほとんどのケースで何らかの実装は必要になります。また、仮に JavaScript を用いずにネイティブの機能で実現したとしても、その機能は古いブラウザではサポートされていないことも多いため、ある意味 Progressive Enhancement とは逆行する可能性も含んでいます。

## Progressive Enhancement をどこまで追求するか？

実際のところ、たとえば Remix や Next.js の機能を活用することで、フォーム操作などの点において、ある程度 Progressive Enhancement に沿った形での実装は可能かと思います。

しかし、それはごく一部だけを切り取った形になります。

〜ここを書く〜

# まとめ

というわけで、Progressive Enhancement について考えてみた内容を書き綴ってみました。

少々主観の部分も含まれているため、もしかしたら間違いもあるかもしれません。
