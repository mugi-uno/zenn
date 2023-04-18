---
title: "Next.js 13 の cache 周りを理解する - revalidate"
emoji: "🚚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js", "フロントエンド"]
published: false
publication_name: "cybozu_frontend"
---

Next.js 13 App Router の cache 周りを理解したい記事の第二弾です。

前回:
https://zenn.dev/cybozu_frontend/articles/next-caching-dedupe

# Incremental Static Regeneration / ISR

Next.js では、もともとレンダリング方法のひとつとして、Incremental Static Regeneration = ISR が利用可能でした。

https://nextjs.org/docs/basic-features/data-fetching/incremental-static-regeneration

静的なページ生成を前提としながらも、一定間隔を超えた時点でページを再生成してくれる仕組みで、間隔は、`getStaticProps` の `revalidate` オプションで指定可能でした。

```ts:従来のISRとrevalidate
export async function getStaticProps() {
  const res = await fetch("https://example.com/api");
  const posts = await res.json();

  return { revalidate: 10 };
}
```

Next.js 13 App Router でも、ISR に相当する機能は存在しており、従来よりも細かい制御が可能となっています。

# fetch() ごとの revalidate

従来 ISR はページ単位での制御でしたが、Next.js 13 App Router からは、個々の fetch() リクエスト単位で revalidate 期間が制御可能になりました。

利用する際には、fetch() のオプションに `revalidate` を直接指定します。
https://beta.nextjs.org/docs/api-reference/fetch#optionsnextrevalidate

