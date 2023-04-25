---
title: "Next.js 13 の cache 周りを理解する - fetchCache"
emoji: "🚚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js", "フロントエンド"]
published: false
publication_name: "cybozu_frontend"
---

Next.js 13 App Router の cache 周りを理解したい記事シリーズです。

1. [Automatic fetch() Request Deduping](https://zenn.dev/cybozu_frontend/articles/next-caching-dedupe)
1. [revalidate](https://zenn.dev/cybozu_frontend/articles/next-caching-revalidate)
1. fetchCache ← この記事

# Route Segment Config - `fetchCache` オプション

App Router では、[Route Segment Config](https://beta.nextjs.org/docs/api-reference/segment-config)という仕組みで、レイアウトあるいはページから特定の変数を export すると動作をカスタマイズできます。[前回の記事](https://zenn.dev/cybozu_frontend/articles/next-caching-revalidate)では、このオプションのひとつである `revalidate` について動作を確認しました。

しかし、Route Segment Config には他にもキャッシュに関係するオプションがあります。

それが **[`fetchCache`](https://beta.nextjs.org/docs/api-reference/segment-config#fetchcache)** オプションです。

このオプションに関して、beta ドキュメント上には次の記載があります。

> This is an advanced option that should only be used if you specifically need to override the default behavior.
> DeepL 翻訳: これは高度なオプションであり、特にデフォルトの動作を上書きする必要がある場合にのみ使用する必要があります。

本当に必要な特別なケースでのみの利用を推奨されているようです。その前提を頭の片隅に残しつつ、実際にどういうオプションか見てみましょう。

# Dynamic Functions と `fetch()`

`fetchCache` オプションを理解するためには、**Dynamic Functions** と呼ばれる関数群と `fetch()` の関係性を把握しておく必要があります。

Next.js 13 App Router では、Cookie や HTTP ヘッダ情報といった、リクエストスコープに紐づく情報を取得するための関数をいくつか提供しています。それらはドキュメント上では Dynamic Functions と呼ばれ、 `cookies()` や `headers()` といった関数が該当します。

そして、これらを利用した場合 `fetch()` のデフォルトのキャッシュ挙動が変化します。

`fetch()` はデフォルトでは cache:force-cache と同等の挙動で、すべてのクエリをキャッシュします。しかし、Dynamic Functions が呼ばれると、同一リクエスト内での以降の `fetch()` は cache:no-store に切り替わり、キャッシュを利用しなくなります。（※`fetch()` の cache オプションに関しては、[別の記事](https://zenn.dev/cybozu_frontend/articles/next-caching-dedupe)でも解説しています。）

実際に確認してみます。ネストしたレイアウト・ページで `fetch()` を実行しています。しかし、途中の `Foo Layout` 内での `fetch()` 実行前のタイミングで `cookies()` を呼び出しています。

- Root Layout
  - Foo Layout (`fetch()`前に `cookies()` を呼び出し)
    - Bar Layout
      - Bar Page

```tsx:FooLayoutのコード例
import { cookies } from "next/headers";

export default async function Layout({ children }: { children: React.ReactNode }) {
  // ★Dynamic Functions
  cookies();

  const res = await fetch("http://localhost:3001/sample-api?page=foo_layout");
  const { data } = await res.json();

  return (
    <div className="bg-gray-200 p-4 border-2 border-gray-400 rounded">
      Foo Layout : {data}
      {children}
    </div>
  );
}
```

すると、次のような描画結果となります。（確認のためリロードしています）

![](/images/next-caching/fetch-cache-1.gif)

`cookies()` の呼び出しより上位階層ではキャッシュが利用されていないことがわかります（レンダリングの順番としては下位階層から描画されるため、`fetch()` の呼び出しも下位階層のほうが先となります）。

これが `fetchCache` 理解の前提となる Dynamic Functions と `fetch()` の挙動です。

## `fetchCache` オプションのデフォルト動作

さて、Dynamic Functions と `fetch()` の関係を把握できましたが、この動作を制御するためのオプションが `fetchCache` です。

`fetchCache` には次のオプションを指定可能です。

- `'auto'`
- `'default-cache'`
- `'default-no-store'`
- `'only-cache'`
- `'only-no-store'`
- `'force-cache'`
- `'force-no-store'`

ひとつずつオプションの挙動を確認してみましょう。

# `fetchCache` 各オプションの説明

## ⚠️ 注意事項

Next.js 13.3 で手元で動作を確認しましたが、一部ドキュメントの通りに動作しないようです。
（コードも見てみましたが、不具合かもしれない...）

以下説明は、ドキュメント上での記述を解説したものであり、Next.js 13.3 以前のバージョンでは安定的に動作しない可能性があります。
現状では「将来的にこういうオプションが使えるかも」程度の温度感で見ていただくほうが良さそうです。

## `fetchCache = 'auto'` (default)

fetchCache に何も指定しなかった場合のデフォルトの挙動です。

- `fetch()` に指定された cache オプションを利用する
- `fetch()` に cache オプションが無い場合、Dynamic Functions の実行前後で挙動が変化する
  - 実行前の場合は force-cache 相当
  - 実行前の場合は no-store 相当

つまり、さきほどの Dynamic Functions と `fetch()` の動作の正体は、この `'auto'` 指定によるものでした。

整理すると次のようになります。

| fetch() - cache オプション | Dynamic Functions | キャッシュの利用 |
| -------------------------- | ----------------- | ---------------- |
| (指定なし)                 | なし              | キャッシュする   |
| force-cache                | なし              | キャッシュする   |
| no-store                   | なし              | キャッシュしない |
| (指定なし)                 | 実行後            | キャッシュしない |
| force-cache                | 実行後            | キャッシュする   |
| no-store                   | 実行後            | キャッシュしない |

## `fetchCache = 'default-cache'`

cache オプションを省略した場合、すべて force-cache 相当で動作させます。

- `fetch()` に指定された cache オプションを利用する
- `fetch()` に cache オプションが無い場合、Dynamic Functions の実行に関わらず force-cache 指定となる

※太字は `'auto'` 指定との差異

| fetch() - cache オプション | Dynamic Functions | キャッシュの利用   |
| -------------------------- | ----------------- | ------------------ |
| (指定なし)                 | なし              | キャッシュする     |
| force-cache                | なし              | キャッシュする     |
| no-store                   | なし              | キャッシュしない   |
| (指定なし)                 | 実行後            | **キャッシュする** |
| force-cache                | 実行後            | キャッシュする     |
| no-store                   | 実行後            | キャッシュしない   |

Dynamic Functions の実行後でもデフォルトではキャッシュ対象になる点が `'auto'` との大きい差異ですね。

一方で、cache オプションの指定自体は可能なので、任意箇所で no-store でキャッシュを無効化することは可能です。

## `fetchCache = 'default-no-store'`

cache オプションを省略した場合、すべて no-store 相当で動作させます。

- `fetch()` に指定された cache オプションを利用する
- `fetch()` に cache オプションが無い場合、Dynamic Functions の実行に関わらず no-store 指定となる

※太字は `'auto'` 指定との差異

| fetch() - cache オプション | Dynamic Functions | キャッシュの利用     |
| -------------------------- | ----------------- | -------------------- |
| (指定なし)                 | なし              | **キャッシュしない** |
| force-cache                | なし              | キャッシュする       |
| no-store                   | なし              | キャッシュしない     |
| (指定なし)                 | 実行後            | キャッシュしない     |
| force-cache                | 実行後            | キャッシュする       |
| no-store                   | 実行後            | キャッシュしない     |

`'default-cache'` がデフォルトを force-cache にするのに対して、その逆の動作になるイメージですね。

## `fetchCache = 'force-cache'`

すべての `fetch()` リクエストをすべてキャッシュ対象とします。`fetch()` に no-store オプションが指定されたとしてもキャッシュ対象になります。

- `fetch()` に指定された cache オプションは利用されない
- `fetch()` に cache オプションが無い場合、Dynamic Functions の実行に関わらず force-cache 指定となる

※太字は `'auto'` 指定との差異

| fetch() - cache オプション | Dynamic Functions | キャッシュの利用   |
| -------------------------- | ----------------- | ------------------ |
| (指定なし)                 | なし              | キャッシュする     |
| force-cache                | なし              | キャッシュする     |
| no-store                   | なし              | **キャッシュする** |
| (指定なし)                 | 実行後            | **キャッシュする** |
| force-cache                | 実行後            | キャッシュする     |
| no-store                   | 実行後            | **キャッシュする** |

## `fetchCache = 'force-no-store'`

`'force-cache'` の逆で、すべてをキャッシュしません。`fetch()` に force-cache オプションが指定されたとしてもキャッシュされません。

- `fetch()` に指定された cache オプションは利用されない
- `fetch()` に cache オプションが無い場合、Dynamic Functions の実行に関わらず no-store 指定となる

※太字は `'auto'` 指定との差異

| fetch() - cache オプション | Dynamic Functions | キャッシュの利用     |
| -------------------------- | ----------------- | -------------------- |
| (指定なし)                 | なし              | **キャッシュしない** |
| force-cache                | なし              | キャッシュしない     |
| no-store                   | なし              | **キャッシュしない** |
| (指定なし)                 | 実行後            | **キャッシュしない** |
| force-cache                | 実行後            | **キャッシュしない** |
| no-store                   | 実行後            | キャッシュしない     |

とにかくキャッシュしません。

## `fetchCache = 'only-cache'`

cache オプションを省略した場合、すべて force-cache 相当で動作させます。
また、cache オプションに no-store が指定された場合には `fetch()` がエラーとなります。

- `fetch()` に cache オプションが無い場合、Dynamic Functions の実行に関わらず force-cache 指定となる
- `fetch()` に指定された cache オプションが no-store の場合はエラーとなる

※太字は `'auto'` 指定との差異

| fetch() - cache オプション | Dynamic Functions | キャッシュの利用   |
| -------------------------- | ----------------- | ------------------ |
| (指定なし)                 | なし              | キャッシュする     |
| force-cache                | なし              | キャッシュする     |
| no-store                   | なし              | **(エラー)**       |
| (指定なし)                 | 実行後            | **キャッシュする** |
| force-cache                | 実行後            | **キャッシュする** |
| no-store                   | 実行後            | **(エラー)**       |

## `fetchCache = 'only-no-store'`

cache オプションを省略した場合、すべて no-store 相当で動作させます。
また、cache オプションに force-cache が指定された場合には `fetch()` がエラーとなります。

`'only-cache'` の逆ですね。

- `fetch()` に cache オプションが無い場合、Dynamic Functions の実行に関わらず no-store 指定となる
- `fetch()` に指定された cache オプションが force-cache の場合はエラーとなる

※太字は `'auto'` 指定との差異

| fetch() - cache オプション | Dynamic Functions | キャッシュの利用     |
| -------------------------- | ----------------- | -------------------- |
| (指定なし)                 | なし              | **キャッシュしない** |
| force-cache                | なし              | **(エラー)**         |
| no-store                   | なし              | キャッシュしない     |
| (指定なし)                 | 実行後            | キャッシュしない     |
| force-cache                | 実行後            | **(エラー)**         |
| no-store                   | 実行後            | キャッシュしない     |

---

というわけで fetchCache オプションの紹介でした。

`only-no-store` などは、決してキャッシュさせたくないことが明確なページなどで事故防止のために使えそうですね。

`fetchCache` オプションは現状ではまだ動作が怪しいですが、App Router が正式なものになった際には活用できるかもしれません。
