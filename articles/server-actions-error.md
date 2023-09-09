---
title: "Next.js Server Actions でのエラー周りの挙動を確認する"
emoji: "💥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "フロントエンド", "React"]
published: true
publication_name: "cybozu_frontend"
---

# Next.js Server Actions

Next.js 13.4 に、新機能として **Server Actions** が追加されました。

https://nextjs.org/blog/next-13-4#server-actions-alpha

2023 年 5 月現在では α 版ですが、Next.js でのアプリケーション設計方法に大きく影響を与えそうな気配をガンガン醸し出しています。

## Server Actions？

Server Actions 自体が何かについては、すでに多くの方が記事を投稿されているため、詳細な説明は省きますが、とりあえずは「"use server" を付与したバックエンド処理を含む関数をクライアント側から直接呼び出すかのように振る舞う機能」と認識しておけばよいかと思います。（もちろんそのまま直接実行はできないため、実際には Next.js の仕組みで間接的にバックエンドと通信して実行しています。）

Next.js の公式ドキュメントにも Server Actions の解説ページが存在するため、そちらも参考になります。

https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions

## エラーの取り扱いがよくわからない

前述の通り Server Actions では、バックエンド処理を含む関数をクライアント側から実行できます。

このとき、Server Actions 側でエラーが発生したらどういう挙動になるのでしょうか。また、エラーはどのようにハンドリングすればよいのでしょう。

というわけで、そのあたりドキュメントを見てもあまりピンと来なかったため、実際に動きを確認してみたよ、という記事です。

# 正常なケースの動作を確認する

まず、正常ケースの動作を確認しておきましょう。

簡単な Server Actions を作成します。ファイル先頭に "use server" を付与することで、ファイル内すべての関数が Server Actions として定義できます。

```ts:action.ts
"use server";

export async function myAction() {
  return "success";
}
```

そして、これを呼び出す Client Component / Server Component を作成します。
呼び出し方は２種類用意し、Client Component からは Custom invocation (`startTransition` を介した実行) とし、Server Component 側からは form の action 経由で実行します。

```tsx:ActionComponent(Client Component)
"use client";

import { startTransition } from "react";
import { myAction } from "./action";

export const ActionComponent = () => {
  const handleClick = () => {
    startTransition(() => {
      (async () => {
        const res = await myAction();
        console.log(`Called from Client Component : ${res}`);
      })();
    });
  };

  return (
    <div className="inline-block p-2 border-green-400 border rounded-sm">
      <div>Client Component</div>
      <button
        type="button"
        onClick={handleClick}
        className="p-2 border-gray-400 bg-white border rounded"
      >
        Call Action
      </button>
    </div>
  );
};
```

```tsx:page.tsx(Server Component)
import { ActionComponent } from "./ActionComponent";
import { myAction } from "./action";

export default function Page() {
  return (
    <div className="p-4 bg-blue-100">
      <div>Page (Server Component)</div>
      <form action={myAction} className="mb-2">
        <div>
          <button
            type="submit"
            className="p-2 border-gray-400 bg-white border rounded"
          >
            Submit
          </button>
        </div>
      </form>

      <ActionComponent />
    </div>
  );
}
```

見た目は次のようになります。

![](/images/sa-error/success-1.png)

実行してみると、実際には fetch リクエストとして送信されており、Server Actions の実行結果が戻り値として返されていることがわかります。

![](/images/sa-error/success-2.png)

# エラーハンドリング : Error Boundary がある場合

では、肝心のエラー時の挙動を見てみましょう。

まず、Next.js App Router では error.js というファイルでページと同階層に Client Component を配備しておくと、自動的に Error Boundary で Wrap してエラーハンドリングすることができます。これと組み合わせてどうなるか見てみましょう。

まずファイルを作成します。（TypeScript なので .tsx 拡張子です）

```tsx:error.tsx
"use client";

export default function Error() {
  return <div className="p-4 bg-red-400 text-white">Page Error!</div>;
}
```

そして、Server Actions では意図的にエラーを発生させます。

```ts:action.ts
"use server";

export async function myAction() {
  throw new Error("Action Error!!");
}
```

## form action でエラーが発生した場合

まず、form action でエラーが発生したケースを見てみます。

"Submit" ボタンをクリックしてみると..

![](/images/sa-error/error-1.png)

Error Boundary によってキャッチできていますね。form action 経由での実行の場合、意図しない例外などはこれで処理できそうです。

