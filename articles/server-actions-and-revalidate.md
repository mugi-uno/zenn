---
title: "Next.js Server Actions と revalidate 周りの挙動を確認する"
emoji: "🦑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js", "フロントエンド", "React"]
published: true
publication_name: "cybozu_frontend"
---

# revalidatePath & revalidateTag

Next.js 13.4 から、Next.js App Router で利用可能な新しい API として `revalidatePath` と `revalidateTag` の２つが追加されました。

https://nextjs.org/docs/app/api-reference/functions/revalidatePath

https://nextjs.org/docs/app/api-reference/functions/revalidateTag

以前（Pages Router）からも任意タイミングでの revalidate のために On-demand ISR という手法が可能でしたが、これは revalidate 用にエンドポイントを用意した上で、特定パスの revalidate を行うというものでした。App Router ではセグメントや fetch 単位でキャッシュ制御を行うため、revalidate においても `revalidatePath` `revalidateTag` を用いることで、より柔軟に対応できます。

# revalidatePath の動きをみてみる

`revalidatePath` や `revalidateTag` は、それ単体では指定したパスやタグに対して revalidate を行うというシンプルなものです。

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

少しわかりづらいですが、`revalidatePath` に指定したパスだけが適切に revalidate されています。また、Root のレイアウトは共通であるため、いずれかで revalidate されると、もう一方のページでも最新化されます。Pages Router のときの On-demand ISR と似たような体験ですね。

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

この Route Handler にアクセスしてみると、指定したタグに応じてピンポイントで revalidate できていることがわかります。

![](/images/sa-revalidate/revalidatetag.gif)

パスではなくクエリ単位となるため、中間に位置するレイアウトなどを対象もできるのが `revalidatePath` とは大きく違うところですね。

# Server Actions と組み合わせる

上記のサンプルでは Route Handler から `revalidatePath` `revalidateTag` を呼び出していますが、Next.js 13.4 から同じくアルファ機能として追加された Server Actions からも呼び出すことができます。

Server Actions と `revalidatePath` `revalidateTag` を組み合わせると、**いま現在表示している画面に対して revalidate に応じた更新を行う**ことができます。

具体的に見てみましょう。

ルートレイアウト - レイアウト - ページ の構成とし、それぞれで API を別の `tags` を付与して fetch() 実行し描画します。ページは `/action` とし `tags` は次の通り指定します。

- Root レイアウト : `['root']`
- `/action` レイアウト : `['layout']`
- `/action` ページ: `['page']`

そして、`/aciton` ページで Server Actions を実行する Server Component を描画します。Server Actions では、`'layout'` タグを対象に `revalidateTag` を呼び出します。コードは次の通りです。

```tsx:ActionComponent.tsx
import { revalidateTag } from "next/cache";

export const ActionComponent = () => {
  async function action() {
    "use server";

    revalidateTag("layout");
  }

  return (
    <form action={action}>
      <button
        type="submit"
        className="p-2 border-gray-400 bg-white border rounded"
      >
        Submit Action
      </button>
    </form>
  );
};
```

すると、次のような表示となります。

![](/images/sa-revalidate/sa-default.png)

そしてこの状態で Submit ボタンを押してみると...

![](/images/sa-revalidate/sa-revalidatetag.gif)

クリックするたびに Layout (`/action` のレイアウト) が更新されています。また、フルリロードはしておらず、React Server Components によって、差分のみが再描画されていることがわかります。

Route Handlers 経由で `revalidateTag` `revalidatePath` を呼び出した際は、該当のページに対して次回アクセスした際にキャッシュを最新化する動作になりますが、Server Actions の場合、今見ているページが即座に更新されているのが大きく違う点になりそうです。

## 使い道

ではどういったときに役に立つの？という話ですが、具体的な利用ケースをイメージするとわかりやすいかもしれません。

たとえば、アプリケーションのヘッダーにログインユーザーの名前が常に表示されており、それを設定画面で更新するケースを想定してみましょう。GitHub の Settings のようなページをイメージすると良いかもしれません。

![](/images/sa-revalidate/sa-sample.png)

このとき、次の要件の実装が必要だと仮定します。

- Name を更新できるが、更新時には画面遷移はせずに留まる
- 更新後は右上のメニュー内に表示されている Name も最新化する

これを Server Actions なしで実装しようとすると、メニューの更新処理が少し手間になります。

更新用のフォームとヘッダーをそれぞれコンポーネントとして組んでいくと、コンポーネントツリー上での位置関係が遠い存在になることが多いはずです。しかし、ヘッダー側を何らかの方法で最新化しなければなりません。いくつか方法が考えられます。

- Context を使う
- 何らかの状態管理ライブラリで連携させる
- React Query などを利用し、キャッシュを破棄する
- pub/sub のような仕組みでイベントを通知してヘッダー側で拾う

いずれにしても、それなりにコストがかかりますし、このためだけにライブラリを入れるのもな...という気持ちにもなりますね。

これが Server Actions の場合、たとえばヘッダー側でのデータ取得用 fetch() 時に `"userData"` というタグを付与しておけば、更新処理の Server Actions 側で `revalidateTag("userData")` を呼び出すだけで自動的にヘッダーも更新できます。とても簡単ですね。

ただ、いたるところでこの操作を自由に行うと、操作に対してどこが更新されるのかが追えなくなる可能性も抱えていそうです。タグは整理して型付けしておくなど、無秩序にならないための工夫は必要になるかもしれません。

しかし、うまく使えば非常に効率的にアプリケーションを構築していけるポテンシャルは持っているように感じます。Server Actions は 2023/06/08 現在ではまだアルファ版ですが、個人的にはとても期待しています。
