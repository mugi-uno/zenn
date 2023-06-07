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

App Router ではない Pages Router のときでも任意タイミングでの revalidate のために On-demand ISR という方法が可能でしたが、これは revalidate 用にエンドポイントを用意した上で、特定パスの revalidate を行うというものでした。これで事足りるケースもありますが、App Router ではセグメントや fetch 単位でキャッシュ制御を行うため、revalidate においても `revalidatePath` `revalidateTag` を用いることで、より柔軟に対応できます。

# revalidatePath の動きをみてみる

`revalidatePath` や `revalidateTag` は、それ単体では指定したタグやパスに対して revalidate を行うというシンプルなものです。

まずは `revalidatePath` の動きを見てみましょう。

`/foo` ページと、`/bar` ページを用意し、それぞれ API から取得したデータ（タイムスタンプ）を表示しています。常時リロードしていますが、fetch のデフォルトの動作は `force-cache` ^[fetch のオプションの詳細については[別の記事](https://zenn.dev/cybozu_frontend/articles/next-caching-dedupe)でも解説しています]となるため、何度アクセスしても内容が変化しないことがわかります。また、共通となる Root のレイアウトは API が共通のため同じ表示となっています。

![](/images/sa-revalidate/revalidate-init.gif)

ではここで、`revalidatePath` を呼び出すための Route Handlers を定義してみます。

```ts:revalidate/route.ts
import { NextRequest, NextResponse } from "next/server";
import { revalidatePath } from "next/cache";

export async function GET(request: NextRequest) {
  const path = request.nextUrl.searchParams.get("path") || "/";
  revalidatePath(path);
  return NextResponse.json({ revalidated: true, now: Date.now() });
}
```

クエリパラメータに付与された path の値に対してそのまま `revalidatePath` を発行しているだけです。この状態で、`/revalidate?path=/foo` または `/revalidate?path=/bar` にアクセスしてみます。

![](/images/sa-revalidate/revalidatepath.gif)

`revalidatePath` に指定したパスだけが適切に revalidate されていることがわかります。また、Root のレイアウトは共通であるため、いずれかで revalidate されると、もう一方のページでも最新化されます。Pages Router のときの On-demand ISR と似たような体験ですね。

# revalidateTag の動きをみてみる

では続いて revalidateTag のほうを見てみましょう。

Next.js 13.4 以降、fetch のオプションに `tags` を指定可能になっています。すべての fetch の実行に対して `tags` を指定しておきます。

```ts:fetchにtagを指定する例
await fetch("http://localhost:3001/sample-api?foo", {
  next: { tags: ["foo-page"] },
});
```

呼び出し箇所に応じて付与する`tags`は次のとおりです。

- Root レイアウト : `['root']`
- `/foo` レイアウト : `['layout', 'foo-layout']`
- `/foo` ページ: `['foo-page']`
- `/bar` レイアウト: `['layout', 'bar-layout']`
- `/bar` ページ: `['bar-page']`

そして、`revalidateTag` を呼び出すための Route Handler を用意します。`revalidatePath` のサンプルと似た感じで、パラメータでタグを受け取ってそのまま `revalidateTag` を呼び出します。

```ts:revalidate-tag/route.ts
import { revalidateTag } from "next/cache";
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  const tag = request.nextUrl.searchParams.get("tag") || "/";
  revalidateTag(tag);
  return NextResponse.json({ revalidated: true, now: Date.now() });
}
```

指定したタグに応じてピンポイントで revalidate できていることがわかります。

![](/images/sa-revalidate/revalidatepath.gif)

パスではなくクエリ単位となるため、中間に位置するレイアウトなどを対象もできるのが `revalidatePath` とは大きく違うところですね。

# Server Actions と組み合わせる

上記のサンプルでは Route Handler から `revalidatePath` `revalidateTag` を呼び出していますが、Next.js 13.4 から同じくアルファ機能として追加された Server Actions からも呼び出すことができます。

Server Actions から呼び出した場合、Router Handler 経由での呼び出しとは異なり、**いま現在表示している画面内の要素に対して revalidate に応じた更新を行う**ことができます。