## Custom Invocation でエラーが発生した場合

では、form action を介さない、Custom Invocation でエラーが発生したケースを見てみます。

"Call Action" ボタンをクリックしてみると...

![](/images/sa-error/error-2.png)

Error Boundary ではキャッチできないようです。

これは Next.js というよりも、Error Boundary そのものの制約によるものだと思われます。

https://legacy.reactjs.org/docs/error-boundaries.html#introducing-error-boundaries

> Error boundaries do not catch errors for:
>
> ・Event handlers (learn more)
> ・Asynchronous code (e.g. setTimeout or requestAnimationFrame callbacks)
> ・Server side rendering
> ・Errors thrown in the error boundary itself (rather than its children)

今回の場合は、Event handlers に該当するため Error Boundary では処理できません。

# Custom Invocation の実行時に try-catch で拾えるか？

そうはいっても、現実的な問題として何らかの形でエラーが処理できないと困ります。

エラーキャッチといえば、基本に立ち返ると try-catch です。Custom Invocation 時のエラーは try-catch で拾えるのか試してみましょう。

Server Actions の実行を try-catch で囲ってみます。もしエラーが拾えた場合、それを画面に表示します。

```tsx:ActionComponent
"use client";

import { startTransition, useState } from "react";
import { myAction } from "./action";

export const ActionComponent = () => {
  const [error, setError] = useState<string | null>(null);

  const handleClick = () => {
    startTransition(() => {
      (async () => {
        try {
          const res = await myAction();
          console.log(`Called from Client Component : ${res}`);
        } catch (e) {
          setError((e as Error).message);
        }
      })();
    });
  };

  return (
    <div className="inline-block p-2 border-green-400 border rounded-sm">
      <div>Client Component</div>
      <button
        type="button"
        onClick={handleClick}
        className="p-2 border-gray-400 bg-white border rounded"
      >
        Call Action
      </button>

      {error && <div className="p-2 bg-red-400 text-white">{error}</div>}
    </div>
  );
};
```

そして再度実行してみると...

![](/images/sa-error/error-3.png)

無事にキャッチできました！！これで安心ですね！！！

## キャッチできていいのか...?

try-catch で例外拾えたやったー！と言いたいところですが、実際のところ throw されるエラーは意図的なものではないケースも多いかと思います。

仮に DB アクセスなどでエラーが発生した場合、SQL などの情報がエラーに含まれている可能性もあるでしょう。これがクライアント側に漏れちゃうのはめちゃくちゃ致命的なのでは...？という疑問が浮かびます。

しかし、ここまで動作を確認していたのは `next dev` で起動した開発モードでした。`next build` `next start` として production モードで起動してから同様の操作を行うと次のような結果になります。

![](/images/sa-error/error-4.png)

まさにメッセージに書いてある通りですが、クライアントに渡されるエラーについては、機密漏洩を防ぐ観点から具体的なメッセージは含まれません。

このあたりは、Next.js ドキュメント全体の Error Handling に説明があります。

https://nextjs.org/docs/app/building-your-application/routing/error-handling#handling-server-errors

つまり、Custom Invocation の場合、「try-catch でエラーは拾えるが、"エラーがあった"という事実以外は何も知ることができない」と考えたほうが良さそうです。

## 対応案: Result 型を利用する

Custom Invocaton も含めた形でのエラーハンドリングを考えてみましたが、一案として、レスポンスは正常系・異常系を含めて Result 型として扱うのが良いかもしれません。

```ts:action.ts
type Result =
  | {
      success: true;
      message: string;
    }
  | {
      success: false;
      error: string;
    };

export async function action(): Promise<Result> {
  try {
    await xxxFunction() // 何らかのエラーの可能性のある処理

    return {
      success: true,
      message: "success!!",
    };
  } catch (e) {
    return {
      success: false,
      error: "An error occurred",
    };
  }
}
```

それでも throw される Error に関しては、クライアント側では内容を知ることはできないため、何か汎用的なエラー処理を用意して処理する必要がありそうです。

# まとめ

form action なのか Custom Invocation なのかでエラーハンドリングの考え方を変えないといけないのは注意点になりそうです。

とはいえ、action 側から「どこで呼ばれるか？」まで想定して作ると、それはそれで責務の分離が曖昧になるため、Result 型で統一しておくなど、何か一定のルールを設けると良いかもしれません。
