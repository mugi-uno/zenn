---
title: "現代の Progressive Enhancement について考える"
emoji: "🚶‍♀️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["フロントエンド", "アクセシビリティ"]
published: true
publication_name: "cybozu_frontend"
---

この記事は [Cybozu Frontend Advent Calendar 2023](https://adventar.org/calendars/9255) の 6 日目の記事です。

---

最近になって、**Progressive Enhancement（プログレッシブ・エンハンスメント）** というワードを目にする機会が増えています。

Remix や Next.js といったフレームワークにおいても頻繁に登場します。

https://remix.run/docs/en/main/discussion/progressive-enhancement

https://nextjs.org/docs/app/api-reference/functions/server-actions#progressive-enhancement

Progressive Enhancement という概念は最近になって登場したものではなく、Wikipedia によると最初に登場したのは 2003 年のことだそうです。

https://ja.wikipedia.org/wiki/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%AC%E3%83%83%E3%82%B7%E3%83%96%E3%82%A8%E3%83%B3%E3%83%8F%E3%83%B3%E3%82%B9%E3%83%A1%E3%83%B3%E3%83%88

今回はこの "Progressive Enhancement" について少し考えてみたいと思います。

# Progressive Enhancement とは

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

しかし、現実的に JavaScript が完全に無効なケースはどこまで考えるべきでしょうか。たとえば、JavaScript の利用状況に関してはさまざまな調査が行われており、いずれにしても無効化している割合はかなり小さいという結果が出ているようです。いくつか例を紹介します。

**Blockmetry**
2016 年時点で全世界のトラフィックの 0.2% で JavaScript が無効
https://deliberatedigital.com/blockmetry/javascript-disabled

**gov.uk (イギリス政府)**
2013 年時点でイギリス政府のウェブサービス利用者のうち 1.1%が JavaScript の利用なし（明示的な無効化は 0.2%）
https://gds.blog.gov.uk/2013/10/21/how-many-people-are-missing-out-on-javascript-enhancement/

**WebAIM**
2019 年時点でスクリーンリーダー利用ユーザーのうち 0.7% が JavaScript を無効化
https://webaim.org/projects/screenreadersurvey8/#javascript

これらを多いと見るか少ないと見るかは様々な前提条件によって変わりそうではあります。

ただ、Progressive Enhancement が提唱された当時は JavaScript が現在ほど活用されておらず、ブラウザの設定によって JavaScript が無効化されているシチュエーションを想定する比重は高かったのではないかと予想されます。しかし、現代においてはかなり状況が変わっており、当時の思想をそのまま適用すると少し歪みが生まれるようにも感じます。

## 「JavaScript が無効」とは

現代ではほぼすべてのユーザーが JavaScript が利用可能な環境でブラウジングをしていると考えても差し支えないかもしれませんが、一方で、JavaScript が動作していないシチュエーション自体は日常的にありえます。

Remix の Progressive Enhancement のページには次の一文があります。

> While your users probably don't browse the web with JavaScript disabled, everybody has JavaScript disabled until it has finished loading.
> ChatGPT による翻訳: ユーザーの多くは JavaScript を無効にしてウェブを閲覧することはないでしょうが、JavaScript が完全に読み込まれるまでは、全員が JavaScript を無効にしています。

常時 JavaScript が無効かどうか？とはまた別の話として、ページが読み込まれて初期化が完了するまでは JavaScript に期待する動作は実現できません。この観点で考えると、特定のユーザーに限ったものではなく、すべてのユーザーにとって関係のある話になってきます。

## JavaScript だけではない

Remix や Next.js の観点だと JavaScript を中心にした話になりがちですが、Progressive Enhancement は JavaScript だけケアするものではありません。

たとえば、古い環境での動作を想定する場合には CSS についても考慮が必要です。モダンな CSS が利用できない環境では大幅に表示が崩れてしまい、まともに操作ができない...といった状態になると、Progressive Enhancement とは乖離します。

# アクセシビリティ観点での Progressive Enhancement

アクセシビリティ観点で Progressive Enhancement を考えてみます。

まず、Progressive Enhancement を実現しようと試みた場合、伴って一定のアクセシビリティが担保されやすいと思われます。フォームでの入力・送信を例に考えた場合、仮に "ボタンの `click` イベントで form の `submit()` を実行" といった実装をしていると、JavaScript が有効化される前の段階では動作しません。そのため、できるかぎり純粋な `<form>` タグや `<input type="submit">` などを用いる必要が出てきます。バリデーションなども同様で、最低限のものであれば HTML で利用可能なフォームバリデーション（`require` 属性、`type` 属性、など）を用いて実現しなければいけません。結果的に基本的なマークアップでの実装が増えることで、ブラウザが提供する標準的なアクセシビリティの動作に沿うことができます。

また、通信回線が遅い環境でのユーザーへの配慮がアクセシビリティ文脈で必要となる可能性もあります。自分が調べた限りでは WCAG や JIS X 8341-3 では回線速度についての直接の言及はない（違ったらすいません）のですが、現実には配慮しているケースも多く、国内の公共の Web ページのサイトポリシーなどで言及されていることも珍しくありません。いくつか例を紹介します。

**環境省**
ウェブサイト作成ガイドライン

> 配慮の対象となる利用者 - "ダイヤルアップ接続など通信速度が速くない環境で利用している"
> https://www.env.go.jp/other/gyosei-johoka/web_gl/02_1.html

**岐阜県**
岐阜県ウェブアクセシビリティに関するガイドライン

> そのほか配慮が必要な人 - "通信速度が遅い（速くない）環境で利用している人"
> https://www.pref.gifu.lg.jp/page/6842.html

**豊川市**
ウェブアクセシビリティへの取り組み

> 主な配慮事項 - "通信速度の遅い回線を使っている方でも画像やファイル等の画像表示が遅くならないように、画像やファイルの容量をできるだけ小さくする"
> https://www.city.toyokawa.lg.jp/aboutweb/webaccessibility.html

2024 年 4 月以降は改正された障害者差別解消法が施行され、事業者としては Web アクセシビリティへの対応も一定の義務が発生します。
https://www8.cao.go.jp/shougai/suishin/sabekai.html

そういった背景を踏まえると、通信回線が遅い環境への対処を一概に無視することはできず、結果的に Progressive Enhancement を意識した設計・実装がエンジニアにも求められてくるのではないかと思います。

## ユーザビリティ観点では？

ユーザビリティ観点での Progressive Enhancement はどうでしょうか。

書籍 [Web アプリケーションアクセシビリティ](https://gihyo.jp/book/2023/978-4-297-13366-5)では、"ユーザビリティ" の定義は JIS Z 8521:2020 より抜粋し次のように説明されていました。

> 特定のユーザが特定の利用状況において，システム，製品又はサービスを利用する際に，効果，効率及び満足を伴って特定の目標を達成する度合い。

アクセシビリティ上で課題がないようなユーザーに対して、よりリッチであったり快適なサービス体験の提供などが含まれそうです。

Progressive Enhancement に基づいて実装した場合、必然的にサーバーからは JavaScript がない状態でも動作可能な HTML を返す必要があるため、Core Web Vitals に含まれる FID および CLS の改善が期待できます。これは回線速度などに不都合のないユーザにとっても恩恵を受けられるものであり、純粋にユーザー体験の向上に寄与しそうです。

# 現実的な実装観点からの Progressive Enhancement

では現実的に Progressive Enhancement を実現するには何をすべきなのでしょうか。

実際問題として、現代のフロントエンド開発において JavaScript を一切用いないことを前提として完全なサービス提供を行うのはかなり厳しいものがあります。

ブラウザ側でも Popover API や `<selectlist>` といった新しい機能が日々追加され、JavaScript を使わずにブラウザ自体から提供可能なものも増えつつあります。

参考: JavaScript なしで動作するモダンなコードの書き方 / [@azukiazusa](https://twitter.com/azukiazusa9) さん
https://speakerdeck.com/azukiazusa1/javascript-nasidedong-zuo-surumodannakodonoshu-kifang

しかし、現実的なユースケースとしてはまだ十分出揃っているとは言えず、ほとんどのケースで JavaScript を用いた何らかの独自実装は必要になります。

また、仮に JavaScript を用いずにネイティブの機能で実現したとしても、その機能は古いブラウザではサポートされていないことも多いため、「古いブラウザでも動かしたい」というモチベーションから考えると、ある意味 Progressive Enhancement とは逆行することになります。しかし、Progressive Enhancement の捉え方が「JavaScript がロードされる前（Hydration 前など）のタイミングでも動くようにしたい！」といったものであれば有効に活用できます。

何をしたいか、背景事情によって実現方法も変わってきそうです。

## フレームワークの機能で Progressive Enhancement の対応は OK？

Next.js や Remix などのフレームワークが提供する機能を活用することで、フォーム操作など、ある程度 Progressive Enhancement に沿った形での実装は可能です。しかし、それで完璧かと言われるとそんなこともなく、ごく一部だけを切り取った形になってしまう可能性があります。

Next.js App Router を例に挙げると、Server Actions を有効活用することで、JavaScript が無効な状態でもフォームの動作を実現できます。しかし、他の機能の大半が Client Components を用いてクライアント側での JavaScript 動作を前提とした実装であった場合、Progressive Enhancement が実現されているとは呼べない状態になります。

また、先述の通り CSS なども含めて一貫した設計が必要で、フレームワークの特定の機能を使ったからオッケー！というものでもありません。

順序としては、まず大前提に「サービスとして何を実現すべきなのか・実現したいのか」が存在しており、そのための手段として Progressive Enhancement が存在する、と考えたほうが良さそうです。

なお、先日公開されたこちらの記事も大変参考になる内容でした。

https://oisham.hatenablog.com/entry/2023/12/03/234541

> つまり、 仕様自体も 0 か 1 かではなく、状況に応じた仕様の定義が必要になる と考えています。

# まとめ

少々主観の部分も含まれているため、もしかしたら間違いもあるかもしれませんが、Progressive Enhancement について考えてみた内容を書き綴ってみました。

ライブラリやフレームワークなど、技術寄りの視点で謳われる Progressive Enhancement というワードに意識が持っていかれがちですが、実際のところは仕様や要件といった部分に大きく影響を受ける点は注意しておきたいところです。また、Web エンジニアとしてはその仕様や要件に応じて、Progressive Enhancement を実現する手段が何かは抑えておく必要はありそうだな〜と感じました。特にアクセシビリティ周りが代表的なもので、努力目標ではなく必達要件になる可能性もあるため、必須知識として求められるケースも出てくるかもしれません。

むずかしい〜〜
