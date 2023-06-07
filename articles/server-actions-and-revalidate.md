---
title: "Next.js Server Actions と revalidate 周りの挙動を確認する"
emoji: "🦑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js", "フロントエンド", "React"]
published: false
publication_name: "cybozu_frontend"
---

# revalidatePath & revalidateTag

Next.js 13.4 から、Next.js App Router で利用可能な新しい API として `revalidatePath` と `revalidateTag` の２つが追加されました。

https://nextjs.org/docs/app/api-reference/functions/revalidateTag

App Router 以前でも任意タイミングでの revalidate のために On-demand ISR という方法が可能でしたが、これは revalidate 用にエンドポイントを用意した上で、特定パスの revalidate を行うというものでした。これで事足りるケースもありますが、App Router ではセグメントや fetch 単位でキャッシュ制御を行うため、revalidate においても `revalidatePath` `revalidateTag` を用いることで、より柔軟に対応できます。

# revalidatePath の動きをみてみる

`revalidatePath` や `revalidateTag` は、それ単体では指定したタグやパスに対して revalidate を行うというシンプルなものです。

まずは `revalidatePath` の動きを見てみましょう。

`/foo` ページと、`/bar` ページを用意し、それぞれ API から取得したデータ（タイムスタンプ）を表示しています。常時リロードしていますが、fetch のデフォルトの動作は `force-cache` となるため、何度アクセスしても内容が変化しないことがわかります。^[fetch のオプションの詳細については[別の記事](https://zenn.dev/cybozu_frontend/articles/next-caching-dedupe)でも解説しています]

![](/images/sa-revalidate/revalidate-init.gif)

# 別 URL の revalidatePath

# 別 URL の revalidateTag

# 今見ている画面の reavlidatePath

# 今見ている画面の revalidateTag
