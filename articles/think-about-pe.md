---
title: "Progressive Enhancement について考える"
emoji: "🚶‍♀️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["フロントエンド", "React"]
published: false
publication_name: "cybozu_frontend"
---

# Progressive Enhancement

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

# JavaScript が無効な状態でもサポートすれば OK？

「そうか！じゃあ Progressive Enhancement に徹底的にこだわろう！」と思いたくもなりますが、現実的にどこまで考慮すべきなのか？というのは難しい問題です。

主観ですが、Progressive Enhancement について調べたり話したりしていると、暗黙的に "Progressive Enhancement = JavaScript が無効化されている環境でも動く" という前提になることが多い印象です。

しかし、現実的に JavaScript が完全に無効なケースはどこまで考えるべきでしょうか。たとえば、Web アクセシビリティに関するサーベイなどを行っている WebAIM の 2019 年時点の調査では、スクリーンリーダー利用ユーザーのうち、JavaScript を無効化しているユーザーの割合は全体のうちの 0.7% だったそうです。
https://webaim.org/projects/screenreadersurvey8/#javascript

これを多いと見るか少ないと見るかは様々な前提条件によって変わりそうではあります。

ただ、Progressive Enhancement が提唱された当時は JavaScript が現在ほど活用されておらず、JavaScript が無効化されているシチュエーションを想定する比重は高かったのではないかと予想されます。しかし、現代においてはかなり状況が変わっており、当時の思想をそのまま適用すると少し歪みが生まれるように感じます。

# アクセシビリティ観点での Progressive Enhancement

# ユーザビリティ観点での Progressive Enhancement

# SEO 観点での Progressive Enhancement

# 現実的な実装観点からの Progressive Enhancement

# Progressive Enhancement をどこまで追求するか？

# まとめ
