---
title: "Next.js 13 の cache 周りを理解する - Automatic fetch() Request Deduping"
emoji: "🚚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js", "フロントエンド"]
published: false
publication_name: "cybozu_frontend"
---

# Next.js 13 App Router の cache はむずかしい

Next.js 13 以降 [App Router](https://beta.nextjs.org/docs/app-directory-roadmap) と呼ばれる、`app/` ディレクトリを起点とする新たなレイアウト・レンダリング機能が導入されました。

併せて、レンダリングを効率化するためのキャッシュ機構も大きく手を加えられました。
基本的には意識せずとも恩恵を受けられるものが多いですが、把握しておかないと意図しない描画に繋がる可能性もあるため、App Router を利用する場合には抑えておきたいところです。

App Router に対応している beta 版ドキュメント^[https://beta.nextjs.org/docs/getting-started]に基本的な情報は記載されているため、
その内容をベースに、実際にサンプルアプリケーションで動作を見つつ、内容を確認してみます。

なお、「キャッシュ」と言っても実際にはさまざまな機能が存在するため、記事ごとに１機能にフォーカスして確認していきます。

# Automatic fetch() Request Deduping / fetch() の自動重複排除

Next.js 13 では **Automatic fetch() Request Deduping** と呼ばれる機能が含まれます。

ページ描画のために必要なデータを Server Components から fetch で取得する際、コンポーネントツリー内で同一のデータを取得しようとすると、自動的に重複が排除され、最適化された回数だけリクエストが発行される仕組みです。

https://beta.nextjs.org/docs/data-fetching/fundamentals#automatic-fetch-request-deduping

今回の記事では、この機能を中心に確認してみます。

まずはダミーの API を用意します。簡易的なもので良いので、他の Next.js でアプリケーションを立ち上げ、Route Handlers を定義します。
`app/sample-api/route.ts` を用意し、毎回ランダムな値を出力し、かつ実行されたことがわかるようにログを出力します。

```ts:app/sample-api/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  console.log(">>> Called /api/sample-api !!!");
  return NextResponse.json({ data: Math.random() });
}
```

ページはネストした形とし、すべての `layout.tsx` `page.tsx` から、用意した API を fetch() 経由で呼び出し、レスポンスをコンテンツとして描画します。

```text:ディレクトリ構造
app
├── layout.tsx
├── page.tsx
└── foo
    ├── layout.tsx
    ├── page.tsx
    └── bar
        ├── layout.tsx
        └── page.tsx
```

```ts:APIコール関数
export const fetchData = async () => {
  // 別のNext.jsでPort=3001でAPIが起動している想定
  const res = await fetch("http://localhost:3001/sample-api");
  const { data } = await res.json();
  return data;
};
```

```ts:layout.tsxの例
import { fetchData } from "@/lib/fetchData";

export default async function BarLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const data = await fetchData();

  return (
    <div className="bg-gray-200 p-4 border-2 border-gray-400 rounded">
      Bar Layout : {data}
      {children}
    </div>
  );
}
```

```ts:page.tsxの例
import { fetchData } from "@/lib/fetchData";

export default async function Bar() {
  const data = await fetchData();

  return (
    <div className="bg-blue-300 p-4 border-2 border-gray-400 rounded">
      Bar : {data}
    </div>
  );
}
```

この状態で Next.js を起動し、 `/foo/bar` にアクセスしてみると、API からはランダム値で返される値が、レンダリング結果ではすべて同一値であることがわかります。

![](/images/next-caching/dedupe-ui.png)

また、ログ出力を確認してみても、API へのアクセスは一度しか来ていないことが確認できます。

![](/images/next-caching/dedupe-console.png)

Next.js 13 App Router では、Server Components を組み合わせた効率の良いレンダリングのため、レイアウトとページ間でのデータ共有は行わず、代わりに利用したい箇所で適宜データフェッチすることが推奨^[https://beta.nextjs.org/docs/rendering/server-and-client-components#sharing-fetch-requests-between-server-components]されています。

そこで、Automatic fetch() Request Deduping の存在により、同一 fetch が複数回実行されることによるコストが自動的に軽減されるようになっています。

# fetch() のオプションと組み合わせるとどうなるのか？

ところで、Next.js では fetch() を拡張しており、`options.cache` 指定によりサーバサイドで fetch() が実行された場合のキャッシュの動作をコントロールできます。
https://beta.nextjs.org/docs/api-reference/fetch#optionscache

オプションには `force-cache` または `no-store` を指定できます。`force-cache` の場合はキャッシュを利用して値を返そうとしますが、`no-store` の場合にはキャッシュは利用されません。そしてデフォルトの値は `force-cache` です。

では、もし fetch() のオプションに `no-store` を指定した場合、Automatic fetch() Request Deduping はどのように動作するのでしょうか？

実際に指定して確認してみます。API の呼び出しに `no-store` 指定を追加します。

```ts:APIコール関数
export const fetchData = async () => {
  const res = await fetch("http://localhost:3001/sample-api", {
    cache: "no-store",
  });
  const { data } = await res.json();
  return data;
};

```

すると、単一のリクエスト上では fetch() の重複は排除されていますが、リロードすると再度 API が実行されることが確認できます。
（なお、`no-store` を付与していなかった場合は、リロードしても同じ値が表示されます）

![](/images/next-caching/dedupe-no-store.gif)

つまり、fetch() 自体の cache 設定と Automatic fetch() Request Deduping の動作は干渉せず、仮に `no-store` でキャッシュを無効化していたとしても、fetch() の重複排除は動作するようです。

# fetch() が利用できない場合

Automatic fetch() Request Deduping は GET でしか動作しないなど、一部制約があります。

しかし、たとえば次のようなケースでも似たように重複排除が欲しくなる可能性があります。

- GraphQL エンドポイントなど、POST が必要である
- fetch()を介さず、直接 DB アクセスなどでデータを取得する

そういった場合、React が提供する `cache()` 関数を利用することで、同等のリクエスト単位でのキャッシュを実現できます。

https://beta.nextjs.org/docs/data-fetching/caching#per-request-caching

---

というわけで、Next.js のキャッシュ機構から Automatic fetch() Request Deduping の話でした。

キャッシュに関しては revalidate 周りも抑えておいたほうが良いため、そちらは別途記事として公開予定です。
