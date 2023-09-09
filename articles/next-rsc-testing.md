---
title: "Next.jsでServer Componentsがちょっとだけテストできるようになってた"
emoji: "🚚"
type: "tech"
topics: ["nextjs", "フロントエンド"]
published: false
publication_name: "cybozu_frontend"
---

先日 Vercel の[@leeerob](https://twitter.com/leeerob)氏が次のようなツイートをしていました。

https://twitter.com/leeerob/status/1698758756924825723

> I'm working on updated testing docs for the @nextjs App Router.
> `next@canary` supports Jest for server & client components, metadata, `server-only`, and more

なんと `next@canary` で、Server Components の Jest でのテスト実行がサポートされているとのことです！これは試さないと！

## next@13.4.19 時点での Server Components テストの課題

Next.js App Router で開発するときの大きな課題の一つがテストで、現状では Server Components と testing-library を組み合わせたテストにはかなり制限があります。

### `server-only` パッケージの import があるとテストできない

Server Components を用いた開発をしていると、バックエンド向けのコードがクライアント側に紛れ込んでしまうリスクがあります。たとえば、秘匿情報である環境変数を参照するバックエンド向けの関数を Client Component で import してしまうと、コード上にバンドルされて漏れ出てしまいます。

対策として、バックエンドでのみ利用されるコードで `server-only` パッケージを import しておくと、仮にこれを Client Component から import していた場合は、ビルドの時点でエラーとして弾くことができる仕組みが存在します。

https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns#keeping-server-only-code-out-of-the-client-environment

一方、Next.js でのコンポーネントを Jest でテストする際は、`next/jest` を用いると、SWC を利用した Rust Compiler を利用できます。
https://nextjs.org/docs/pages/building-your-application/optimizing/testing#setting-up-jest-with-the-rust-compiler

しかし、testing-library などを使うために Jest の testEnvironment を `jest-environment-jsdom` とすると、現状ではすべてのコンポーネントが Client Component として処理されてしまい、先述の `server-only` チェックの結果でビルドエラーが発生し、テストが実行できません。

参考: next@13.4.19 で ClientComponents 判断をしている箇所
https://github.com/vercel/next.js/blob/d96e0258de2caf34e9322d0d32ab5748533c4465/packages/next/src/build/swc/jest-transformer.ts#L101-L103

![](/images/rsc-test/server-component-failure.png)
_server-only で NEXT_RSC_ERR_CLIENT_IMPORT エラーが発生した様子_

### `metadata` / `generateMetadata`

テストにおける `server-only` と似た問題として、`metadata` / `generateMetadata` の存在も挙げられます。

App Router の Page または Layout で OGP などの metadata を提供する際、`metadata` 定義 or `generateMetadata` 関数が利用できます。

https://nextjs.org/docs/app/building-your-application/optimizing/metadata

しかし、こちらも `server-only` と同様、Client Component で利用すると Rust Compiler 側のチェックでエラーが発生します。

![](/images/rsc-test/metadata-failure.png)
_metadata の export で NEXT_RSC_ERR_CLIENT_METADATA_EXPORT エラーが発生した様子_

仮に Page コンポーネントで Async Component などの Server Components 固有の機能を利用していなくても、`metadata` / `generateMetadata` が定義されているだけでビルドエラーとなるため、テストが実行できません。

## 該当の PR で解決された Issue

[該当の PR](https://github.com/vercel/next.js/pull/54891)で対応された Issue は次の 2 つです。

- [\[NEXT-889\] Unable to Test Server Components in Next.js 13 App Dir · Issue #47448 · vercel/next.js](https://github.com/vercel/next.js/issues/47448)
- [\[NEXT-863\] Unable to test page components using metadata API with Jest · Issue #47299 · vercel/next.js](https://github.com/vercel/next.js/issues/47299)

これらは、先にテスト時の課題として挙げた `server-only` と、`metadata` / `generateMetadata` の問題に該当します。

## 実際に試してみる

というわけで実際に試してみます。

Next.js アプリケーションの設定と Jest の設定を行い、次のコードを用意します。
（事前の設定詳細はこの記事では割愛します）

```tsx:Page.tsx
import { serverOnlyFn } from "./lib";

export const metadata = {
  title: "my title",
};

export default function Page() {
  serverOnlyFn();
  return <h1>yeah</h1>;
}
```

```ts:lib.ts
import "server-only";

export const serverOnlyFn = () => {
  return "this is server only function!";
};
```

```tsx:Page.test.tsx
import { render, screen } from "@testing-library/react";
import Page from "./page";

it("sample test", () => {
  render(<Page />);
  expect(screen.getByRole("heading")).toHaveTextContent("yeah");
});
```

`server-only` と `metadata` の両方が定義されています。

このテストを、`next@13.4.19` で実行するとエラーとなります。

![](/images/rsc-test/latest-failure.png)
_`next@13.4.19`でエラーとなる様子_

一方、この記事の執筆時点での最新 canary である `next@13.4.20-canary.20` で実行すると、問題なくテストがパスします。

![](/images/rsc-test/canary-success.png)
_`next@13.4.20-canary.20`でテストがパスする様子_

確かに解決されていそうですね！

## なぜパスするようになったのか？

せっかくなので、なぜテストがパスするようになったのかも調べてみましたが、かなりシンプルな対応でした。

Rust Compiler 側で `server-only` など Client Component の妥当性をチェックする `assert_client_graph` という関数が存在しますが、その先頭で `disable_checks` オプションが有効な場合はチェックを丸ごとスキップするようになったようです。

https://github.com/vercel/next.js/blob/v13.4.20-canary.20/packages/next-swc/crates/core/src/react_server_components.rs#L411-L430

実際、Jest で用いられるオプションでは `disableChecks` に true が設定されています。

https://github.com/vercel/next.js/blob/v13.4.20-canary.20/packages/next/src/build/swc/options.ts#L282

「チェック飛ばして大丈夫なの？」とも思いましたが、`next build` の実行時には従来通りチェックされるため、すり抜けて公開されることはないようです。

## Async Server Components は？

さて、Server Components のテストにおいてほぼぶつかる課題が Async Server Components の存在です。

最初のツイートを見たとき「ついに Async Server Components のテストが！！」とテンションが上がってすぐに内容を確認したのですが、残念ながらそちらは未サポートで、別の Issue が存在しています。

https://github.com/vercel/next.js/issues/47131

実際動かしてみても現状では動作しません。

```tsx:page.tsx(Async Server Components)
import { serverOnlyFn } from "./lib";

export const metadata = {
  title: "my title",
};

// async を付与
export default async function Page() {
  serverOnlyFn();
  return <h1>yeah</h1>;
}
```

![](/images/rsc-test/error-async.png)
_Async Server Components のテストがエラーとなる様子_

ただこれは Next.js というよりも、テストライブラリ側で RSC のレンダリングが可能となる必要があるため、別途 testing-library 側でも Issue が存在し議論されています。

https://github.com/testing-library/react-testing-library/issues/1209

何か最適な解決策が登場するまで、今しばらくは Async Server Components のテストは E2E などで対応するか、あるいは Playwright をベースにした next/testmode^[https://zenn.dev/uyas/articles/bc58a4bed15ed4] を利用するなど、他の方法でテストを行う必要がありそうです。

## まとめ

というわけで、Next.js 13 App Router で Server Components のテストがちょっとだけできるようになったよ！というまとめでした。

今回対応されたのは限定的な範囲ではありますが、Next.js 公式としてテストを改善していく方向性が感じられたのは嬉しいですね。
