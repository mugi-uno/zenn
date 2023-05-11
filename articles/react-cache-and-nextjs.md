---
title: "React cache() で Next.js の Per-request Caching が実現できるのはなぜか"
emoji: "🐑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js", "フロントエンド", "React"]
published: false
publication_name: "cybozu_frontend"
---

Next.js App Router では、リクエスト単位で処理をキャッシュする機構が存在し、ドキュメント上にも **Per-request Caching** として説明があります。

https://nextjs.org/docs/app/building-your-application/data-fetching/caching#per-request-caching

ひとつは、fetch() による Automatic fetch() Request Deduping で、単一リクエスト内で同一の fetch()リクエストが存在した場合、重複を排除し、最低限の実行としてくれます。これには特に設定などは必要なく、一定条件（GET かどうか、など）を満たしていれば自動的に最適化されます。

詳細は ↓ をご覧ください。

https://zenn.dev/cybozu_frontend/articles/next-caching-dedupe

そしてもう一つ、React が提供する cache() 関数を実行することでも同様にリクエスト単位でのキャッシュが実現できます。

https://nextjs.org/docs/app/building-your-application/data-fetching/caching#react-cache

これは Automatic fetch() Request Deduping では対処できないケースで活躍し、DB からの直接のデータ取得や、GraphQL の実行も最適化できます。

## cache() を呼ぶだけで Per-request Caching になるのはなんで？

筆者は Next.js や React の実装の詳細な部分までは明るくはなく、ひとつ疑問がありました。

cache() 関数は React が提供するものですが、これを呼ぶだけで Next.js リクエストスコープが実現できるのはどういうことなのでしょう？

なんだかモヤモヤしてしまうので、コードを追って仕組みを確認してみましょう。

### 最初に結論

AsyncLocalStorage でいい感じにやってる

## cache() 自体のコードを追う

cache() 関数は、React 内の ReactCache.js に定義されています。

https://github.com/facebook/react/blob/16d053d592673dd5565d85109f259371b23f87e8/packages/react/src/ReactCache.js

色々と処理をしていますが、基本は次の内容だと思われます。

- `dispatcher` (ReactCurrentCache.current) から Map を取得する
- Map にキャッシュがあればそれを利用。なければ関数を実行してキャッシュに記録

`ReactCurrentCache.current` の設定箇所を探してみると、`ReactFlightServer#createRequest()` に行き着きます。

https://github.com/facebook/react/blob/16d053d592673dd5565d85109f259371b23f87e8/packages/react-server/src/ReactFlightServer.js#L188-L204

設定される実体は `DefaultCacheDispatcher` のようです。

https://github.com/facebook/react/blob/16d053d592673dd5565d85109f259371b23f87e8/packages/react-server/src/flight/ReactFlightServerCache.js#L26-L46

Map の取得のために呼ばれている関数を辿ると

- `getCacheForType()`
  - `resolveCache()`
    - `resolveRequest()`

と、`ReactFlightServer#resolveRequest()` が大本であり、`requestStorage.getStore()` というコードにたどり着きます。

https://github.com/facebook/react/blob/16d053d592673dd5565d85109f259371b23f87e8/packages/react-server/src/ReactFlightServer.js#L246-L252

`requestStorage` 自体は、実行される環境に応じたパッケージによって設定されます。

Next.js App Router の場合 `react-server-dom-webpack/server.edge` が利用されます。

https://github.com/vercel/next.js/blob/c93747eb33582bf8988dc7a53d4cf9d643b4a753/packages/next/src/build/webpack/loaders/next-app-loader.ts#L605

これに相当する React 側の Config は `packages/react-server/src/forks/ReactFlightServerConfig.dom-edge-webpack.js` が該当するようです。
（Rollup でのビルド時に `dom-edge-webpack` などのファイル名に応じて、対応する Config が利用されるようです。）

https://github.com/facebook/react/blob/16d053d592673dd5565d85109f259371b23f87e8/packages/react-server/src/forks/ReactFlightServerConfig.dom-edge-webpack.js#L17-L20

というわけで、どうやら `cache()` 関数では、`AsyncLocalStorage` で管理される Map をもとにキャッシュ管理をするようです。

## AsyncLocalStorage

https://nodejs.org/api/async_context.html

`AsyncLocalStorage` は Node.js で利用可能な API で、これを利用すると非同期操作・Promise Chain などが含まれていてもメモリセーフに状態管理を行えます。

[`AsyncLocalStorage#run()`](https://nodejs.org/api/async_context.html#asynclocalstoragerunstore-callback-args)経由でコールバックを実行すると、第一引数として与えた Store はスレッドローカルに扱われます。

では、この `run()` はどこで呼ばれているのでしょうか？

## Next.js のレンダリング側から追う

Next.js App Router ではレンダリング時に `renderToReadableStream` を実行します。これは、 `renderHTML` → `renderToHTMLOrFlight` 経由で実行されます。

https://github.com/vercel/next.js/blob/c93747eb33582bf8988dc7a53d4cf9d643b4a753/packages/next/src/server/next-server.ts#L940-L957
https://github.com/vercel/next.js/blob/c93747eb33582bf8988dc7a53d4cf9d643b4a753/packages/next/src/server/app-render/app-render.tsx#L1188-L1189

`renderToReadableStream` の実体を見てみると、`ReadableStream` の `start` メソッドのタイミングで `startWork` 関数が実行されていることがわかります。

https://github.com/facebook/react/blob/16d053d592673dd5565d85109f259371b23f87e8/packages/react-server-dom-webpack/src/ReactFlightDOMServerEdge.js#L64-L66

そしてこの `startWork` を見てみると..

https://github.com/facebook/react/blob/16d053d592673dd5565d85109f259371b23f87e8/packages/react-server/src/ReactFlightServer.js#L1358-L1365

`requestStorage.run` がありました！！

このとき第一引数に渡される `request` オブジェクトは、`renderToReadableStream` 内での `createRequest` で生成され、`cache()` 関数で利用される `request.cache` に対して空の Map を指定しており、これがキャッシュとして利用されます。

https://github.com/facebook/react/blob/16d053d592673dd5565d85109f259371b23f87e8/packages/react-server/src/ReactFlightServer.js#L215

## まとめ

長くなりましたが、整理すると次のような流れのようです。

- Next.js App Router でリクエストごとのレンダリング時に `renderToReadableStream` が呼ばれる
- `renderToReadableStream` 内で `AsyncLocalStorage#run()` を実行し、キャッシュ用の Map がスレッドローカルで利用可能になる
- `cache()` 実行時は、`AsyncLocalStorage` 経由で Map を取得しキャッシュを管理する

`AsyncLocalStorage` をフル活用している感じですね。

むずかしかった...
