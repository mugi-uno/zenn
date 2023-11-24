---
title: "Server Actions の React と Next.js のそれぞれの役割はなんなのか"
emoji: "💥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "フロントエンド", "React"]
published: false
publication_name: "cybozu_frontend"
---

# Server Actions のコア実装は React 本体が持っている

Next.js 14.x で Stable となった Server Actions ですが、Server Actions のコアとなる実装部分は React 本体に含まれています。

実際、`'use server'` のドキュメントも存在しています。

https://react.dev/reference/react/use-server

React 側では、 `18.3.0-canary-546178f91-20231005` の Canary リリースに含まれています。

> **余談: Next.js が Stable で React が Canary なのは大丈夫なの？**
>
> Next.js 側が Stable と言っているのに React が Canary なのは大丈夫なの？という気持ちになりますが、React における Canary release channel を明確に説明するブログ記事が存在します。
> https://react.dev/blog/2023/05/03/react-canaries
>
> 概略になりますが、Canary リリースに含まれる内容は、将来的な正式採用に向けた準備が整っているものであり、万が一リグレッションや致命的な問題が発生した場合には、Stable で発生したものと同様の扱いで対応されます。致命的な変更の可能性がある実験的なリリースは、Canary ではなく [Experimental channel](https://react.dev/community/versioning-policy#experimental-channel) に含まれます。各種フレームワーク・ライブラリ側で React での個々の新機能を先行して採用し実装するのが Canary release channel の目的の一つであり、Server Actions が Next.js では Stable であり React では Canary というのは、この説明に基づいており、それほど違和感が無いことがわかります。

# React 本体が Server Actions を持ってるってどういうこと？

Server Actions は RPC のようにクライアント側から実行できる機能ですが、それを React 本体が持っている、というのはどういうことなのでしょうか？
主観ですが、"Web アプリケーション構築用のフレームワークである Next.js の機能です" と言われたほうが違和感がありません。

ということで、Server Actions というものについて、React 本体が提供する部分と、それを利用する Next.js が提供する部分が一体何なのかを追ってみましょう。

# コード上の境界はどこか？

# まとめ

あばば
