---
title: "Next.js Server Actions の裏側を理解したくて動きとコードを追う"
emoji: "🕵️‍♀️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js", "フロントエンド", "React"]
published: false
publication_name: "cybozu_frontend"
---

# Next.js Server Actions

Next.js 13.4 で、新機能として Server Actions^[https://nextjs.org/blog/next-13-4#server-actions-alpha] が追加され、バックエンド側のコードをあたかもクライアントから直接呼び出せるかのようにコードを書くことができるようになりました。

リリースブログに掲載されていた Server Actions のコードを見てみましょう。

```tsx
// app/post/[id]/page.tsx (Server Component)

import kv from "./kv";

export default function Page({ params }) {
  async function increment() {
    "use server";
    await kv.incr(`post:id:${params.id}`);
  }

  return (
    <form action={increment}>
      <button type="submit">Like</button>
    </form>
  );
}
```

この場合、 `increment()` 関数にはデータストアへのアクセスを行うバックエンドロジックが含まれますが、それを `<form>` の `action` に直接指定しています。
この状態で Submit した場合、Next.js 側がいい感じに吸収して、`increment()` 関数を実行することができます。すごいですね。

# Server Actions、実際のところ何がどう動いてるの？

先述の例で「Next.js 側がいい感じに吸収して」と書きましたが、これは一体何が起きてるのでしょうか？

さすがに謎が深いので、もう少し中身を追ってみましょう、というのがこの今回の記事の内容です。

## 簡易的な Server Actions 実行コードを用意

まず、動作確認のため超簡易的な Server Actions 実行用コードを用意してみます。

`npx create-next-app@latest` でアプリを作成し、`/action` というパスで次のようなページを用意します。

```tsx
export default async function Page() {
  async function myAction() {
    "use server";

    console.log("called myAction");

    await new Promise((resolve) => setTimeout(resolve, 500));

    console.log("done myAction");
  }

  return (
    <form action={myAction}>
      <button type="submit">Submit</button>
    </form>
  );
}
```

Submit ボタンが存在するだけで、クリックすると中身のない Server Actions を実行します。

![](/images/sa-dive/page.png)

実行してみると、バックエンド側でログが出ていることがわかります。

![](/images/sa-dive/log.png)

動いてるようですね。

## レンダリングされている内容

DevTools から、実際どのような HTML が描画されているのか見てみます。

![](/images/sa-dive/dom.png)

次のように描画されるようです。

- `<form>` に `enctype="multipart/form-data"` が付与
- `$ACTION_ID_64e74a461bb5c3e4c531c75dddd90882ed42773e` という name が付与されている `<input type="hidden" />` が追加

謎の ID みたいなものが出てきました。

## 謎の name `$ACTION_ID_xxx` を追う

おもむろに `next build` した結果出力される `.next` 配下を見てみると、`.next/server/server-reference-manifest.json` というファイルが生成され、次の定義となっています。

```json
{
  "node": {
    "64e74a461bb5c3e4c531c75dddd90882ed42773e": {
      "workers": { "app/action/page": 1716 },
      "layer": { "app/action/page": "sc_server" }
    }
  },
  "edge": {}
}
```

`64e74a461bb5c3e4c531c75dddd90882ed42773e` が、明らかに `<input type="hidden" />` に付与されていた name とマッチしてるので、何らかの突き合わせに使われてそうです。

次に、prefix になっている `$ACTION_ID_` のほうで Next.js 側のコードを grep してみると、次のファイルがヒットします。

- react-server-dom-webpack-client.edge.development.js - `encodeFormAction()`
- react-server-dom-webpack-server.edge.development.js - `decodeAction()`

これらの実体は React 本体側に存在するコードです。

`encodeFormAction()`
https://github.com/facebook/react/blob/21a161fa37dce969c58ae17f67f2856d06514892/packages/react-client/src/ReactFlightReplyClient.js#L410-L457

`decodeAction()`
https://github.com/facebook/react/blob/21a161fa37dce969c58ae17f67f2856d06514892/packages/react-server/src/ReactFlightActionServer.js#L56-L111

> 余談ですが、Server Actions って Next.js の機能で基本的にコードも Next 側で完結してるのかと思いこんでいました。普通に React 側に手が入ってるんですね、知りませんでした…

### `encodeFormAction()`

この関数が return している値は ↓ の通りです。

```ts
return {
  name: name,
  method: "POST",
  encType: "multipart/form-data",
  data: data,
};
```

`<form />` を構築する際に必要な値ぽいですね。

### `decodeAction()`

FormData に含まれる `$ACTION_ID_xxx` を元に、`xxx` を対象に `loadServerReference()` を呼び出しています。

```ts
if (key.startsWith("$ACTION_ID_")) {
  const id = key.slice(11);
  action = loadServerReference(serverManifest, id, null);
  return;
}
```

ここで渡される `serverManifest` は、Next.js 側から渡されます。

Next.js: `packages/next/src/server/app-render/action-handler.ts`

https://github.com/vercel/next.js/blob/9bc73dbc1a329377f9ea30ad2acd12fb4bad48b4/packages/next/src/server/app-render/action-handler.ts#L258-L271

https://github.com/vercel/next.js/blob/9bc73dbc1a329377f9ea30ad2acd12fb4bad48b4/packages/next/src/server/app-render/action-handler.ts#L292-L300

`serverActionsManifest` を辿っていくと、`server-reference-manifest.json` に辿り着きます。

https://github.com/vercel/next.js/blob/9bc73dbc1a329377f9ea30ad2acd12fb4bad48b4/packages/next/src/export/index.ts#L488-L492

つまり、Action ごとに一意の ID が割り振られており、その ID をもとにマッピングした関数を呼び出す仕組みになっているようですね。

### `handleAction()`

`decodeAction()` を追っていくと、呼び出し元に `handleAction()` という関数が出てきますが、これは、App Router がリクエストを受け取りレンダリングを行う `renderToHTMLOrFlight` 経由で呼び出されています。

https://github.com/vercel/next.js/blob/9bc73dbc1a329377f9ea30ad2acd12fb4bad48b4/packages/next/src/server/app-render/app-render.tsx#L1606-L1621

`handleAction()` では、次のいずれかの条件のときのみ処理が実行されています。

- POST かつ contentType が 'application/x-www-form-urlencoded'
- POST かつ contentType が 'multipart/form-data' から始まる値
- POST かつ リクエストヘッダに 'Next-Action' が含まれる

https://github.com/vercel/next.js/blob/9bc73dbc1a329377f9ea30ad2acd12fb4bad48b4/packages/next/src/server/app-render/action-handler.ts#L244-L255

最初に作ったサンプルページは `/action` というパスで作成しましたが、Server Action が POST される先も同じ URL です。

![](/images/sa-dive/submit.png)

ここまで見た内容から

- Server Actions ではそれ用に特別な URL を用意しているわけではない
- 通常のレンダリングの途中で、条件を満たす場合に Server Actions を実行する

という流れになっていることがわかりました。

## `revalidatePath` `revalidateTag` の実行時に描画を更新できるのはなぜ？

Server Actions の中で `revalidatePath` `revalidateTag` を呼び出すと、そのレスポンスによって即座に画面を最新化できます。

この挙動は別の記事でも紹介しています。

https://zenn.dev/cybozu_frontend/articles/server-actions-and-revalidate

なぜこういった動作になるのか不思議だったのですが、これも先ほど確認した

> 通常のレンダリングの途中で、条件を満たす場合に Server Actions を実行する

という挙動がわかってしまえばそれほど不思議でもなく、通常どおりページを構築する途中で `revalidatePath` `revalidateTag` が実行されるため、そのままの流れで自動的に該当する fetch() などの箇所は最新化したデータが取得されていただけですね。

## Custom invocation 時の挙動の違いはあるのか？

ここまでは form action を利用する形での確認を行ってきました。しかし、Server Actions には Custom Invocation と呼ばれる、Client Component からも呼び出せる形での実行方法があります。この場合には何か挙動に差異はあるのでしょうか？

実際に動作を見てましょう。

`/custom-invocation` というパスで新たに Custom Invocation を行うページを作成します。

```ts:action.ts
"use server";

export async function myAction() {
  console.log("called myAction");

  await new Promise((resolve) => setTimeout(resolve, 500));

  console.log("done myAction");
}
```

```tsx:MyComponent
"use client";

import { useTransition } from "react";
import { myAction } from "./action";

export function MyComponent() {
  const [_, startTransition] = useTransition();

  const handleClick = () => {
    startTransition(async () => {
      await myAction();
    });
  };

  return <button onClick={handleClick}>Custom Invocation</button>;
}
```

```tsx:page.tsx
import { MyComponent } from "./MyComponent";

export default async function Page() {
  return <MyComponent />;
}
```

画面では、ボタンがひとつ描画されるのみです。

![](/images/sa-dive/ca-init.png)

そして実際に実行してみると、一部リクエストヘッダが form action とは異なります。

![](/images/sa-dive/ca-post.png)

form action のときは ContentType が multipart/form-data でしたが、今回は text/plain です。この場合、Next.js 側の `handleAction()` 内の分岐が微妙に変化し、`isMultipartAction` は false として扱われるようです。

https://github.com/vercel/next.js/blob/c8f65ede874ff8ae6e049a93882e7d2374d8e5fe/packages/next/src/server/app-render/action-handler.ts#L246-L247

とはいえ、Server Actions に渡すパラメータの解析などに微妙に違いはありつつも、基本的には同じ挙動となるようです。

---

Server Actions を実際に動かしてみたときは「なんだこれは魔法やで・・・」と思ってしまいましたが、コードを眺めてみると雰囲気が掴めて、魔法ではなく意外とシンプルに処理していることがわかりました。中を見てみるのは大事ですね！