```ts:fetch()とrevalidate
// キャッシュのライフタイムを30秒にする
fetch(`https://example.com/api`, { next: { revalidate: 30 } });
```

実際に動きを見てみましょう。

次のようにネストしたレイアウト・ページで、異なる API に対して fetch()を実行し、それぞれに revalidate を設定しています。

- Root Layout: fetch()の revalidate=5
  - Foo Layout: fetch()の revalidate=3
    - Bar Layout: fetch()の revalidate=2
      - Bar Page: fetch()の revalidate=1

```tsx:revalidateを用いたページコンポーネントの例
export default async function Bar() {
  const res = await fetch("http://localhost:3001/sample-api?page=bar", {
    next: { revalidate: 1 },
  });
  const { data } = await res.json();

  return (
    <div className="bg-blue-300 p-4 border-2 border-gray-400 rounded">
      Bar (revalidate 1) : {data}
    </div>
  );
}
```

リロードを繰り返すと、fetch() ごとに revalidate で指定した間隔でキャッシュが更新されていることがわかります。

![](/images/next-caching/revalidate-1.gif)

## `cache` オプションと `revalidate` オプションの競合

ところで、fetch() には、`cache` オプションも存在し、キャッシュの取り扱いを指定できます。特に指定しなかった場合には `cache` に `force-cache` を指定した場合と同義となり、すべてのクエリがキャッシュされます。`cache` に `no-store` を指定すると、常に最新データをフェッチするようになります。

しかし、`revalidate` と組み合わせて次のような指定をした場合には矛盾が生じるように感じます。

- `cache`=`no-store` と `revalidate`に 1 以上の値を指定した場合
- `cache`=`force-cache` と `revalidate`に 0 を指定した場合

実際に指定してみると、両方指定した場合にはどうやら `cache` の指定が優先されるようです。

![](/images/next-caching/revalidate-2.gif)

![](/images/next-caching/revalidate-3.gif)

しかし、beta 版のドキュメント^[https://beta.nextjs.org/docs/api-reference/fetch#optionsnextrevalidate]には次の記述があります。

> As a convenience, it is not necessary to set the cache option if revalidate is set to a number since 0 implies cache: 'no-store' and a positive value implies cache: 'force-cache'.

`revalidate` 指定時には、値に応じて適切な `cache` が指定されたのと同等の動きとなるようで、基本的に両方を指定するのは想定されていないようです。

ただ、同時に次の記述もあります。

> Conflicting options such as { revalidate: 0, cache: 'force-cache' } or { revalidate: 10, cache: 'no-store' } will cause an error.

両方指定するとエラーになるそうです。
これについては私の手元では確認できず、先に確認した内容の通り、`cache` が優先されるような動作となりました。

# レイアウト・ページ単位での revalidate

fetch() 単位ではなく、ページ単位での revalidate も指定可能です。

App Router では、**Route Segment Config**^[https://beta.nextjs.org/docs/api-reference/segment-config#revalidate]という仕組みで、レイアウトあるいはページから特定の変数を export すると動作をカスタマイズできます。

そのうちの一つの値に `revalidate` が用意されており、デフォルトの ravalidate 期間を指定できます。

```ts
export const revalidate = 2;
```

次のように、ページのみから revalidate を export してみます。

- Root Layout: revalidate=指定なし
  - Foo Layout: revalidate=指定なし
    - Bar Layout: revalidate=指定なし
      - Bar Page: revalidate=2

すると、ページから export されている revalidate の値に従い、すべての fetch() のキャッシュが 2 秒を超えるとリフレッシュされることがわかります。

![](/images/next-caching/revalidate-page.gif)

なお、複数のレイアウト・ページから revalidate が export されており、かつその値が異なっている場合は、もっとも更新頻度が高くなる値が適用されます。

たとえば、次のように revalidate を設定すると、Foo Layout の revalidate=1 が全体として適用されるようになります。

- Root Layout: revalidate=10
  - Foo Layout: revalidate=1
    - Bar Layout: revalidate=5
      - Bar Page: revalidate=15

![](/images/next-caching/revalidate-page2.gif)

なお、`revalidate = 0` を指定すると、常に動的にクエリを発行してのレンダリングがデフォルト動作となります。

# fetch()とレイアウト・ページの revalidate の併用

fetch()のオプションと Route Segment Config のそれぞれで revalidate が設定できることがわかりました。

では、両方を指定したらどうなるでしょうか？

ドキュメント内の Route Segment Config - revalidate の説明^[https://beta.nextjs.org/docs/api-reference/segment-config#revalidate]を見てみると、次の記述があります。

> Set the default revalidation time for a layout or page. This option does not override the revalidate value set by individual fetch requests.

> DeepL による翻訳:
> レイアウトまたはページのデフォルトの再バリデーション時間を設定します。このオプションは、個々のフェッチ・リクエストによって設定された revalidate 値を上書きすることはありません。

つまり、Route Segment Config の値は全体としてのデフォルトの値を設定するものですが、それよりも常に fetch() のオプションに指定された revalidate 間隔は優先して適用されます。

次のようなケースで確認してみましょう。

- Root Layout: revalidate=10, fetch()の revalidate 指定なし
  - Foo Layout: revalidate=1, fetch()の revalidate 指定なし
    - Bar Layout: revalidate=5, fetch()の revalidate=3
      - Bar Page: revalidate=15, fetch()の revalidate 指定なし

すると、各ページ・レイアウトで呼ばれる fetch()リクエストは次のような revalidate 間隔となります。

- Root Layout: revalidate 間隔=1 (Foo Layout の値を利用)
  - Foo Layout: revalidate 間隔=1 (Foo Layout の値を利用)
    - Bar Layout: revalidate 間隔=3 (fetch のオプションの値を利用)
      - Bar Page: revalidate 間隔=1 (Foo Layout の値を利用)

![](/images/next-caching/revalidate-page3.gif)

特定のレイアウト配下全体で一定のポリシーで revalidate 間隔を指定しつつ、キャッシュは使いたくない or もっと長期間のキャッシュ保持期間としたい API などに対して、個々に fetch()オプションを指定することで、柔軟に対応ができそうですね。

---

というわけで revalidate の話でした。

fetch() と レイアウト・ページの 2 軸の設定が可能な点がポイントですね。
